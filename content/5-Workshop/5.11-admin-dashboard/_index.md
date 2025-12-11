---
title : "Using Admin Dashboard"
date :  "`r Sys.Date()`" 
weight : 11
chapter : false
pre : " <b> 5.11 </b> "
---

# Using Admin Dashboard

Guide to using Admin Dashboard to manage documents.

---

## Access

- **URL**: http://localhost:5173/admin (or Amplify URL)
- **Requirement**: Account in `admin` group

---

## Step 1: Login Admin

Log in with admin account (created in previous step).

---

## Step 2: Dashboard Overview

After logging in, you will see:
- Upload section (drag & drop)
- Documents table with pagination
- Status filter and auto-refresh

![Admin Dashboard](/images/5-Workshop/11-admin-dashboard/admin-dashboard.png)

---

## Step 3: Upload Documents

### 3.1. Select files

- Drag & drop PDF files to upload area
- Or click **Browse Files** to select

### 3.2. Upload Progress

Each file displays progress bar and status:
- `uploading` - Uploading
- `success` - Upload successful
- `error` - Upload failed

![Upload Progress](/images/5-Workshop/11-admin-dashboard/upload-progress.png)
![Upload Progress](/images/5-Workshop/11-admin-dashboard/upload.png)

---

## Step 4: Document Status

After upload, document will be processed through IDP pipeline:

| Status | Description | Time |
|--------|-------------|------|
| `UPLOADED` | Waiting for processing | - |
| `IDP_RUNNING` | Processing | 1-5 min |
| `EMBEDDING_DONE` | Ready | - |
| `FAILED` | Error | - |

> 💡 **Tip**: Enable **Auto-refresh (5s)** to automatically update status.

![Document Status](/images/5-Workshop/11-admin-dashboard/document-status.png)

---

## Step 5: Manage Documents

### Filter by Status

Use **Status** dropdown to filter:
- All
- Uploaded
- Processing
- Done
- Failed

### Pagination

Documents are paginated (5 items/page). Use pagination controls at footer.

### View Document

Click 👁️ icon to view document details.

### Delete Document

Click 🗑️ icon to delete document.

> ⚠️ **Warning**: Deleting document will delete from S3, DynamoDB, and Qdrant.

---

## Step 6: Processing History

Click **Processing History** link to view document processing history.

![Processing History](/images/5-Workshop/11-admin-dashboard/processing-history.png)

---

## Error Handling

| Issue | Solution |
|-------|----------|
| Upload failed | Check file size (<50MB), format (PDF only) |
| Document stuck in IDP_RUNNING | Check worker logs on EC2 |
| Document FAILED | See error message in Processing History |

---

## Checklist

- [ ] Logged in to admin dashboard
- [ ] Upload document successful
- [ ] Document processed (EMBEDDING_DONE)
- [ ] Filter/pagination working
- [ ] Auto-refresh working
