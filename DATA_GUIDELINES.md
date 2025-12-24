# QUY ĐỊNH CHUẨN HÓA DỮ LIỆU ĐẦU VÀO CHO AI

Để Chatbot trả lời chính xác, các phòng ban vui lòng tuân thủ quy tắc đặt tên và soạn thảo file trước khi upload lên Google Drive.

## 1. Quy tắc đặt tên File
*Tên file phải thể hiện rõ nội dung, không viết tắt khó hiểu.*

* ❌ **Sai:** `BC_T10.xlsx`, `scan001.pdf`, `final_v2.docx`
* ✅ **Đúng:** `Bao_cao_ton_kho_thang_10_2024.xlsx`, `Ban_ve_mong_nha_xuong_A.pdf`, `Quy_trinh_xin_nghi_phep_2025.docx`

## 2. Đối với file Excel (Kho, IT, Nhân sự)
AI đọc file Excel rất khó khăn nếu định dạng lộn xộn.
* **Không gộp ô (Merge Cells):** Tuyệt đối tránh merge cell ở tiêu đề.
* **Hàng đầu tiên là tiêu đề:** Hàng 1 phải chứa tên cột (Ví dụ: Tên sản phẩm, Số lượng, Đơn vị).
* **Thêm Sheet "Ghi chú" (Khuyên dùng):** Tạo thêm 1 sheet mô tả bằng lời văn về nội dung file này.
    * *Ví dụ:* "File này chứa danh sách tồn kho tính đến ngày 25/12. Các mã hàng bắt đầu bằng 'H' là hàng hư hỏng."

## 3. Đối với file PDF/Word (Văn bản, Quy trình)
* Sử dụng Heading (Tiêu đề 1, Tiêu đề 2) rõ ràng trong Word để AI nhận biết cấu trúc bài viết.
* Nếu là file Scan (ảnh), bắt buộc phải chạy qua phần mềm OCR (nhận dạng chữ) trước khi upload, nếu không AI sẽ không đọc được gì.
