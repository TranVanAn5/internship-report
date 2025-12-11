---
title : "Triển khai giải pháp"
date :  "`r Sys.Date()`" 
weight : 6
chapter : false
pre : " <b> 5.6 </b> "
---

Trong phần này, chúng ta sẽ clone repository và triển khai toàn bộ hạ tầng AWS cho hệ thống ARC Chatbot.

---

## Bước 1: Clone Repository

Clone repository từ GitHub:

```bash
git clone https://github.com/CrystalJohn/ARC-project.git
cd ARC-project
```

---

## Bước 2: Build Dashboard

Trước khi triển khai ứng dụng, chúng ta cần build frontend dashboard.

### Di chuyển đến thư mục frontend

```bash
cd frontend
```

### Cài đặt dependencies

Chạy lệnh sau để cài đặt các thư viện cần thiết:

```bash
npm install
```

### Build Dashboard

Sau khi cài đặt hoàn tất, chạy lệnh build:

```bash
npm run build
```

Sau khi quá trình hoàn tất, một thư mục `dist` sẽ được tạo. Kiểm tra file `index.html` và thư mục `assets`:

```bash
ls dist/
# index.html  assets/
```

### Quay lại thư mục gốc của project

```bash
cd ..
```

---

## Bước 3: Triển khai CDK Application

Triển khai ứng dụng CDK. Quá trình sẽ mất khoảng **20-30 phút** để triển khai tất cả các tài nguyên.

```bash
cd terraform
terraform init
terraform apply --auto-approve
```

> ⚠️ **Note**: Nếu bạn gặp lỗi ở bước này, hãy đảm bảo **Docker đang chạy** trên máy tính của bạn.

> 💡 **Info**: Thay thế `<account_id>` bằng AWS Account ID thực tế của bạn.

---

## Bước 4: Xác minh Triển khai

Sau khi hoàn thành tất cả các bước trên, môi trường của bạn đã được triển khai thành công.

Bạn có thể xác minh triển khai bằng cách kiểm tra:

1. **AWS Console**: Kiểm tra các resources đã được tạo (EC2, S3, Cognito, DynamoDB, etc.)
2. **Terraform State**: Chạy `terraform state list` để xem danh sách resources
3. **S3 Buckets**: Bucket cho documents và frontend đã được tạo
4. **EC2 Instance**: Instance cho backend đã được khởi tạo

### Kiểm tra Outputs

```bash
terraform output
```

Các outputs quan trọng:

| Output | Mô tả |
|--------|-------|
| `api_endpoint` | Backend API URL |
| `cognito_user_pool_id` | Cognito User Pool ID |
| `cognito_client_id` | Cognito App Client ID |
| `s3_bucket_name` | S3 bucket cho documents |
| `cloudfront_url` | Frontend URL |

---

## Các Bước Tiếp Theo

Bây giờ bạn có thể tiếp tục:

- [Thiết lập Backend](../7-backend/)
- [Thiết lập IDP Pipeline](../8-idp-pipeline/)
- [Thiết lập Frontend](../9-frontend/)

