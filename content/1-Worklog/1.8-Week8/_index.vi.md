---
title: "Worklog Tuần 8"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:

- Rà soát và tổng hợp lại toàn bộ hành trình 7 tuần: kiến thức nền tảng AWS (Tuần 1-2) và dự án RAG Knowledge Assistant (Tuần 3-7).
- Kiểm tra lại xem toàn bộ hệ thống có còn chạy đúng từ đầu đến cuối hay không, đối chiếu kết quả cuối cùng với đề xuất ban đầu ở Tuần 3.
- Tự đánh giá kiến thức đã tích lũy so với mục tiêu học tập ban đầu, xác định điểm mạnh và những gì còn thiếu.
- Tổng hợp lại worklog và hoàn thiện tài liệu báo cáo thực tập.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc                                                                                                                                                                                                                                                            | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu     |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | ---------------- | -------------------- |
| 2   | **Rà soát kiến thức AWS nền tảng (Tuần 1-2):**<br>- Tự kiểm tra lại các dịch vụ cốt lõi đã học: IAM, VPC, EC2, S3, RDS, Lightsail, Auto Scaling, CloudWatch, Route 53, DynamoDB, ElastiCache, CloudFront.<br>- Thử tự giải thích lại vài kiến trúc (ví dụ: mô hình VPC/Multi-AZ) mà không xem lại note, để chắc chắn là hiểu thật chứ không chỉ quen mặt.                    | 10/08/2026   | 10/08/2026       | Ghi chú cá nhân       |
| 3   | **Rà soát toàn bộ project từ đầu đến cuối:**<br>- Đi lại từng luồng (Ingestion, Realtime QA, Monitoring, Evaluation).<br>- Test lại Luồng 1–3 trên hệ thống thật: upload tài liệu, hỏi đáp, kiểm tra cache hit, xác nhận monitoring còn cảnh báo.<br>- Xác nhận lại Luồng 4 ở mức **Partial** (thiết kế + skeleton; xem Proposal §9 / [5.6](../5-Workshop/5.6-RAGAS/)).                  | 11/08/2026   | 11/08/2026       | Dự án cá nhân        |
| 4   | **Đối chiếu kết quả với đề xuất ban đầu:**<br>- So sánh bàn giao thật với proposal Tuần 3 và sơ đồ kiến trúc.<br>- Công bố bảng **Done / Partial / Deferred** (Luồng 1–3 Done; Luồng 4 Partial; rate-limiting & bộ eval lớn hơn Deferred). | 12/08/2026   | 12/08/2026       | Dự án cá nhân        |
| 5   | **Hoàn thiện tài liệu:**<br>- Gom lại toàn bộ worklog 8 tuần thành một bản báo cáo thực tập mạch lạc.<br>- Cập nhật lại runbook kiến trúc và README theo những thay đổi phát sinh trong quá trình rà soát.<br>- Viết phần "bài học rút ra", bao gồm cả kiến thức AWS nền tảng lẫn kiến trúc Serverless/GenAI.                    | 13/08/2026   | 13/08/2026       | Dự án cá nhân        |
| 6   | **Tổng kết & khép lại tuần:**<br>- Trình bày phần rà soát tổng thể trước nhóm/mentor: đã học được gì, project đi từ đề xuất đến trạng thái sẵn sàng vận hành như thế nào, và nếu làm lại thì sẽ thay đổi điều gì.<br>- Ghi nhận góp ý cuối cùng và phác thảo các hướng phát triển tiếp theo sau kỳ thực tập.                    | 14/08/2026   | 14/08/2026       | Dự án cá nhân        |

### Kết quả đạt được tuần 8:

- **Xác nhận vẫn nhớ vững kiến thức AWS nền tảng:** Rà lại nội dung Tuần 1-2 mà không cần mở note ra xem, tôi nhận thấy mình vẫn nắm chắc các nhóm dịch vụ cốt lõi (Compute, Storage, Networking, Database) — và quan trọng hơn là hiểu được cách chúng kết hợp với nhau thành một kiến trúc hoàn chỉnh. Chính khả năng "ghép nối" này là nền tảng giúp tôi triển khai được project ở Tuần 3-7.

- **Xác nhận Luồng 1–3 vẫn chạy ổn định:** Chạy lại upload → OCR → hỏi đáp → cache → giám sát cho thấy hệ thống ổn định lâu dài chứ không chỉ demo một lần. Luồng 4 cố ý giữ **Partial**.

- **Đối chiếu thẳng thắn với đề xuất ban đầu:** So với proposal Tuần 3, Luồng 1–3 là **Done**; Luồng 4 (RAGAS) là **Partial**; rate-limiting API và bộ dữ liệu đánh giá lớn hơn là **Deferred** — theo dõi ở Proposal §9 thay vì bỏ lặng.

| Hạng mục | Trạng thái |
|---|---|
| Luồng 1 Ingestion | Done |
| Luồng 2 Realtime Q&A | Done |
| Luồng 3 Monitoring | Done |
| Luồng 4 RAGAS | Partial |
| Frontend / Backend / CI/CD | Done |
| API rate-limiting | Deferred |
| Điểm RAGAS production-hardened + dataset lớn hơn | Deferred |

- **Tài liệu dự án đã được tổng hợp đầy đủ:** Runbook, README và worklog 8 tuần đã thống nhất, kèm khai báo Partial cho Luồng 4.

- **Nhìn lại toàn bộ hành trình:** Từ tạo tài khoản AWS ở Tuần 1 đến một stack GenAI có giám sát vận hành — bài học lớn nhất là chuyển tư duy từ "làm cho chạy được" sang "làm cho vận hành được", kể cả kỷ luật đánh dấu vòng đánh giá chưa đóng là Partial thay vì overclaim.
