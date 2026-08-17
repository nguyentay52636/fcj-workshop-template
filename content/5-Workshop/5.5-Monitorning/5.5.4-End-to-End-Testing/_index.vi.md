---
title: "Kiểm thử end-to-end"
date: 2026-08-03
weight: 4
chapter: false
pre: " <b> 5.5.4. </b> "
---

Sau khi hoàn tất SNS ([5.5.1](../5.5.1-SNS-2-Channels-By-Severity/)), Alarms ([5.5.2](../5.5.2-CloudWatch-Alarms/)) và Dashboard ([5.5.3](../5.5.3-Dashboard-Custom-Metrics/)), bước cuối là xác nhận chuỗi cảnh báo thực sự chạy — phần hay bị bỏ qua vì hạ tầng "trông ổn" ngay cả khi subscription email còn Pending hoặc Chatbot chưa gắn Slack.

Hạ tầng Luồng 3 đã apply (2 SNS topic, 4 alarm, dashboard 9 widget). Sơ đồ đối chiếu với Console:

<div align="center">

![Sơ đồ chi tiết Luồng 3 - Monitoring và Alert](/images/5-Workshop/5.5-Monitorning/image.png)

<p style="font-size: 0.85em; color: #666; font-style: italic; margin-top: 8px;">
Hình 5.5.4a. Kiến trúc giám sát đã khai báo — SNS theo severity, CloudWatch Alarm, Dashboard overview
</p>

</div>

#### Kịch bản test và kết quả

| #   | Kịch bản                                                                      | Kỳ vọng                                                                                                                   | Kết quả |
| --- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- | ------- |
| 1   | SNS Console → subscription `alerts-info`                                      | Trạng thái `Confirmed` (không phải `Pending confirmation`)                                                                | Pass — đã bấm mail xác nhận; subscription không còn Pending |
| 2   | AWS Chatbot → Slack channel configuration                                     | Đúng workspace/channel; nếu `slack_*` = `NONE` thì resource Chatbot không tồn tại (đúng thiết kế)                         | Pass theo tfvars — Chatbot chỉ tạo khi Slack đã điền đủ |
| 3   | Giả lập lỗi Lambda (throw exception liên tục vài phút, vượt ngưỡng Errors)    | `lambda-errors` → `ALARM`, nhận email qua `alerts-info`                                                                   | Pass (giả lập tay) — alarm chuyển ALARM rồi về OK |
| 4   | Gây lỗi 5xx phía API/Lambda (ví dụ transient fault trong chat-engine trả 500) | `apigw-5xx` → `ALARM` khi tỷ lệ 5xx > ngưỡng %, nhận email                                                                | Pass (giả lập tay) — dùng lỗi 5xx phía server, không dùng 4xx |
| 5   | Gây `ThrottlingException` (spam request Bedrock hoặc mock lỗi trong code)     | Log filter bắt chuỗi, `bedrock-throttle` → `ALARM`, tin nhắn trên Slack (`alerts-critical`)                               | Pass khi Slack đã cấu hình; nếu `slack_*` = `NONE` thì chỉ thấy alarm trên Console |
| 6   | Đẩy 1 message lỗi vào DLQ (ingestion DLQ hoặc function DLQ)                   | `dlq-depth` → `ALARM` ngay khi depth > 0, tin nhắn Slack                                                                  | Pass (giả lập tay) — depth > 0 là Critical |
| 7   | Mở dashboard `${name_prefix}-overview`                                        | 7 widget AWS có dữ liệu sau khi có traffic; Cache Hit Rate sau vài lần gọi `/chat`; RAGAS sau khi evaluation chạy ≥ 1 lần | Partial — widget 1–8 có dữ liệu sau traffic UI; widget RAGAS trống vì Luồng 4 chưa deploy |
| 8   | Dừng giả lập lỗi sau khi đã `ALARM`                                           | Alarm tự về `OK` khi hết vi phạm ngưỡng — không cần reset thủ công                                                        | Pass — `treat_missing_data = notBreaching`, tự về OK |

{{% notice tip %}}
Kịch bản 4: request sai path/method thường ra **4xx**, không kích hoạt `apigw-5xx`. Cần lỗi phía server (5xx) hoặc tỷ lệ 5xx đủ lớn trong cửa sổ 5 phút.
{{% /notice %}}

{{% notice note %}}
📌 **Bằng chứng hình Console** (Alarm OK/ALARM, email SNS hoặc Slack critical, dashboard overview) **chưa gắn file riêng**. Mentor đối chiếu Terraform + sơ đồ trên với [5.5.1](../5.5.1-SNS-2-Channels-By-Severity/)–[5.5.3](../5.5.3-Dashboard-Custom-Metrics/). Khi chụp, đặt đúng tên: `alarm-ok-or-alarm.png`, `sns-email-or-slack.png`, `dashboard-overview.png` trong `static/images/5-Workshop/5.5-Monitorning/`.
{{% /notice %}}

#### Kết quả đạt được

- Hai kênh SNS theo severity — Warning (email) và Critical (Slack) — giảm alarm fatigue.
- `bedrock-throttle` tận dụng log Lambda (Luồng 2) thay vì hạ tầng theo dõi throttle riêng.
- `dlq-depth` gộp cả 2 tầng DLQ (Luồng 1) — không bỏ sót message ở một tầng.
- Dashboard 9 widget: 7 metric AWS + cache hit rate (EMF) + RAGAS (`put_metric_data`) — widget RAGAS cố ý trống cho đến khi Luồng 4 chạy.
- `treat_missing_data = "notBreaching"` tránh cảnh báo giả khi môi trường ít traffic.
