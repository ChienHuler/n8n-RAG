# HƯỚNG DẪN XÂY DỰNG AI CHATBOT TRA CỨU DỮ LIỆU DOANH NGHIỆP (n8n + Google Drive)

**Mục tiêu:** Tạo bot Telegram trả lời câu hỏi dựa trên tài liệu (PDF, Word, Excel) từ Google Drive.

---

## 🛠 PHẦN 1: CHUẨN BỊ MÔI TRƯỜNG (PREREQUISITES)

Trước khi vào n8n, bạn cần lấy "chìa khóa" (API Key) của các dịch vụ sau:

1.  **Google Cloud Console:**
    * Bật **Google Drive API**.
    * Tạo Service Account, tải file JSON credentials về.
    * *Quan trọng:* Share thư mục Drive chứa dữ liệu cho email của Service Account này (để n8n có quyền đọc).
2.  **Pinecone (Vector Database):**
    * Tạo Index mới.
    * Dimensions: `1536` (Chuẩn của OpenAI).
    * Metric: `Cosine`.
3.  **OpenAI:**
    * Lấy API Key để dùng mô hình `gpt-4o-mini` (rẻ và thông minh) và `text-embedding-3-small`.
4.  **Telegram:**
    * Chat với @BotFather để tạo bot mới và lấy Token.

---

## 🔄 PHẦN 2: THIẾT KẾ WORKFLOW A - "NGƯỜI QUẢN KHO DỮ LIỆU" (INGESTION)

**Nhiệm vụ:** Tự động đọc file từ Drive, biến đổi thành số (vector) và cất vào Pinecone.

### Sơ đồ tư duy:
`Google Drive (File mới)` -> `Tải file` -> `Trích xuất chữ (Text)` -> `Cắt nhỏ (Chunking)` -> `Tạo Vector (Embedding)` -> `Lưu vào Pinecone`

### Các bước cài đặt trên n8n:

#### Bước 1: Trigger (Kích hoạt)
* **Node:** `Google Drive Trigger`.
* **Cấu hình:** Chọn "File Created" hoặc "File Updated".
* **Lọc:** Chỉ định ID của thư mục chứa tài liệu công ty (Kho, IT, Dự án...).

#### Bước 2: Tải file
* **Node:** `Google Drive`.
* **Operation:** `Download`.
* **File ID:** Lấy từ node Trigger phía trước.

#### Bước 3: Xử lý định dạng file (Quan trọng)
*Đây là bước khó nhất vì công ty có nhiều loại file. Ta dùng node "Switch" hoặc logic đơn giản hóa:*

* **Đối với PDF/Word:**
    * Dùng node `Read Binary File` (hoặc các node parser có sẵn trong n8n community).
    * Mục tiêu: Lấy ra toàn bộ văn bản thô (Plain text).
* **Đối với Excel (Dữ liệu Kho, Giấy phép IT):**
    * *Lưu ý:* AI đọc bảng tính rất kém.
    * **Giải pháp:** Dùng node `Spreadsheet File` để đọc dữ liệu, sau đó dùng node `Code` để chuyển đổi từng dòng thành câu văn.
    * *Ví dụ code chuyển đổi:*
        ```javascript
        // Biến dòng Excel: {SanPham: "Sắt", SoLuong: 50, Ngay: "2023-10-01"}
        // Thành câu: "Vào ngày 2023-10-01, kho đã xuất 50 đơn vị Sắt."
        return { json: { text: `Vào ngày ${item.Ngay}, kho đã xuất ${item.SoLuong} ${item.SanPham}.` } }
        ```
* **Đối với Bản vẽ (Ảnh/CAD):**
    * AI không nhìn thấy hình trong Vector Store cơ bản.
    * **Giải pháp:** Tên file phải cực kỳ chi tiết (VD: `Ban_ve_mong_nha_A_ngay_20_10.pdf`) hoặc kèm theo một file text mô tả nội dung bản vẽ đó.

#### Bước 4: Cắt nhỏ & Nhúng (Chunk & Embed)
* **Node:** `Recursive Character Text Splitter` (Nối vào node xử lý text).
    * Size: 1000 ký tự.
    * Overlap: 100 ký tự (để giữ ngữ cảnh liền mạch).
* **Node:** `Embeddings OpenAI`.
    * Model: `text-embedding-3-small`.
* **Node:** `Pinecone Vector Store`.
    * Operation: `Upsert` (Chèn hoặc cập nhật).

---

## 🤖 PHẦN 3: THIẾT KẾ WORKFLOW B - "TRỢ LÝ TRẢ LỜI" (RETRIEVAL)

**Nhiệm vụ:** Nhận câu hỏi từ Telegram, tìm dữ liệu liên quan và trả lời.

### Các bước cài đặt trên n8n:

#### Bước 1: Nhận tin nhắn
* **Node:** `Telegram Trigger`.
* **Event:** `Message`.

#### Bước 2: Tìm kiếm dữ liệu (Retrieval)
* **Node:** `Embeddings OpenAI` (Biến câu hỏi người dùng thành vector).
* **Node:** `Pinecone Vector Store`.
    * Operation: `Retrieve` (Truy xuất).
    * Limit: Lấy khoảng 5-10 đoạn văn bản giống nhất.

#### Bước 3: Tổng hợp câu trả lời (Generation)
* **Node:** `OpenAI` (Chat Model).
* **Model:** `gpt-4o`.
* **System Prompt (Câu lệnh cho não bộ AI):**
    * *Đây là phần quan trọng nhất để AI thông minh.*
    * Nội dung Prompt mẫu:
    ```text
    Bạn là trợ lý ảo nội bộ của công ty.
    Dưới đây là các thông tin tìm được từ tài liệu công ty (Context):
    {{ $json.context }}

    Câu hỏi của người dùng: {{ $json.question }}

    Yêu cầu:
    1. Chỉ trả lời dựa trên Context được cung cấp.
    2. Nếu dữ liệu là về số lượng (tồn kho, giấy phép), hãy liệt kê rõ ràng.
    3. Nếu câu hỏi về tiến độ, hãy tóm tắt mốc thời gian gần nhất.
    4. Nếu không có thông tin trong Context, hãy trả lời: "Tôi chưa tìm thấy tài liệu về vấn đề này, vui lòng kiểm tra lại Drive."
    ```

#### Bước 4: Gửi lại Telegram
* **Node:** `Telegram`.
* **Operation:** `Send Message`.
* **Chat ID:** Lấy từ Trigger ban đầu.

---

## 💡 MẸO TỐI ƯU CHO NGƯỜI LẬP TRÌNH MỚI (BEST PRACTICES)

### 1. Vấn đề "Bản vẽ" và "Hình ảnh"
Workflow trên chỉ xử lý văn bản (Text). Để AI hiểu bản vẽ, bạn có 2 cách đơn giản:
* **Cách 1 (Dễ):** Đặt tên file bản vẽ thật chi tiết. AI sẽ tìm được dựa trên tên file.
* **Cách 2 (Khó hơn):** Dùng một Workflow phụ, sử dụng `GPT-4o Vision` để "nhìn" ảnh bản vẽ, viết ra một đoạn mô tả (ví dụ: "Đây là bản vẽ móng cọc dự án A..."), sau đó lưu đoạn mô tả đó vào Pinecone.

### 2. Vấn đề "Excel" và Số liệu chính xác
Vector Database tìm kiếm theo "ngữ nghĩa" (gần giống), đôi khi nó bỏ sót con số chính xác.
* **Lời khuyên:** Hãy yêu cầu các phòng ban upload file Excel nhưng có thêm cột "Mô tả" bằng lời văn. Hoặc chuyển đổi file Excel sang PDF trước khi upload lên Drive để AI đọc tốt hơn.

### 3. Debug (Sửa lỗi)
* Luôn dùng nút **Execute Node** trong n8n để chạy thử từng bước xem dữ liệu đầu ra (Output) trông như thế nào trước khi nối sang bước tiếp theo.
