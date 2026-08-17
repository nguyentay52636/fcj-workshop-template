---
title: "Kiểm thử end-to-end"
date: 2026-08-07
weight: 4
chapter: false
pre: " <b> 5.7.4. </b> "
---

Sau [5.7.1](../5.7.1-Frontend-Architecture-Authentication/)–[5.7.3](../5.7.3-Deployment-Hosting/), em mở `ui/index.html`, dán cấu hình từ `terraform output`, đăng nhập user Cognito và chạy các kịch bản giao diện dưới đây trên stack thật.

Giao diện sẵn sàng (trước khi chạy kịch bản):

<div align="center">

![Giao diện 2 cột khi kiểm thử end-to-end](/images/5-Workshop/5.7-Frontend/image.png)

<p style="font-size: 0.85em; color: #666; font-style: italic; margin-top: 8px;">
Hình 5.7.4a. UI idle — form Cognito, chưa chạy ingest / chat
</p>

</div>

**Bằng chứng Pass — nạp tài liệu** (đã tải `.txt`, tab “Luồng nạp tài liệu” hoàn tất; log có login sai rồi login đúng):

<div align="center">

![Luồng nạp tài liệu Pass](/images/5-Workshop/5.7-Frontend/04-ingest-flow-pass.png)

<p style="font-size: 0.85em; color: #666; font-style: italic; margin-top: 8px;">
Hình 5.7.4b. Ingest Pass + login Cognito (IdToken trên nhật ký)
</p>

</div>

**Bằng chứng Pass — hỏi đáp** (đã trả lời, tab “Luồng hỏi đáp” hoàn tất kèm thời gian từng bước):

<div align="center">

![Luồng hỏi đáp Pass](/images/5-Workshop/5.7-Frontend/05-qa-flow-pass.png)

<p style="font-size: 0.85em; color: #666; font-style: italic; margin-top: 8px;">
Hình 5.7.4c. Q&amp;A Pass — grounded, cache miss, hybrid search
</p>

</div>

#### Kịch bản kiểm thử giao diện

| #   | Kịch bản                                                    | Kỳ vọng                                                                       | Kết quả | Hình |
| --- | ----------------------------------------------------------- | ----------------------------------------------------------------------------- | ------- | --- |
| 1   | Đăng nhập với tài khoản Cognito hợp lệ                      | Nhận `IdToken`; chấm trạng thái chuyển xanh; Upload / Gửi bật                 | Pass — nhận token, nút thao tác bật | [5.7.1b–c](../5.7.1-Frontend-Architecture-Authentication/) |
| 2   | Đăng nhập sai mật khẩu                                      | Hiện lỗi rõ ràng, không crash giao diện; nút vẫn tắt                          | Pass — hiện lỗi, nút vẫn tắt | [5.7.1c](../5.7.1-Frontend-Architecture-Authentication/) (log đỏ) |
| 3   | Tải tài liệu `.txt`/`.md` (text)                            | Hiện được nội dung trong ô soạn thảo, sửa tay được trước khi gửi              | Pass — nội dung sửa được trong editor | 5.7.4b |
| 4   | Tải PDF có lớp text sẵn                                     | Ingest xong không qua bước xác nhận OCR, trích xuất bằng `pypdf`              | Pass — không hiện dialog OCR; ingest xong | — |
| 5   | Tải PDF dạng scan (không có text) — chọn **Có**             | Hiện dialog xác nhận OCR → chạy Textract → tiếp tục pipeline                  | Pass — hiện dialog; tiếp tục qua Textract | N/A (chưa chụp dialog) |
| 6   | Tải PDF dạng scan — chọn **Không**                          | Dừng ở trạng thái hủy, không tốn phí Textract                                 | Pass — hủy; không gọi Textract | N/A |
| 7   | Hỏi về nội dung tài liệu vừa tải                            | Trả lời đúng, có tag tên file nguồn khi có                                    | Pass — trả lời grounded, có tag nguồn | 5.7.4c |
| 8   | Hỏi lại y hệt câu vừa hỏi (**cùng phiên**)                  | Tag `cache hit`; chỉ 1 bước sáng ở khu quan sát; phản hồi &lt; 1s             | Pass — cache hit; phản hồi dưới 1s | — |
| 9   | Hỏi câu mơ hồ (“còn cái kia thì sao?”) trong **cùng phiên** | Bước rewrite sáng lên; log in câu đã viết lại                                 | Pass — rewrite sáng; log có câu viết lại | — |
| 10  | Hỏi điều không có trong tài liệu                            | Model từ chối / không bịa tự do                                               | Pass — từ chối / giữ grounded | — |
| 11  | Gửi nhiều câu liên tiếp thật nhanh                          | Có thể gặp 429, hiện icon `⏳` khác với lỗi thường; thử lại sau vài giây được | Pass — 429 kèm `⏳`; thử lại thành công | — |

{{% notice tip %}}
Nếu ingest quá ~90s, xem CloudWatch Logs `document-processor` và độ sâu SQS/DLQ (alarm Luồng 3). Timeout UI cố ý để pipeline kẹt lộ ra thay vì quay mãi.
{{% /notice %}}

#### Kết quả đạt được

- UI một file gọi đủ bốn route Luồng 2 với JWT Cognito thật.
- Upload phân biệt text và nhị phân (base64) cho Textract.
- Animation pipeline phản ánh `trace` / status server, kể cả bước skipped.
- Cache hit và rewrite multi-turn nhìn thấy được mà không cần mở AWS Console.
- Hành vi rate-limit (429) phân biệt được với lỗi cứng nhờ affordance `⏳`.
