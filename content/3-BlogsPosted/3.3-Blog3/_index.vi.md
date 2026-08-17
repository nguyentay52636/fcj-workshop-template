---
title: "Blog 3"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# Từ lý thuyết đến thực thi: Bài toán Data Engineering trong một đồ án GenAI/RAG thực tế

Xin chào mọi người,

Ở bài viết trước, mình đã chia sẻ góc nhìn về việc một Data Engineer nên ưu tiên học những gì trên AWS để tối ưu hóa thời gian và sẵn sàng cho công việc.

Hôm nay, mình muốn chia sẻ kĩ hơn về cách mình đem những tư duy và dịch vụ AWS đó áp dụng trực tiếp vào đồ án thực tế của mình. Mặc dù bài toán chính của đồ án thuộc mảng GenAI / RAG (Retrieval-Augmented Generation), nhưng khi bắt tay vào triển khai kiến trúc hệ thống, mình nhận ra: Có tới 80% sức mạnh và độ ổn định của hệ thống nằm ở khâu Data Engineering.

Dưới đây là bức tranh toàn cảnh về cách mình ứng dụng các kỹ thuật xử lý dữ liệu cloud trong đồ án này.

## 1. Data Ingestion & Event-Driven Pipeline (Luồng xử lý nạp dữ liệu)

Khi xử lý tài liệu đầu vào, nếu chỉ làm theo cách truyền thống là gọi hàm xử lý trực tiếp thì hệ thống rất dễ bị nghẽn (bottleneck) hoặc quá tải. Do đó, mình xây dựng theo hướng Event-Driven Architecture:

- **Tự động kích hoạt (S3 Event Trigger):** Mỗi khi người dùng hoặc Admin upload tài liệu mới (PDF/TXT/Scan) vào Amazon S3 Raw Documents, một sự kiện (S3 Event) sẽ ngay lập tức kích hoạt luồng xử lý mà không cần polling thủ công.
- **Hàng đợi đệm & Phân rã hệ thống (Amazon SQS):** Để hệ thống không bị ngợp khi lượng file đẩy vào quá lớn cùng lúc, mình đưa dữ liệu qua Amazon S3 Event → Amazon SQS (Buffer Queue). Việc sử dụng SQS đóng vai trò làm điểm tựa chịu tải, kết hợp cơ chế Retry tự động và Dead Letter Queue (DLQ) giúp hứng các message lỗi, đảm bảo tuyệt đối không bị mất mát dữ liệu (zero data loss).

## 2. ETL & Unstructured Data Processing (Trích xuất & Biến đổi dữ liệu)

Dữ liệu văn bản phi cấu trúc (Unstructured Data) cần trải qua một quy trình ETL nghiêm ngặt trước khi có thể phục vụ cho các mô hình AI:

- **Trích xuất dữ liệu (OCR):** AWS Lambda sẽ nhận tin nhắn từ SQS, tự động gọi Amazon Textract để thực hiện OCR, trích xuất chính xác văn bản từ các file Scan hay PDF phức tạp.
- **Cấu trúc hóa & Lưu trữ Vector (Chunking & Vectorization):** Dữ liệu sau khi xử lý sẽ được cắt nhỏ (chunking) theo mô hình Parent-Child, sau đó gọi Embedding API để biến đổi thành các vector. Toàn bộ thông tin cấu trúc này được lưu trữ vào Amazon DynamoDB – tối ưu hóa cho các truy vấn Hybrid Search (kết hợp Cosine Similarity và BM25 bằng thuật toán RRF) nhằm đạt độ chính xác cao nhất khi tìm kiếm.

## 3. Low-Latency Data Retrieval & Caching (Tối ưu hóa truy vấn thời gian thực)

Một bài toán cốt lõi của Data Engineering trong các ứng dụng Web/App là độ trễ (latency) và chi phí (cost):

- **Exact-match cache (không phải semantic cache thật):** Mình dùng Amazon ElastiCache Serverless làm lớp cache câu hỏi → câu trả lời, có TTL. Khóa cache là hash của câu hỏi **sau khi chuẩn hóa** (hạ chữ thường, bỏ dấu câu, gộp khoảng trắng). Hỏi **đúng y hệt** (sau normalize) thì trúng cache, khỏi gọi lại LLM. ElastiCache Serverless **không có** module RediSearch/vector nên **không** so khớp “câu gần nghĩa”. UI vẫn có thể ghi nhãn “Semantic cache”, nhưng implementation là exact-match — mình giữ đúng bản chất này thay vì nói quá.
- **Quản lý trạng thái & Phản hồi (Transaction Store):** Toàn bộ lịch sử hội thoại (Chat History) và dữ liệu phản hồi (Feedback Store) được lưu trữ trên DynamoDB – dòng NoSQL Database đáp ứng tốc độ ghi siêu nhanh với độ trễ chỉ tính bằng milisecond.

## 4. Automated Batch Processing & Continuous Evaluation (Pipeline đánh giá — Partial)

Không dừng lại ở luồng dữ liệu thời gian thực, một hệ thống dữ liệu chuẩn mực cần luồng batch để đánh giá chất lượng. **Phần này mình ghi rõ trạng thái Partial**, không nói như đã chạy production:

- **Thiết kế Terraform đã có:** EventBridge Scheduler gọi Lambda đánh giá (đóng gói **container image** trên ECR, khác 2 Lambda zip của ingestion/chat), kết quả dự kiến ghi S3 Evaluation Results và metric RAGAS (Faithfulness, Relevancy, Precision, Recall) lên CloudWatch. Có **gate 2 pha**: apply lần 1 chỉ tạo ECR rỗng + IAM; phải build/push image rồi bật cờ mới tạo Lambda/lịch/alarm.
- **Chưa chạy đủ trên hạ tầng:** `evaluation_runner.py` vẫn là **skeleton**; cờ `evaluation_image_pushed = false` nên Lambda / lịch EventBridge / alarm `ragas-faithfulness-low` **chưa live**. Widget RAGAS trên dashboard Luồng 3 cố ý trống cho đến khi job chạy ít nhất một lần. Đây là phần mình chủ động để Partial, không gộp với Luồng 1–3 đã test E2E.

## 5. Data Observability & Monitoring (Giám sát dòng chảy dữ liệu)

Cuối cùng, dữ liệu chạy trong hệ thống Cloud cần phải "nhìn thấy được" (observable) để kịp thời phát hiện sự cố:

- **Tập trung hóa Log & Metric:** Amazon CloudWatch thu thập log và metric pipeline đang chạy thật: độ sâu DLQ, tỉ lệ 5xx API Gateway, cache hit/miss (EMF từ chat-engine). Các custom metric RAGAS (Faithfulness, Relevancy, Precision) **đã khai báo trên dashboard** nhưng **chưa có số** cho tới khi luồng đánh giá (mục 4) chạy — khớp trạng thái Partial.
- **Cảnh báo theo severity:** Warning (email qua SNS `alerts-info`) và Critical (SNS `alerts-critical`, Slack qua AWS Chatbot khi đã cấu hình). PagerDuty để sẵn dạng tùy chọn, **chưa bật mặc định**.

## Ba "điểm sáng" Data Engineering mình rút ra từ đồ án

Nếu bạn cũng đang chuẩn bị làm đồ án hoặc project cá nhân, đây là 3 tư duy Data Engineering mà mình thấy đáng giá nhất khi đưa vào bài toán:

- **Tính Decoupled (Tách biệt hệ thống):** Khâu nạp dữ liệu (Ingestion) và khâu truy vấn (Serving) hoạt động độc lập qua hàng đợi SQS và DynamoDB. Dù Admin có upload hàng ngàn file PDF cùng lúc thì trải nghiệm chat của người dùng cuối vẫn hoàn toàn mượt mà, không hề bị ảnh hưởng.
- **Xử lý bất đồng bộ (Asynchronous Serverless):** Việc kết hợp S3 Event + SQS + Lambda giúp hệ thống hoạt động bất đồng bộ hoàn toàn. Tài nguyên máy tính chỉ bật lên khi có dữ liệu chạy qua, giúp tối ưu hóa 100% chi phí vận hành Cloud.
- **Tối ưu hóa truy vấn dữ liệu (Query Optimization):** Thay vì gọi LLM mọi lần, mình kết hợp **exact-match cache** (ElastiCache, hash câu đã chuẩn hóa) và Hybrid Search (BM25 + vector, RRF). Biết giới hạn của Serverless Redis (không RediSearch) rồi chọn đúng loại cache — cũng là một phần của tư duy Data Engineer, không kém việc vẽ kiến trúc “semantic” cho đẹp.

## Lời kết

Đồ án này đã giúp mình chứng minh một điều: Làm Data Engineer không chỉ là viết các câu lệnh SQL hay chạy script Spark, mà là khả năng thiết kế một hạ tầng Cloud tin cậy, tự động hóa luồng dữ liệu và tối ưu chi phí vận hành cho hệ thống.

Hy vọng góc nhìn chia sẻ từ đồ án thực tế này sẽ cho các bạn thấy một bức tranh sinh động hơn về cách các dịch vụ AWS như S3, SQS, Lambda, DynamoDB, ElastiCache phối hợp với nhau trong công việc.
##Link
<https://www.facebook.com/groups/awsstudygroupfcj/permalink/2240430060055287/#>
