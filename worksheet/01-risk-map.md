| Trường                 | Điền vào đây                                                                                                                                                                                                           |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Họ tên                 | Trần Xuân Trường                                                                                                                                                                                                       |
| Mã học viên            | 2A202600321                                                                                                                                                                                                            |
| Track number           | Track 02                                                                                                                                                                                                               |
| Tên track              | Trợ lý đặt vé và chăm sóc khách hàng hàng không                                                                                                                                                                        |
| Vì sao chọn track này? | Các hãng hàng không luôn phải sử lý hỗ trợ lượng khách lớn trước, trong và sau chuyến bay, việc ứng dụng AI là điều cần thiết để giảm tải chi phí chăm sóc khách hàng, cũng như tối ưu được trải nghiệm của người dùng |

## 2. Scenario — bound use case

| Trường                                                                | Điền vào đây                                                                  |
| --------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| **System / workflow** — AI làm gì cụ thể? AI KHÔNG được làm gì?       | Trợ lý AI đặt vé, trả lời các câu hỏi, tư vấn chuyến bay cho khách hàng       |
| **User** — ai dùng trực tiếp? Role/background/giai đoạn của họ là gì? | Nhân viên CSHK, khách đặt vé                                                  |
| **Context** — dùng ở đâu, lúc nào, qua kênh nào?                      | Trang web chính thức                                                          |
| **Real-work consequence** — nếu AI sai thì ai mất gì?                 | không đặt được vé, đặt vé ảo, vé không chính xác, thông tin bị sai, khiếu nại |

## 3. Failure candidates + layer mapping

### 5 layer để chọn

| Layer            | Nghĩa ngắn                              | Failure thường gặp                      |
| ---------------- | --------------------------------------- | --------------------------------------- |
| **Input**        | Prompt, dữ liệu, tài liệu nguồn, RAG    | Thiếu nguồn chính thức nên AI bịa       |
| **Model**        | Câu trả lời thô từ mô hình              | Nịnh user, đoán, trả lời quá tự tin     |
| **UI**           | Cách câu trả lời hiện ra cho user       | User tưởng câu AI là cam kết chính thức |
| **Human review** | Người thật review, fallback, escalation | Case cần người thật nhưng AI vẫn tự xử  |
| **Monitoring**   | Log, audit, feedback sau khi dùng       | Lỗi lặp lại nhưng không ai phát hiện    |

### 8 failure modes tham khảo

| Failure mode        | Nghĩa ngắn                                       |
| ------------------- | ------------------------------------------------ |
| Hallucination       | AI bịa fact, policy, số liệu, deadline           |
| Bias / fairness     | AI đối xử bất công với một nhóm người            |
| Sycophancy          | AI chiều/nịnh user thay vì giữ đúng sự thật      |
| Over-reliance       | User tin AI quá mức và bỏ qua kiểm tra           |
| Harmful advice      | AI đưa lời khuyên vượt vai trò an toàn           |
| Privacy / data leak | AI lưu, lộ, hoặc xử lý dữ liệu nhạy cảm sai cách |
| Escalation failure  | Case cần chuyển người thật nhưng AI vẫn trả lời  |
| Misuse / jailbreak  | User cố dùng AI sai mục đích hoặc bypass rule    |

### Phần bạn cần điền

| Candidate | Failure mode  | Trigger                             | Bad behavior                                                                                                                       | Severity                                                                             | Layer chính | Layer phụ    | Vì sao                                                                                                                                 |
| --------- | ------------- | ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | ----------- | ------------ | -------------------------------------------------------------------------------------------------------------------------------------- |
| C1        | Hallucination | Người dùng hỏi đáp trên chat hỗ trợ | Người dùng vô tình/cố ý hỏi về 1 số thông tin về chuyến ba không tồn tại -> AI vẫn xác nhận đặt vé thành công                      | Critical: air canada đã bị dính case discount tương tự ảnh hưởng xấu tới thương hiệu | UI          | Input        | Dữ liệu đầu vào bị người dùng khai thác, không kiểm tra kỹ chuyến bay thực tế của hệ thống nên AI tự chỉnh sửa làm vừa lòng người dùng |
| C2        | Privacy       | Người dùng hỏi đáp trên chat hỗ trợ | Ứng dụng đưa thông tin nhạy cảm của khách hàng/ chuyến bay cho người dùng không có quyền truy cập                                  | Hight: lộ thông tin gây ảnh hưởng lớn tới uy tín của hãng bay                        | UI          | Monitoring   | Dữ liệu đã được đánh giá là private ên được kiểm tra quyền truy cập trước khi đưa cho người dùng                                       |
| C3        | Bias          | Người dùng hỏi đáp trên chat hỗ trợ | Chatbot ưu tiên giảm giá cho những người thuộc 1 số nước nhất định -> những nước khác có được ưu đãi những không thông báo rõ ràng | Medium: có thể factcheck nhanh trên các quốc gia                                     | UI          | Human review | Với dữl liệu quá mới AI chưa kịp update, cần có human để đảm bảo quyền lợi người dùng                                                  |

## 4. Primary failure deep dive

> Khi khách hàng đặt chuyến bay không tồn tại, AI có xu hướng gợi ý chuyến bay đó, hoặc có vẻ giống chuyến bay người dùng cần thay vì thông báo chuyến bay không tồn tại, gây hậu quả cho hành khách và hãng bay.

## 5. Harm Map

Harm Map giúp nhìn xa hơn direct user: ai bị ảnh hưởng dù không trực tiếp dùng AI, và nếu workflow scale lên thì harm gì xuất hiện.

| Lens                                                                               | Điền vào đây                                                                                                                      |
| ---------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **Direct user** — người dùng trực tiếp AI là ai? Họ thấy gì?                       | Khách đặt vé                                                                                                                      |
| **Affected person** — ai bị ảnh hưởng khi AI sai dù không tự dùng AI?              | Hãng bay                                                                                                                          |
| **Hidden harm** — nếu workflow scale lên nhiều người dùng, hệ quả dài hạn là gì?   | Hãng bay sẽ mặc định AI sẽ xử lý toàn bộ quy trình, nếu xảy ra lỗi hệ thống mạng, không có người thật đảm bảo quy trình chạy tiếp |
| **Case eval naïve sẽ miss** — case rơi giữa category, dễ bị test set thường bỏ sót |                                                                                                                                   |
