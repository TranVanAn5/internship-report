---
title : "Thiết lập Frontend"
date :  "`r Sys.Date()`" 
weight : 9
chapter : false
pre : " <b> 5.9 </b> "
---

# Thiết lập Frontend

Cấu hình và deploy Frontend React application với AWS Amplify.

## Bước 1: Lấy Terraform Outputs

```bash
cd terraform
terraform output
```

Ghi lại: `cognito_user_pool_id`, `cognito_client_id`, `alb_dns_name`

---

## Bước 2: Cấu hình Environment

```bash
cd ARC-project
cp .env.example .env
```

Chỉnh sửa `.env`:

```bash
VITE_AWS_REGION=ap-southeast-1
VITE_COGNITO_POOL_ID=ap-southeast-1_xxxxxxx
VITE_COGNITO_CLIENT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxx
VITE_API_URL=http://arc-chatbot-dev-alb-xxxxx.ap-southeast-1.elb.amazonaws.com
```

---

## Bước 3: Install & Test Local

```bash
npm install
npm run dev
```

Mở http://localhost:5173

![Dev Server](/images/5-Workshop/9-frontend/dev-server.png)

---

## Bước 4: Build & Deploy

```bash
npm run build
```

Push code lên GitHub, Amplify sẽ tự động deploy:

```bash
git add .
git commit -m "Update frontend config"
git push origin main
```

> 💡 Amplify app đã được tạo qua Terraform và connected với GitHub.

![Amplify Deploy](/images/5-Workshop/9-frontend/amplify-deploy.png)

---

## Bước 5: Cập nhật Cognito Callback URLs

Sau khi có Amplify URL:

```bash
aws cognito-idp update-user-pool-client \
  --user-pool-id ap-southeast-1_xxxxxxx \
  --client-id xxxxxxxxxx \
  --callback-urls "http://localhost:5173" "https://main.xxxxx.amplifyapp.com" \
  --logout-urls "http://localhost:5173" "https://main.xxxxx.amplifyapp.com"
```

---

## Bước 6: Tạo Test Users

```bash
# Admin user
aws cognito-idp admin-create-user \
  --user-pool-id ap-southeast-1_xxxxxxx \
  --username admin@example.com \
  --user-attributes Name=email,Value=admin@example.com \
  --temporary-password "TempPass123!"

aws cognito-idp admin-add-user-to-group \
  --user-pool-id ap-southeast-1_xxxxxxx \
  --username admin@example.com \
  --group-name admin
```

---

## Xử lý Lỗi

| Lỗi | Giải pháp |
|-----|-----------|
| CORS error | Kiểm tra FastAPI CORS config |
| "User pool does not exist" | Kiểm tra VITE_COGNITO_POOL_ID |
| Build failed | Kiểm tra Amplify environment variables |

---

## Checklist

- [ ] `.env` đã cấu hình
- [ ] Local dev server chạy được
- [ ] Amplify deploy thành công
- [ ] Cognito callback URLs đã cập nhật
- [ ] Login/Register hoạt động

