---
title : "Sử dụng Admin Dashboard"
date :  "`r Sys.Date()`" 
weight : 11
chapter : false
pre : " <b> 5.11 </b> "
---

# Sử dụng Admin Dashboard

Hướng dẫn sử dụng Admin Dashboard để quản lý documents.

---

## Truy cập

- **URL**: http://localhost:5173/admin (hoặc Amplify URL)
- **Yêu cầu**: Tài khoản thuộc group `admin`

---

## Bước 1: Đăng nhập Admin

Đăng nhập với tài khoản admin (đã tạo ở bước trước).

---

## Bước 2: Dashboard Overview

Sau khi đăng nhập, bạn sẽ thấy:
- Upload section (drag & drop)
- Documents table với pagination
- Status filter và auto-refresh
---

## Bước 3: Upload Tài liệu

### 3.1. Chọn file

- Drag & drop PDF files vào vùng upload
- Hoặc click **Browse Files** để chọn

### 3.2. Upload Progress

Mỗi file hiển thị progress bar và status:
- `uploading` - Đang upload
- `success` - Upload thành công
- `error` - Upload thất bại

![Upload Progress](/images/5-Workshop/11-admin-dashboard/upload-progress.png)
![Upload Progress](/images/5-Workshop/11-admin-dashboard/upload.png)

---

## Bước 4: Document Status

Sau khi upload, document sẽ được xử lý qua IDP pipeline:

| Status | Mô tả | Thời gian |
|--------|-------|-----------|
| `UPLOADED` | Chờ xử lý | - |
| `IDP_RUNNING` | Đang xử lý | 1-5 phút |
| `EMBEDDING_DONE` | Sẵn sàng | - |
| `FAILED` | Lỗi | - |

> 💡 **Tip**: Bật **Auto-refresh (5s)** để tự động cập nhật status.

![Document Status](/images/5-Workshop/11-admin-dashboard/document-status.png)

---

## Bước 5: Quản lý Documents

### Filter theo Status

Sử dụng dropdown **Status** để lọc:
- All
- Uploaded
- Processing
- Done
- Failed

### Pagination

Documents được phân trang (5 items/page). Sử dụng pagination controls ở footer.

### View Document

Click icon 👁️ để xem chi tiết document.

### Delete Document

Click icon 🗑️ để xóa document.

> ⚠️ **Warning**: Xóa document sẽ xóa khỏi S3, DynamoDB và Qdrant.

---

## Bước 6: Processing History

Click **Processing History** link để xem lịch sử xử lý documents.

![Processing History](/images/5-Workshop/11-admin-dashboard/processing-history.png)

---

## Xử lý Lỗi

| Vấn đề | Giải pháp |
|--------|-----------|
| Upload failed | Kiểm tra file size (<50MB), format (PDF only) |
| Document stuck in IDP_RUNNING | Kiểm tra worker logs trên EC2 |
| Document FAILED | Xem error message trong Processing History |

---

## Checklist

- [ ] Đăng nhập admin dashboard
- [ ] Upload document thành công
- [ ] Document processed (EMBEDDING_DONE)
- [ ] Filter/pagination hoạt động
- [ ] Auto-refresh hoạt động


