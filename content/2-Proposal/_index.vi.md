---
title: "Bản đề xuất"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# RAG Knowledge Assistant

## Giải pháp AWS Serverless cho hỏi đáp tài liệu nội bộ

---

### 1. Tóm tắt

Dự án này xuất phát từ một bài toán thực tế mà nhiều tổ chức đang gặp phải: tài liệu nội bộ thì nhiều, nhưng tra cứu lại chậm và không hiệu quả.

**RAG Knowledge Assistant** là một chatbot hỏi đáp tài liệu nội bộ, được xây dựng trên nền kiến trúc **RAG (Retrieval-Augmented Generation)**. Thay vì chỉ dựa vào kiến thức huấn luyện sẵn của LLM, hệ thống cho phép nhân viên tải lên tài liệu thực tế (PDF, ảnh scan, văn bản thuần) và nhận câu trả lời được đối chiếu trực tiếp với nội dung đó.

Toàn bộ hệ thống chạy trên **AWS Serverless** — Lambda, SQS, Amazon Bedrock, DynamoDB — kết hợp với Terraform để đảm bảo hạ tầng nhất quán, có thể review và tái dựng dễ dàng. Ngoài luồng hỏi đáp chính, hệ thống còn có: semantic cache để kiểm soát chi phí, kiểm duyệt nội dung qua Bedrock Guardrails, giám sát vận hành thời gian thực, và vòng đánh giá chất lượng tự động hàng ngày bằng framework RAGAS.

---

### 2. Vấn đề & Giải pháp

#### Vấn đề

Khi bắt đầu nhìn vào bài toán quản lý tri thức nội bộ, có ba điểm mà tôi nhận ra là thực sự đáng giải quyết:

**Thứ nhất**, tài liệu của doanh nghiệp thường nằm rải rác trong hàng trăm file PDF và ảnh scan. Mỗi lần cần tra cứu, nhân viên phải mở từng file, tìm thủ công — vừa chậm, vừa lặp đi lặp lại không cần thiết.

**Thứ hai**, các LLM có sẵn tuy trả lời trôi chảy, nhưng không được đối chiếu với nội dung nội bộ thực tế. Điều này dẫn đến hiện tượng *hallucination* — câu trả lời sai nhưng nghe rất "tự tin" — đặc biệt nguy hiểm trong môi trường doanh nghiệp.

**Thứ ba**, thường không có cách nào đo lường định lượng xem chatbot có đang trả lời đúng không. Hầu hết các nhóm đánh giá bằng cảm tính: "có vẻ ổn" — mà điều đó rõ ràng là chưa đủ.

#### Giải pháp

Thay vì xây dựng một pipeline phức tạp nhiều tầng, tôi chọn hướng thiết kế tập trung vào **bốn luồng xử lý rõ ràng**, mỗi luồng có trách nhiệm cụ thể:

- Tài liệu được tiếp nhận qua **Amazon S3**, đưa qua hàng đợi đệm **Amazon SQS** (kèm Dead Letter Queue để đảm bảo retry an toàn), rồi **AWS Lambda** kết hợp **Amazon Textract** để số hóa các file scan.

- Nội dung trích xuất được chia thành chunk cha/con, tạo embedding qua **Amazon Bedrock**, và lưu trực tiếp trong **Amazon DynamoDB** dưới dạng vector đóng gói cùng dữ liệu BM25. Một lớp hybrid search tự viết bằng Python (cosine similarity + BM25, hợp nhất qua Reciprocal Rank Fusion) chạy ngay trong Lambda — không cần đến một search engine riêng.

- Câu hỏi từ người dùng đi qua **Amazon API Gateway** (bảo vệ bởi **Amazon Cognito**), kiểm tra cache **ElastiCache Serverless** trước để tránh gọi Bedrock không cần thiết, truy xuất ngữ cảnh qua hybrid search, rồi sinh câu trả lời qua **Amazon Bedrock (Claude 3)** được lọc qua **Bedrock Guardrails**.

- Hệ thống giám sát dùng **CloudWatch + SNS + AWS Chatbot** để phân loại và định tuyến cảnh báo tới Slack theo mức độ nghiêm trọng. **EventBridge Scheduler** kích hoạt một Lambda chạy hàng ngày để đánh giá các chỉ số RAGAS (Faithfulness, Answer Relevancy, Context Precision) trên các hội thoại gần nhất.

#### Tại sao thiết kế như vậy?

Một quyết định quan trọng là **không dùng search engine riêng** (như OpenSearch Serverless). Thay vào đó, lưu vector và dữ liệu BM25 trực tiếp trong DynamoDB. Điều này giúp loại bỏ một khoản chi phí nền chạy liên tục, giữ lớp retrieval ở mức pay-per-use — phù hợp với tinh thần serverless của toàn bộ hệ thống.

---

### 3. Kiến trúc giải pháp

Toàn bộ hạ tầng được quản lý qua Terraform để mọi thay đổi đều có thể review qua Pull Request, và có thể dựng lại từ đầu bất cứ lúc nào.

![Sơ đồ kiến trúc tổng thể RAG Knowledge Assistant](/images/5-Workshop/5.1-Workshop-overview/aws-new.drawio.png)

#### Dịch vụ AWS sử dụng

| Dịch vụ | Vai trò |
|---|---|
| **AWS Lambda** | Chạy logic xử lý tài liệu, chat engine, đánh giá RAGAS (Python 3.12) |
| **Amazon S3** | Lưu tài liệu gốc và kết quả đánh giá RAGAS |
| **Amazon SQS** | Đệm sự kiện xử lý tài liệu, kèm Dead Letter Queue |
| **Amazon Textract** | OCR cho file scan và ảnh |
| **Amazon Bedrock** | Sinh embedding (Titan/Cohere) và câu trả lời (Claude 3), kiểm soát qua Guardrails |
| **Amazon DynamoDB** | Lưu chunk tài liệu, vector, dữ liệu BM25, lịch sử hội thoại và feedback |
| **Amazon API Gateway** | Cung cấp endpoint chat, upload, kiểm tra trạng thái |
| **Amazon Cognito** | Xác thực người dùng trước khi cấp quyền API |
| **Amazon ElastiCache Serverless** | Cache cặp hỏi-đáp gần đây để giảm độ trễ và chi phí |
| **Amazon CloudWatch** | Thu thập log/metric, dashboard tùy chỉnh và cảnh báo |
| **Amazon SNS + AWS Chatbot** | Định tuyến cảnh báo theo mức nghiêm trọng tới Slack |
| **Amazon EventBridge Scheduler** | Kích hoạt job đánh giá RAGAS hàng ngày |
| **Terraform (HCP Terraform)** | Quản lý toàn bộ hạ tầng dưới dạng code, remote state |

#### Bốn luồng xử lý chính

**Luồng 1 — Data Ingestion**
S3 nhận file tải lên → S3 Event kích hoạt SQS → Lambda (Document Processor) trích xuất văn bản (Textract cho file scan) → chia chunk cha/con → tạo embedding + BM25 → lưu vào DynamoDB.

**Luồng 2 — Realtime Q&A**
API Gateway (sau Cognito) → Chat Engine Lambda kiểm tra cache → hybrid search trên DynamoDB → Bedrock sinh câu trả lời qua Guardrails → ghi hội thoại vào DynamoDB.

**Luồng 3 — Monitoring & Alert**
CloudWatch Alarms theo dõi lỗi Lambda, tỷ lệ 5xx, độ sâu DLQ, throttle của Bedrock → publish tới SNS topic phân theo mức nghiêm trọng → định tuyến tới Slack qua AWS Chatbot.

**Luồng 4 — RAG Evaluation**
EventBridge Scheduler → Lambda lấy mẫu hội thoại gần đây → chấm điểm RAGAS → lưu kết quả vào S3 → publish điểm tổng hợp lên CloudWatch.

---

### 4. Triển khai kỹ thuật

#### Các giai đoạn phát triển

Dự án theo chu kỳ **5 tuần** sau khi chốt đề tài, mỗi giai đoạn xây dựng trực tiếp trên nền của giai đoạn trước:

1. **Nghiên cứu & thiết kế kiến trúc** — Chốt đề tài, đánh giá lựa chọn giữa Serverless tự xây và các dịch vụ managed sẵn có (ví dụ Bedrock Knowledge Bases), hoàn thiện proposal cùng sơ đồ kiến trúc tổng thể và luồng dữ liệu.

2. **Chuẩn bị môi trường** — Thiết lập cấu trúc project Terraform/IaC, xin cấp quyền truy cập model trên Amazon Bedrock (Claude 3, Titan Embeddings), cấu hình môi trường phát triển.

3. **Xây dựng các luồng cốt lõi** — Triển khai Luồng 1 (Data Ingestion) trước, sau đó tới Luồng 2 (Realtime Q&A kèm Semantic Cache), kiểm thử thực tế từng luồng trước khi chuyển sang luồng tiếp theo.

4. **Quan sát & chất lượng** — Triển khai Luồng 3 (Monitoring & Alerting) và Luồng 4 (đánh giá RAGAS) để hệ thống có thể tự phát hiện sự cố và tự đo lường chất lượng câu trả lời.

5. **Hoàn thiện & bàn giao** — Tinh chỉnh tham số retrieval dựa trên kết quả RAGAS, kiểm thử tải, rà soát quyền IAM, tái cấu trúc Terraform theo module, hoàn thiện tài liệu và demo trực tiếp.

#### Yêu cầu kỹ thuật

- **Tài khoản & vùng**: AWS account ở **us-east-1 (N. Virginia)** — vùng hỗ trợ đầy đủ các model Bedrock cần dùng, cộng thêm tài khoản HCP Terraform để quản lý remote state.
- **Công cụ**: Terraform 1.5+, AWS CLI v2, Python 3.12, Git, và code editor.
- **Quyền hạn**: IAM policy triển khai được giới hạn đúng phạm vi các dịch vụ sử dụng — tuân thủ nguyên tắc least privilege xuyên suốt.
- **CI/CD**: GitHub Actions kiểm tra và plan trên mọi Pull Request; `terraform apply` chỉ được kích hoạt thủ công sau khi có review, không tự động merge.

---

### 5. Lộ trình & Mốc triển khai

_Lộ trình 5 tuần, sau 2 tuần học nền tảng AWS_

| Tuần | Mục tiêu |
|---|---|
| **Tuần 1** | Lên ý tưởng, chốt proposal, thiết kế kiến trúc, chuẩn bị môi trường |
| **Tuần 2** | Hoàn thành Luồng 1 — Data Ingestion end-to-end (S3 → SQS → Lambda → OCR → embedding) |
| **Tuần 3** | Hoàn thành Luồng 2 — Realtime Q&A với API có xác thực, cache và Guardrails |
| **Tuần 4** | Hoàn thành Luồng 3 — Monitoring & Alerting, và Luồng 4 — đánh giá RAGAS tự động |
| **Tuần 5** | Tinh chỉnh retrieval, kiểm thử tải, rà soát IAM, tái cấu trúc IaC, demo trực tiếp |

---

### 6. Ước tính ngân sách

Nhờ quyết định lưu vector và BM25 trong DynamoDB (pay-per-request) thay vì triển khai search engine riêng, chi phí nền chạy liên tục chính chỉ còn đến từ **Amazon ElastiCache Serverless** (dung lượng tối thiểu được cấp phát), cộng thêm chi phí gọi Bedrock theo lượt.

**Chi phí ước tính khi hệ thống đang chạy**: ~**2,5 USD/ngày**

ElastiCache Serverless là yếu tố chi phí chính; Lambda, S3, SQS, DynamoDB và API Gateway tính phí theo lượt sử dụng và chiếm tỷ trọng nhỏ hơn nhiều ở quy mô này.

**Các biện pháp kiểm soát chi phí:**

- Dùng DynamoDB thay search engine riêng → tránh thêm chi phí nền cho lớp retrieval.
- Cache exact-match → giảm gọi Bedrock lặp lại cho câu hỏi giống nhau.
- Script teardown/rebuild riêng → phá hủy toàn bộ hạ tầng (kèm sao lưu) khi không dùng, tránh phát sinh chi phí cố định hàng tháng.
- AWS Budget alert → giám sát chi tiêu độc lập với vòng đời hạ tầng chính.

---

### 7. Đánh giá rủi ro

Bất kỳ dự án kỹ thuật nào cũng có rủi ro. Điều quan trọng là nhận diện chúng sớm và có kế hoạch ứng phó rõ ràng, thay vì để đến khi xảy ra mới xử lý.

#### Ma trận rủi ro

| Rủi ro | Ảnh hưởng | Xác suất |
|---|---|---|
| Chậm được cấp quyền truy cập model Bedrock | Trung bình | Trung bình |
| Chất lượng retrieval thấp (hallucination, ngữ cảnh không khớp) | Cao | Trung bình |
| Vượt ngân sách do ElastiCache Serverless chạy nền | Trung bình | Thấp |
| Cấu hình sai quyền IAM giữa các dịch vụ | Cao | Thấp |
| Lambda bị giới hạn concurrency khi tải cao | Trung bình | Thấp |

#### Chiến lược giảm thiểu

- **Bedrock**: Gửi yêu cầu cấp quyền truy cập model ngay trong giai đoạn chuẩn bị môi trường, trước khi code phụ thuộc vào nó.
- **Chất lượng retrieval**: Xây dựng vòng đánh giá RAGAS sớm — để khi chất lượng giảm, ta có con số cụ thể để tinh chỉnh (kích thước chunk, trọng số hybrid search), thay vì chỉ cảm nhận.
- **Chi phí**: AWS Budget alert + script phá hủy hạ tầng khi không dùng.
- **IAM**: Áp dụng least-privilege từ đầu, thực hiện một đợt rà soát quyền hạn riêng trước khi bàn giao.
- **Concurrency**: Kiểm thử tải trước buổi demo để phát hiện giới hạn sớm và có phương án mở rộng sẵn.

#### Kế hoạch dự phòng

- Nếu cấp quyền Bedrock bị chậm → tiếp tục phát triển hạ tầng bằng lệnh gọi mock, tích hợp model thật khi được cấp quyền.
- Nếu chi phí tiệm cận ngưỡng → chạy ngay script teardown để phá hủy các tài nguyên không thiết yếu.
- Nếu chất lượng retrieval không thể cải thiện đủ trong thời gian cho phép → ghi nhận rõ ràng khoảng cách và đưa vào danh sách việc tiếp theo, thay vì âm thầm bàn giao hệ thống chưa đạt yêu cầu.

---

### 8. Kết quả kỳ vọng

#### Về mặt kỹ thuật

- Tự động hóa việc tiếp nhận tài liệu và OCR, thay thế hoàn toàn xử lý thủ công.
- Câu trả lời được cache trả về trong dưới một giây với câu hỏi lặp lại, so với vài giây cho một lượt gọi Bedrock mới.
- Đường đánh giá RAGAS hàng ngày (**EventBridge → Lambda → S3 / CloudWatch**) được thiết kế để thay kiểm tra cảm tính bằng số liệu — **bàn giao vòng này ở mức Partial** cho tới khi gate container và scorer chạy live (xem §9).
- Cảnh báo vận hành thời gian thực, phân loại theo mức nghiêm trọng, giúp rút ngắn thời gian phát hiện sự cố.

#### Giá trị dài hạn

Dự án này không chỉ là một bài tập kỹ thuật. Kết thúc dự án, tôi kỳ vọng có được:

- Một **kiến trúc tham chiếu** hoàn chỉnh, có tài liệu, có thể tái sử dụng cho các hệ thống GenAI Serverless trên AWS.
- Kinh nghiệm thực tế với **Infrastructure as Code (Terraform)** và thiết kế theo hướng sự kiện (event-driven).
- Một nền tảng có thể mở rộng cho các bài toán quản lý tri thức doanh nghiệp rộng hơn trong tương lai.

---

### 9. Trạng thái bàn giao so với Proposal _(đối chiếu trung thực)_

Cuối kỳ thực tập, phạm vi được theo dõi theo **Done / Partial / Deferred** để mentor thấy rõ phần đã chứng minh trên hệ thống thật và phần mới ở mức thiết kế.

| Hạng mục | Trạng thái | Ghi chú / bằng chứng trong báo cáo |
|---|---|---|
| Luồng 1 — Data Ingestion (S3 → SQS → OCR → embedding) | **Done** | [5.3](../5-Workshop/5.3-Data-Ingestion/), E2E [5.3.6](../5-Workshop/5.3-Data-Ingestion/5.3.6-End-To-End-Testing/) |
| Luồng 2 — Realtime Q&A (Cognito, cache, Guardrails, hybrid search) | **Done** | [5.4](../5-Workshop/5.4-Realtime-QA/), E2E [5.4.8](../5-Workshop/5.4-Realtime-QA/5.4.8-End-To-End-Testing/) |
| Luồng 3 — Monitoring & Alerting (SNS, alarm, Slack) | **Done** | [5.5](../5-Workshop/5.5-Monitorning/), E2E [5.5.4](../5-Workshop/5.5-Monitorning/5.5.4-End-to-End-Testing/) |
| Luồng 4 — Đánh giá RAGAS hàng ngày | **Partial** | Terraform + runner skeleton ([5.6](../5-Workshop/5.6-RAGAS/)); gate image / điểm live chưa đóng |
| Frontend console (`ui/index.html`) | **Done** | [5.7](../5-Workshop/5.7-Frontend/) |
| Backend deep-dive & unit test | **Done** | [5.8](../5-Workshop/5.8-Backend/) |
| CI/CD (plan trên PR, deploy thủ công) | **Done** | [5.9](../5-Workshop/5.9-CICD/) |
| API rate-limiting | **Deferred** | Góp ý mentor Tuần 7 |
| Bộ dữ liệu đánh giá RAGAS lớn hơn / scorer production-hardened | **Deferred** | Phụ thuộc hoàn tất push image Flow 4 + kiểm chứng điểm thật |

**Cách đọc Partial ở Luồng 4:** kiến trúc, hình dạng IAM, thiết kế EventBridge và alarm đã được tài liệu hóa; đường chấm điểm vẫn là skeleton — **không** trình bày như vòng chất lượng hàng ngày đã chạy ổn định cho tới khi có bằng chứng (ECR image, invoke, kết quả S3).
