# Bài 5: Cơ chế In-Context Learning trong Phân loại Review Khách hàng
## Phần 1: Phân tích cơ chế In-Context Learning (ICL)

- 1. Bản chất không cập nhật trọng số (Zero Weight Update)
Không đổi weights: Khác với Fine-tuning, ICL giữ nguyên hoàn toàn ma trận trọng số của LLM và không thực hiện tính toán đạo hàm.

Suy luận thời gian thực: Toàn bộ quá trình "học" diễn ra trực tiếp bên trong cửa sổ ngữ cảnh (Context Window) ngay trong giai đoạn suy luận (Inference).

- 2. Kích hoạt mẫu hình qua Token (Pattern Matching & Induction Heads)
Xử lý chuỗi liên tục: Mô hình tiếp nhận các ví dụ mẫu (Input -> Output) như một chuỗi văn bản liền mạch.

Cơ chế Self-Attention: Tận dụng khả năng tự chú ý để biến đổi ví dụ thành các không gian biểu diễn ẩn (latent representations), qua đó nắm bắt mối quan hệ logic giữa đầu vào và đầu ra mà không cần can thiệp cấu trúc mạng nơ-ron.

- 3. Nguyên nhân Zero-shot thất bại với các bài toán mỉa mai
Hạn chế của Zero-shot: Chỉ phân tích nghĩa hiển ngôn (Surface Lexical Meaning) dựa trên xác suất đồng xuất hiện từ ngữ.


Hiệu quả của Few-shot (ICL): Các ví dụ mẫu giúp mô hình thấu hiểu sắc thái đặc thù của dân công nghệ (khen đểu, chơi chữ, mỉa mai), từ đó định hướng suy luận chính xác nhãn TIÊU CỰC.
# Phần 2: Thiết kế Prompt Few-shot hoàn chỉnh
### 1. Vai trò (Role)
Bạn là Trưởng bộ phận Phân tích Dữ liệu Khách hàng và Kiểm duyệt Chất lượng tại TechShop. Nhiệm vụ của bạn là phân tích chính xác cảm xúc (Tích cực / Tiêu cực) của các đánh giá khách hàng, đặc biệt là nhận diện sắc thái mỉa mai, khen đểu trong lĩnh vực công nghệ.

### 2. Quy tắc (Rules)
- Đọc kỹ toàn bộ ngữ cảnh câu review. Chú ý các chỉ số kỹ thuật bất thường (nhiệt độ quá cao, thời lượng pin quá ngắn, thời gian giao hàng quá lâu) kết hợp với các từ ngữ mang tính khen ngợi bề ngoài để phát hiện ý nghĩa mỉa mai thực tế.
- Trả về kết quả theo đúng định dạng: `Cảm xúc: [Tích cực / Tiêu cực]` kèm theo một câu giải thích ngắn gọn dựa trên ngữ cảnh kỹ thuật.

### 3. Ví dụ mẫu (Few-shot Examples)

---
- Input: "Máy thiết kế đẹp mê ly, cầm rất đầm tay, mùa đông ôm máy chơi game thì khỏi cần bật lò sưởi vì nhiệt độ CPU lên hẳn 98 độ C!"
- Output: 
  - Cảm xúc: TIÊU CỰC
  - Giải thích: Khách hàng dùng từ khen bề ngoài ('đẹp mê ly', 'khỏi cần bật lò sưởi') nhưng thực chất đang mỉa mai lỗi quá nhiệt nghiêm trọng của CPU (lên tới 98°C).
---
- Input: "Điện thoại pin trâu kinh khủng, sạc từ sáng đến tối phải cắm sạc đến 5 lần, đúng là thiết bị kết nối với ổ điện trọn đời!"
- Output: 
  - Cảm xúc: TIÊU CỰC
  - Giải thích: Cụm từ 'pin trâu' kết hợp với việc phải cắm sạc 5 lần một ngày là biểu hiện của sự mỉa mai về thời lượng pin yếu kém.
---
- Input: "Shop giao hàng nhanh như vận mệnh vũ trụ, đặt hàng từ mùa quýt năm ngoái mà đến mùa quýt năm nay mới nhận được hàng trên tay!"
- Output: 
  - Cảm xúc: TIÊU CỰC
  - Giải thích: Lời khen 'nhanh như vận mệnh vũ trụ' mang tính châm biếm khi đối lập với thực tế giao hàng cực kỳ chậm chạp.
---

### 4. Dữ liệu cần phân tích (Input)
- Review: "Máy thiết kế đẹp mê ly, cầm rất đầm tay, mùa đông ôm máy chơi game thì khỏi cần bật lò sưởi vì nhiệt độ CPU lên hẳn 98 độ C!"
- Output: