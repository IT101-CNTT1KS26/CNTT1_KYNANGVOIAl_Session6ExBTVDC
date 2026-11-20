# THIẾT KẾ GIẢI PHÁP PROMPTING: TỰ ĐỘNG HÓA FEYNMAN TUTOR & RUBRIC CHẤM BÀI TẠI TECHSHOP



## BƯỚC 1: THIẾT KẾ GIẢI PHÁP (I/O & ERROR HANDLING)

### 1. Bối cảnh và Bài toán

Tại **TechShop**, đội ngũ nhân sự mới và thực tập sinh cần đào tạo nhanh về quy trình vận hành, chính sách đổi trả và kỹ thuật xử lý đơn hàng. Nếu chỉ giao tài liệu tĩnh, học viên dễ rơi vào *Illusion of Explanatory Depth* (ảo tưởng hiểu sâu), còn mentor thì mất nhiều thời gian chấm bài kiểm tra thực hành thủ công.

Giải pháp tự động hóa kết hợp **Feynman Prompt (Gia sư giải thích đơn giản & kiểm tra lỗ hổng)** với **Rubric chấm bài tự động**, đồng thời thiết lập câu lệnh điều kiện xử lý khi dữ liệu đầu vào bị trống, lạc đề hoặc sai lệch.

### 2. Xác định Đầu vào & Đầu ra (Input / Output)

* **Input (Dữ liệu đầu vào):**
1. `topic_rule`: Chủ đề hoặc quy định chuẩn cần nắm vững (ví dụ: *Chính sách đổi trả hàng lỗi & quy trình xử lý khiếu nại đơn hàng tại TechShop*).
2. `learner_explanation`: Bài giải thích/trình bày lại của học viên bằng ngôn ngữ đời thường (có thể bị bỏ trống, viết quá ngắn, hoặc chứa thông tin sai lệch).
3. `rubric_criteria`: Thang điểm đánh giá chuẩn (tổng 10 điểm, chia theo các tiêu chí: *Độ chính xác kiến thức*, *Tính dễ hiểu kiểu Feynman*, *Khả năng xử lý tình huống ngoại lệ*).


* **Output (Cấu trúc đầu ra mong muốn):**
* **Phần 1 — JSON đánh giá & bắt lỗi (`evaluation_json`):** Ghi nhận trạng thái kiểm tra dữ liệu (`status`), điểm từng tiêu chí, tổng điểm, danh sách lỗ hổng kiến thức (`knowledge_gaps`) và các trường bị thiếu/sai lệch (`error_flags`).
* **Phần 2 — Phản hồi Gia sư Feynman (`feynman_tutor_feedback`):** Chỉ ra chỗ hiểu sai bằng ngôn ngữ đơn giản, đặt câu hỏi gợi mở/giải thích lại như gia sư kèm ví dụ dễ hiểu để học viên tự lấp đầy lỗ hổng.



### 3. Bẫy dữ liệu & Điều kiện xử lý lỗi (Error Handling)

* **Dữ liệu trống hoặc quá sơ sài** (`learner_explanation` rỗng, chỉ có vài từ vô nghĩa, hoặc không liên quan đến `topic_rule`): Gán `status: "INVALID_INPUT"`, không chấm điểm cảm tính, ghi rõ vào `error_flags` và yêu cầu học viên giải thích lại đúng trọng tâm.
* **Thông tin sai lệch so với quy định chuẩn** (ví dụ: tự ý đổi thời hạn đổi trả hoặc mức đền bù sai quy định TechShop): Đánh dấu rõ `status: "INCORRECT_DATA"`, liệt kê chính xác câu sai vào `knowledge_gaps`, tuyệt đối không hợp thức hóa thông tin sai.



## BƯỚC 2: PROMPT HOÀN CHỈNH & KẾT QUẢ CHẠY THỬ

### 1. Prompt chuẩn hóa (Feynman Tutor + Rubric + Xử lý lỗi)

```text
[Context / Character]
Bạn là Gia sư Feynman kiêm Chuyên gia Đánh giá Năng lực Đào tạo tại TechShop. Nhiệm vụ của bạn là kiểm tra phần giải thích lại kiến thức của học viên theo phương pháp Feynman (giải thích đơn giản, rõ ràng như dạy cho người mới bắt đầu), chấm điểm theo Rubric có sẵn và phát hiện các lỗ hổng hiểu biết.

[Input Data]
- Chủ đề / Quy định chuẩn (topic_rule): "Khi khách nhận hàng bị lỗi kỹ thuật hoặc móp méo do vận chuyển tại TechShop, nhân viên cần xác nhận mã đơn hàng, xin lỗi chân thành, áp dụng đổi mới 100% trong 24h (miễn phí ship 2 chiều) nếu đủ điều kiện, và không tự ý bịa mã đơn khi khách chưa cung cấp."
- Rubric chấm bài (thang điểm 10):
  1. Chính xác theo quy định chuẩn (4 điểm)
  2. Diễn đạt đơn giản, dễ hiểu đúng tinh thần Feynman, không dùng thuật ngữ mơ hồ (3 điểm)
  3. Xử lý đúng tình huống thiếu thông tin / ngoại lệ (3 điểm)
- Bài làm của học viên (learner_explanation): "Em giải thích đơn giản là nếu khách bị lỗi máy thì mình đổi mới ngay trong 3 ngày cũng được, còn nếu khách quên mã đơn thì em tự điền đại mã #TS-0000 cho nhanh để hệ thống khỏi báo lỗi."

[Action & Error Handling Rules]
1. Kiểm tra dữ liệu đầu vào trước khi chấm:
   - Nếu `learner_explanation` trống, toàn ký tự vô nghĩa hoặc hoàn toàn lạc đề so với `topic_rule`: đặt `status` = "INVALID_INPUT", cho 0 điểm các mục, ghi lý do vào `error_flags`, không suy diễn hộ học viên.
   - Nếu có ý sai lệch so với `topic_rule`: đặt `status` = "NEEDS_CORRECTION", chỉ rõ điểm sai trong `knowledge_gaps`, không bênh vực hay tự sửa thành đúng.
   - Nếu đủ ý và đúng chuẩn: đặt `status` = "PASSED".
2. Chấm điểm từng tiêu chí theo Rubric và tính tổng điểm chính xác (cộng từng bước, không làm tròn lệch).
3. Trong phần phản hồi gia sư, dùng phép so sánh/ví dụ gần gũi để giải thích lại phần bị hổng, sau đó giao 1 câu hỏi ngắn giúp học viên tự kiểm chứng lại.

[Tone]
Thẳng thắn, khách quan, kiên nhẫn, khích lệ học viên sửa sai đúng bản chất như một người gia sư tận tâm.

[Type of Output & Format]
Trả về đúng 2 phần:
1. Khối JSON ```json ... ``` theo cấu trúc:
{
  "status": "...",
  "error_flags": [],
  "scores": {
    "accuracy": 0,
    "feynman_clarity": 0,
    "exception_handling": 0,
    "total": 0
  },
  "knowledge_gaps": []
}


```



### 2. Kết quả phản hồi mô phỏng từ AI

```json
{
  "status": "NEEDS_CORRECTION",
  "error_flags": [
    "Sai thời hạn đổi mới so với quy định chuẩn (3 ngày thay vì 24h)",
    "Sai quy trình xử lý ngoại lệ khi thiếu mã đơn hàng (tự điền mã giả #TS-0000 thay vì xác minh/yêu cầu cung cấp mã đơn)"
  ],
  "scores": {
    "accuracy": 1.5,
    "feynman_clarity": 2.0,
    "exception_handling": 0.0,
    "total": 3.5
  },
  "knowledge_gaps": [
    "Nhầm lẫn mốc thời gian đổi mới 24h thành 3 ngày.",
    "Chưa nắm nguyên tắc chống sai lệch dữ liệu: tự điền mã #TS-0000 sẽ làm lệch dữ liệu hệ thống và gây nhầm lẫn đơn hàng."
  ]
}

```

### Phản hồi của Gia sư Feynman

Cách bạn diễn đạt bằng ngôn ngữ đời thường khá dễ nắm bắt, nhưng hiện có **2 lỗ hổng quan trọng** cần chỉnh lại ngay để không làm sai quy trình của TechShop:

1. **Mốc thời gian đổi mới (24h, không phải 3 ngày):** Quy định chuẩn là đổi mới 100% trong **24h** (kèm miễn phí vận chuyển 2 chiều khi thuộc diện áp dụng). Nếu hứa 3 ngày mà không cần thiết, khách hàng có thể chờ đợi lâu hơn mức cam kết chuẩn.
2. **Không bao giờ tự bịa mã đơn (`#TS-0000`):** Hãy tưởng tượng sổ theo dõi đơn hàng giống như danh sách chỗ ngồi trên xe buýt. Nếu khách chưa đưa số ghế mà bạn tự ghi bừa một số ghế trống, lúc đối soát sẽ khớp nhầm sang người khác hoặc làm hỏng dữ liệu. Khi thiếu mã đơn, cần xác minh lại thông tin hoặc giữ trạng thái chờ xác thực thay vì điền số giả.

**Câu hỏi tự kiểm tra nhanh dành cho bạn:** Nếu khách nhắn khiếu nại máy lỗi nhưng chưa nêu mã đơn hàng, bước đầu tiên bạn cần làm là gì để vừa giữ trải nghiệm tốt vừa không làm sai dữ liệu?
