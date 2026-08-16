---
title: "Worklog Tuần 7"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7:

- Rà soát lại toàn bộ 4 luồng dưới góc nhìn tổng thể, xử lý các điểm còn thô ráp phát hiện ở tuần trước (đặc biệt là chất lượng retrieval), viết tài liệu vận hành, và demo chính thức trước nhóm/mentor.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc                                                                                                                                                                                                                                                                                                                                                                                                                                           | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                                                                                        |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------------------------------------------------------------------- |
| 2   | **Cải thiện chất lượng Retrieval:**<br>- Đầu tuần họp với nhóm rà lại vấn đề Faithfulness thấp phát hiện tuần trước.<br>- Điều chỉnh lại tham số Hybrid Search trong OpenSearch (tăng trọng số cho vector search so với BM25 từ tỷ lệ 50/50 lên 70/30).<br>- Giảm kích thước chunk con từ 300 xuống 200 token để tăng độ chính xác khi tìm kiếm.<br>- Chạy lại 5 câu hỏi từng bị điểm thấp — 4/5 câu đã cải thiện rõ rệt.                           | 03/08/2026   | 03/08/2026      | Dự án cá nhân                                                                                         |
| 3   | **Kiểm thử tải (load test) sơ bộ & Rà soát IAM:**<br>- Dùng script gửi 50 request đồng thời tới API Gateway để xem hệ thống phản ứng ra sao.<br>- Phát hiện ElastiCache Serverless xử lý tốt nhưng Lambda Chat Engine bị giới hạn concurrency mặc định (1000), không phải vấn đề thực tế ở quy mô hiện tại nhưng ghi chú lại để lưu ý khi scale.<br>- Rà soát lại toàn bộ IAM Policy theo nguyên tắc least privilege lần cuối trước khi hoàn thiện. | 04/08/2026   | 04/08/2026      | [AWS Lambda Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html) |
| 4   | **Hoàn thiện Terraform & IaC:**<br>- Dọn dẹp lại toàn bộ mã Terraform, tách module rõ ràng theo từng luồng (ingestion, chat-api, monitoring, evaluation), viết README hướng dẫn deploy từ đầu.<br>- Đây là phần khá tốn thời gian vì trong lúc làm gấp ở các tuần trước, một số resource được tạo thủ công qua Console chưa được đưa vào code — phải rà lại và import ngược vào Terraform state.                                                    | 05/08/2026   | 05/08/2026      | [Terraform Documentation](https://developer.hashicorp.com/terraform/docs)                             |
| 5   | **Viết tài liệu vận hành & runbook:**<br>- Soạn tài liệu mô tả kiến trúc tổng thể, hướng dẫn xử lý khi có alert (ví dụ: DLQ Depth > 0 thì kiểm tra message lỗi ở đâu, cách requeue thủ công), và checklist bảo trì định kỳ.<br>- Chuẩn bị slide và kịch bản demo cho buổi trình bày cuối kỳ.                                                                                                                                                        | 06/08/2026   | 06/08/2026      | Dự án cá nhân                                                                                         |
| 6   | **Demo chính thức & tổng kết:**<br>- Trình bày dự án trước nhóm/mentor: demo live upload → hỏi đáp → alert giả lập, kèm walkthrough trung thực Luồng 4 (RAGAS) ở mức **Partial** (thiết kế + skeleton; chưa phải báo cáo điểm hàng ngày đã hoàn chỉnh).<br>- Ghi nhận góp ý (mở rộng bộ câu hỏi đánh giá, cân nhắc rate-limiting API).<br>- Tổng kết hành trình 5 tuần và hướng phát triển tiếp theo.                                                                          | 07/08/2026   | 07/08/2026      | Dự án cá nhân                                                                                         |

### Kết quả đạt được tuần 7 — và tổng kết dự án:

- **Tinh chỉnh retrieval có mục tiêu đo lường:** Điều chỉnh Hybrid Search (Vector 70 / BM25 30) và chunk con (300 → 200 tokens) dựa trên case lỗi định tính và **vòng phản hồi RAGAS đã thiết kế** — đồng thời thừa nhận Flow 4 vẫn Partial cho tới khi có bằng chứng invoke live ([5.6](../5-Workshop/5.6-RAGAS/)).
- **Kiểm thử tải & Tối ưu bảo mật:** Kiểm thử tải sơ bộ với 50 request đồng thời khẳng định ElastiCache Serverless xử lý tốt, đồng thời giúp phát hiện giới hạn concurrency mặc định của Lambda Chat Engine để lưu ý khi scale. Rà soát lại toàn bộ IAM Policy theo nguyên tắc least privilege.
- **Hoàn thiện Infrastructure as Code (IaC):** Dọn dẹp mã Terraform, tách module rõ ràng theo 4 luồng hệ thống (ingestion, chat-api, monitoring, evaluation), import ngược các tài nguyên tạo thủ công qua Console vào Terraform state và viết README hướng dẫn deploy từ đầu.
- **Tài liệu vận hành & Demo:** Soạn runbook vận hành; demo live ingestion, chat và alerting; Luồng 4 trình bày như kiến trúc + bước tiếp theo, không như báo cáo chất lượng tự động đã hoàn tất.

---

### Tổng kết hành trình 5 tuần dự án RAG Knowledge Assistant:

Tuần cuối này không phát sinh nhiều kiến thức mới, nhưng lại là tuần "vất vả" theo một cách khác — quay lại rà soát những gì đã làm vội ở các tuần trước để đưa hệ thống về trạng thái ổn định, có thể vận hành lâu dài chứ không chỉ chạy được cho demo. Việc điều chỉnh tỷ lệ Hybrid Search và giảm kích thước chunk hướng tới vòng phản hồi định lượng — trong đó RAGAS được **thiết kế và ghi nhận Partial**, không bị “lặng lẽ” coi như đã hoàn tất.

Nhìn lại toàn bộ 5 tuần triển khai (từ kickoff ở Tuần 3 đến hoàn thiện ở Tuần 7), dự án RAG Knowledge Assistant đã đi qua thiết kế kiến trúc, pipeline dữ liệu, tích hợp mô hình AI, mở API, và giám sát vận hành. Vòng tự động đánh giá chất lượng (RAGAS) là hạng mục **Partial có chủ đích** — minh bạch hơn là tuyên bố đủ 4 luồng đã chạy full.