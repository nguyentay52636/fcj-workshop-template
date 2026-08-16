---
title: "Đánh giá chất lượng RAG với RAGAS"
date: 2024-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Giới thiệu

Luồng 4 tự động đo lường và đánh giá chất lượng câu trả lời của chatbot theo chu kỳ hàng ngày, dùng framework **RAGAS**. Hạ tầng được khai báo tại `modules/evaluation/main.tf`.

{{% notice warning %}}
📌 **Trạng thái: Partial** — Thiết kế Terraform cho Luồng 4 đã có, nhưng `evaluation_runner.py` vẫn là **skeleton/placeholder**, và với `evaluation_image_pushed = false` thì Lambda / lịch EventBridge / alarm RAGAS **chưa chạy đầy đủ trên hạ tầng** cho tới khi hoàn tất gate 2 pha (push image + apply). Khác Luồng 1–3, luồng này **chưa** được chứng minh end-to-end production.
{{% /notice %}}

Luồng gồm 3 phần chính:

- **Hạ tầng (Terraform)** — EventBridge Scheduler gọi thẳng Lambda mỗi ngày (không qua SQS/S3), Lambda đóng gói kiểu container image (khác 2 Lambda kia đóng gói zip), và cơ chế **gate 2 pha** qua biến `evaluation_image_pushed` — bẫy triển khai quan trọng nhất của luồng này.
- **IAM và Alarm chất lượng** — quyền đọc `feedback`/`chat_history`, ghi S3 riêng, và alarm `ragas-faithfulness-low` cảnh báo khi chatbot có dấu hiệu "bịa" câu trả lời.
- **Logic đánh giá (`evaluation_runner.py`)** — lấy dữ liệu hôm qua, chạy 4 metric RAGAS bằng Bedrock làm "giám khảo", ghi kết quả chi tiết vào S3 và publish điểm trung bình lên CloudWatch.

#### Sơ đồ luồng dữ liệu

![Sơ đồ Luồng 4 - RAG Evaluation RAGAS](/images/5-Workshop/5.6-RAGAS/image.png)
_Sơ đồ: EventBridge Scheduler trigger hàng ngày → Lambda RAGAS Evaluation Runner (đọc dữ liệu từ DynamoDB Feedback Store và Chat History) → lưu kết quả vào S3 Evaluation Results._

#### Nội dung chi tiết

1. [Hạ tầng: EventBridge Scheduler và Lambda Container Image](5.6.1-EventBridge-Lambda-Container/)
2. [IAM Permissions và Alarm chất lượng RAG](5.6.2-IAM-Alarm-RAGAS/)
3. [Logic đánh giá RAGAS (evaluation_runner.py)](5.6.3-RAGAS-Evaluation-Logic/)
4. [Kiểm thử và lưu ý triển khai thực tế](5.6.4-Testing-Deployment-Notes/)
