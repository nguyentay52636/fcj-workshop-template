---
title: "Event 3"
date: 2026-08-08
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Báo cáo tóm tắt: “Agent Forge - Deepdive Day 3”

### 1. Thông tin sự kiện
- **Thời gian**: 9:00 – 12:00, Thứ Bảy, 15/08/2026
- **Địa điểm**: Tầng 26, Bitexco Financial Tower, 2 Hải Triều, TP. Hồ Chí Minh
- **Vai trò**: Người tham dự

### 2. Diễn giả
- **Nghia Tran** — Agentic SA
- **Anh Pham** — Cloud Consultant, G-AsiaPacific Vietnam

---

### 3. Nội dung chính

#### Phần lý thuyết

##### Memory
Nếu không có memory, agent dễ “quên” hội thoại khi context window đầy. Buổi học đi vào cách giữ ngữ cảnh hữu ích giữa các lượt chat:

- **Short-term memory**: lịch sử hội thoại gần, lấy lại nhanh khi cần
- **Long-term memory**: rút insight từ chat cũ, lưu dạng vector
- **Chiến lược**: Summary, User Preference, Semantic, Episodic
- **Namespace**: cấu trúc kiểu `/Strategy/Actor/Session` để thu hẹp phạm vi tìm kiếm, giảm token và tăng tốc retrieval

##### Evaluations
Trước khi đưa agent lên production cần cách kiểm tra câu trả lời có đúng, hữu ích, an toàn — và bắt được hallucination hay tool call sai.

- **On-demand**: đánh giá trong lúc đang build
- **Online**: theo dõi liên tục trên production qua telemetry

Có thể chấm ở 3 mức:
- **Session** — cả phiên hội thoại
- **Trace** — một câu trả lời
- **Span** — một lần gọi tool và tham số

Hệ thống dùng **Judge** để phân tích hoạt động của agent; kết quả đẩy sang Observability để team theo dõi và can thiệp khi cần.

##### Observability
Nói ngắn gọn: biết agent đã làm gì, làm thế nào, và tốn bao nhiêu.

- **Logs** — chuyện gì xảy ra
- **Traces** — đường đi của request
- **Metrics** — latency, chi phí token, tỷ lệ lỗi

Ngoài ra còn OpenTelemetry, cảnh báo, và cấu trúc `Session → Trace → Span`.

##### Các thành phần AgentCore
- **Registry** — nơi tái sử dụng skill, tool, API (Admin / Publisher / Consumer)
- **Harness** — khung tối giản để tạo agent từ Model + System Prompt + Tools
- **Tools** — để agent gọi hệ thống ngoài và API realtime
- **Payments** — agent thực hiện thanh toán (Stripe, Coinbase)
- **Optimization** — dùng dữ liệu eval/observability cho A/B test, red teaming, vòng cải tiến
- **Policy** — lớp kiểm soát: human-in-the-loop, Cedar, chế độ strict/permissive, least privilege

#### Phần thực hành
Hướng dẫn dùng Agent SDK, cấu hình AWS Bedrock và CLI: tạo project, deploy, rồi test agent trên AWS.

---

### 4. Những gì em rút ra được

Day 2 giúp em hình dung rõ hơn Memory, Evaluations và Observability gắn với nhau thế nào khi vận hành agent thật, không chỉ demo. Các khối AgentCore (Registry, Harness, Tools, Policy, Optimization) cũng dễ hiểu hơn khi thấy chúng kết nối để tái sử dụng, kiểm soát và cải tiến. Least privilege và human-in-the-loop là điểm em nhớ nhất vì mang tính an toàn thực tế. Buổi lab giúp em quen hơn với Agent SDK + Bedrock + CLI theo flow tạo → deploy → test.
