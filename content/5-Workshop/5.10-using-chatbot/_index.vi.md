---
title : "Sử dụng Chatbot"
date :  "`r Sys.Date()`" 
weight : 10
chapter : false
pre : " <b> 5.10 </b> "
---

Hướng dẫn sử dụng ARC Chatbot để tìm kiếm thông tin từ tài liệu nghiên cứu.

## Truy cập

- **Local**: http://localhost:5173
- **Production**: Amplify URL từ bước trước

---

## Bước 1: Đăng nhập

1. Nhập email và password
2. Click **Login**
3. Redirect đến Chat page

![Login Page](/images/5-Workshop/10-using-chatbot/login-page.png)

---

## Bước 2: Giao diện Chat

Sau khi đăng nhập, bạn sẽ thấy:
- Sidebar với menu Chat, History
- Header với user info và dark mode toggle
- Chat area với welcome message

![Chat Interface](/images/5-Workshop/10-using-chatbot/chat-interface.png)

---

## Bước 3: Đặt câu hỏi

Nhập câu hỏi vào input box và nhấn Enter hoặc click Send.

### Câu hỏi tốt

| Loại | Ví dụ |
|------|-------|
| Định nghĩa | "What is a stack data structure?" |
| So sánh | "Compare stack and queue" |
| Giải thích | "Explain binary search algorithm" |

### Tránh

- ❌ Quá chung: "Tell me about programming"
- ❌ Ngoài tài liệu: "What's the weather today?"

![Query Response](/images/5-Workshop/10-using-chatbot/query-response.png)
![Query Response](/images/5-Workshop/10-using-chatbot/query-response-outside.png)
---

## Bước 4: Citations (Trích dẫn)

Mỗi câu trả lời có citations hiển thị nguồn tài liệu:

```
📚 Sources:
[1] data-structures.pdf - Page 12 - Score: 85%
[2] algorithms.pdf - Page 45 - Score: 72%
```

Click vào citation để xem chi tiết document.

| Field | Mô tả |
|-------|-------|
| [1], [2] | Số thứ tự citation |
| Filename | Tên file PDF |
| Page | Số trang |
| Score | Độ liên quan (%) |

![Citations Display](/images/5-Workshop/10-using-chatbot/citations-display.png)

---

## Bước 5: Conversation History

1. Click **History** trong sidebar
2. Xem danh sách các cuộc hội thoại trước
3. Click vào conversation để load lại
4. Click trash icon để xóa

![Conversation History](/images/5-Workshop/10-using-chatbot/conversation-history.png)

---

## Bước 6: New Chat

Click **New Chat** trong sidebar để bắt đầu cuộc hội thoại mới.

---

## Features

| Feature | Mô tả |
|---------|-------|
| Streaming | Response hiển thị từng phần |
| Markdown | Hỗ trợ code blocks, lists, headers |
| Dark Mode | Toggle trong header |
| History | Lưu và load lại conversations |

---

## Checklist

- [ ] Đăng nhập thành công
- [ ] Gửi query và nhận response
- [ ] Citations hiển thị đúng
- [ ] Click citation xem document
- [ ] History hoạt động
- [ ] New Chat hoạt động

