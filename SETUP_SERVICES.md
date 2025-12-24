# HƯỚNG DẪN CẤU HÌNH DỊCH VỤ (SETUP SERVICES)

Tài liệu này hướng dẫn chi tiết cách lấy API Key và File cấu hình từ các bên thứ 3.

## 1. Google Cloud (Google Drive API) - Quan trọng nhất
*Bước này để n8n có quyền đọc file trong Drive của công ty.*

1.  Truy cập [Google Cloud Console](https://console.cloud.google.com/).
2.  Tạo **New Project** (đặt tên: `n8n-chatbot`).
3.  Vào menu **APIs & Services** > **Library** > Tìm "Google Drive API" > Nhấn **Enable**.
4.  Vào **Credentials** > **Create Credentials** > **Service Account**.
    * Điền tên, nhấn Done.
    * Click vào email Service Account vừa tạo (dạng `...@...iam.gserviceaccount.com`).
    * Tab **Keys** > **Add Key** > **Create new key** > Chọn **JSON**.
    * Tải file `.json` về máy, đổi tên thành `service_account_credentials.json`.
5.  **QUAN TRỌNG:** Mở Google Drive, chuột phải vào thư mục chứa tài liệu > **Share** > Dán địa chỉ email của Service Account vào (cấp quyền Viewer).

## 2. Pinecone (Vector Database)
1.  Đăng ký tại [Pinecone.io](https://www.pinecone.io/).
2.  Tạo **Index**:
    * Name: `company-data`
    * Dimensions: `1536` (Bắt buộc phải số này để khớp với OpenAI).
    * Metric: `cosine`.
3.  Vào mục **API Keys** > Copy Key.

## 3. Telegram Bot
1.  Chat với [@BotFather](https://t.me/BotFather).
2.  Gõ `/newbot` > Đặt tên > Lấy **Token** (dạng `123456:ABC-XYZ...`).
