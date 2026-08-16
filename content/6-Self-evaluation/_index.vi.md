---
title: "Tự đánh giá"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

Trong suốt thời gian thực tập tại **Công ty TNHH Amazon Web Services Việt Nam** từ **22/06/2026** đến **15/08/2026**, tôi đã có cơ hội học hỏi, rèn luyện và áp dụng kiến thức đã được trang bị tại trường vào môi trường làm việc thực tế.  
Tôi đã tham gia dự án **RAG on AWS**, qua đó cải thiện kỹ năng **lập trình, triển khai dự án, giao tiếp**.

Về tác phong, tôi luôn cố gắng hoàn thành tốt nhiệm vụ, tuân thủ nội quy, và tích cực trao đổi với đồng nghiệp để nâng cao hiệu quả công việc.

Để phản ánh một cách khách quan quá trình thực tập, tôi xin tự đánh giá bản thân dựa trên các tiêu chí dưới đây:

| STT | Tiêu chí                            | Mô tả                                                                                            | Tốt | Khá | Trung bình | Minh chứng trong kỳ thực tập |
| --- | ----------------------------------- | ------------------------------------------------------------------------------------------------ | --- | --- | ---------- | --- |
| 1   | **Kiến thức và kỹ năng chuyên môn** | Hiểu biết về ngành, áp dụng kiến thức vào thực tế, kỹ năng sử dụng công cụ, chất lượng công việc | ☐   | ✅  | ☐          | Xây được Luồng 1–3 (ingestion, Q&A, monitoring) bằng Terraform; Luồng 4 RAGAS vẫn Partial (skeleton). |
| 2   | **Khả năng học hỏi**                | Tiếp thu kiến thức mới, học hỏi nhanh                                                            | ☐   | ✅  | ☐          | Học Bedrock, Cognito, hybrid search và gate ECR 2 pha qua tài liệu + debug thực tế. |
| 3   | **Chủ động**                        | Tự tìm hiểu, nhận nhiệm vụ mà không chờ chỉ dẫn                                                  | ✅  | ☐  | ☐          | Tự truy vết lỗi `/documents-decision` **504** tới thiếu VPC endpoint (ghi ở [5.10.1](../5-Workshop/5.10-System-Testing/5.10.1-Manual-E2E-Testing/)). |
| 4   | **Tinh thần trách nhiệm**           | Hoàn thành công việc đúng hạn, đảm bảo chất lượng                                                | ✅  | ☐  | ☐          | Duy trì worklog hàng tuần và giữ module IaC khớp với những gì thực sự chạy trên AWS. |
| 5   | **Kỷ luật**                         | Tuân thủ giờ giấc, nội quy, quy trình làm việc                                                   | ✅  | ☐  | ☐          | Theo nhịp check-in và rà IAM least-privilege trước các tuần demo. |
| 6   | **Tính cầu tiến**                   | Sẵn sàng nhận feedback và cải thiện bản thân                                                     | ☐   | ✅  | ☐          | Nhận góp ý Tuần 7 (rate-limiting, bộ eval lớn hơn) và ghi Deferred, không bỏ qua. |
| 7   | **Giao tiếp**                       | Trình bày ý tưởng, báo cáo công việc rõ ràng                                                     | ✅  | ☐  | ☐          | Viết báo cáo song ngữ và trình bày trạng thái Partial của Luồng 4 khi tổng kết demo. |
| 8   | **Hợp tác nhóm**                    | Làm việc hiệu quả với đồng nghiệp, tham gia nhóm                                                 | ✅  | ☐  | ☐          | Phối hợp OAuth Slack Chatbot với admin workspace; chia sẻ note debug với nhóm. |
| 9   | **Ứng xử chuyên nghiệp**            | Tôn trọng đồng nghiệp, đối tác, môi trường làm việc                                              | ✅  | ☐  | ☐          | Không commit credential; dùng Budget alert và script teardown để kiểm soát chi phí. |
| 10  | **Tư duy giải quyết vấn đề**        | Nhận diện vấn đề, đề xuất giải pháp, sáng tạo                                                    | ☐   | ✅  | ☐          | Mạnh ở infra/debug (504, DLQ, race cache); yếu hơn ở việc đóng bằng chứng RAGAS định lượng. |
| 11  | **Đóng góp vào dự án/tổ chức**      | Hiệu quả công việc, sáng kiến cải tiến, ghi nhận từ team                                         | ☐   | ✅  | ☐          | Đóng góp stack RAG chạy được + báo cáo; còn thiếu evidence Flow 4 production-hardened. |
| 12  | **Tổng thể**                        | Đánh giá chung về toàn bộ quá trình thực tập                                                     | ✅  | ☐  | ☐          | Bàn giao ổn định Luồng 1–3 + tài liệu; điểm cần cải thiện là hoàn tất vòng bằng chứng evaluation. |

### Cần cải thiện

- Tăng tốc độ tiếp thu và biến kiến thức AWS/GenAI mới thành lần chạy đã chứng minh trên hạ tầng sớm hơn (đặc biệt push image + invoke Luồng 4).
- Thực hành sâu hơn mức “demo chạy được” — mỗi khi tuyên bố Done cần kèm bằng chứng Console.
- Chủ động giao tiếp ownership công việc trong nhóm sớm hơn để tránh khoảng trống khi làm song song.
- Rèn thói quen đọc kỹ yêu cầu và checklist trước khi đóng mục báo cáo (bài học cụ thể: ảnh gãy và đoạn overclaim RAGAS trước đây).
