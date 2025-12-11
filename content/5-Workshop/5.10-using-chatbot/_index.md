---
title : "Using Chatbot"
date :  "`r Sys.Date()`" 
weight : 10
chapter : false
pre : " <b> 5.10 </b> "
---

Guide to using ARC Chatbot to search for information from research documents.

## Access

- **Local**: http://localhost:5173
- **Production**: Amplify URL from previous step

---

## Step 1: Login

1. Enter email and password
2. Click **Login**
3. Redirect to Chat page

![Login Page](/images/5-Workshop/10-using-chatbot/login-page.png)

---

## Step 2: Chat Interface

After logging in, you will see:
- Sidebar with Chat, History menus
- Header with user info and dark mode toggle
- Chat area with welcome message

![Chat Interface](/images/5-Workshop/10-using-chatbot/chat-interface.png)

---

## Step 3: Ask Questions

Enter a question in the input box and press Enter or click Send.

### Good Questions

| Type | Example |
|------|---------|
| Definition | "What is a stack data structure?" |
| Comparison | "Compare stack and queue" |
| Explanation | "Explain binary search algorithm" |

### Avoid

- ❌ Too general: "Tell me about programming"
- ❌ Outside documents: "What's the weather today?"

![Query Response](/images/5-Workshop/10-using-chatbot/query-response.png)
![Query Response](/images/5-Workshop/10-using-chatbot/query-response-outside.png)

---

## Step 4: Citations

Each answer has citations showing document sources:

```
📚 Sources:
[1] data-structures.pdf - Page 12 - Score: 85%
[2] algorithms.pdf - Page 45 - Score: 72%
```

Click on citation to view document details.

| Field | Description |
|-------|-------------|
| [1], [2] | Citation number |
| Filename | PDF filename |
| Page | Page number |
| Score | Relevance (%) |

![Citations Display](/images/5-Workshop/10-using-chatbot/citations-display.png)

---

## Step 5: Conversation History

1. Click **History** in sidebar
2. View list of previous conversations
3. Click conversation to reload it
4. Click trash icon to delete

![Conversation History](/images/5-Workshop/10-using-chatbot/conversation-history.png)

---

## Step 6: New Chat

Click **New Chat** in sidebar to start a new conversation.

---

## Features

| Feature | Description |
|---------|-------------|
| Streaming | Response displays in parts |
| Markdown | Supports code blocks, lists, headers |
| Dark Mode | Toggle in header |
| History | Save and reload conversations |

---

## Checklist

- [ ] Successfully logged in
- [ ] Sent query and received response
- [ ] Citations displayed correctly
- [ ] Click citation to view document
- [ ] History working
- [ ] New Chat working
