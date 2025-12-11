---
title : "Setup Frontend"
date :  "`r Sys.Date()`" 
weight : 9
chapter : false
pre : " <b> 5.9 </b> "
---

# Setup Frontend

Configure and deploy Frontend React application with AWS Amplify.

## Step 1: Get Terraform Outputs

```bash
cd terraform
terraform output
```

Note: `cognito_user_pool_id`, `cognito_client_id`, `alb_dns_name`

---

## Step 2: Configure Environment

```bash
cd ARC-project
cp .env.example .env
```

Edit `.env`:

```bash
VITE_AWS_REGION=ap-southeast-1
VITE_COGNITO_POOL_ID=ap-southeast-1_xxxxxxx
VITE_COGNITO_CLIENT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxx
VITE_API_URL=http://arc-chatbot-dev-alb-xxxxx.ap-southeast-1.elb.amazonaws.com
```

---

## Step 3: Install & Test Local

```bash
npm install
npm run dev
```

Open http://localhost:5173

![Dev Server](/images/5-Workshop/9-frontend/dev-server.png)

---

## Step 4: Build & Deploy

```bash
npm run build
```

Push code to GitHub, Amplify will deploy automatically:

```bash
git add .
git commit -m "Update frontend config"
git push origin main
```

> 💡 Amplify app was created via Terraform and connected with GitHub.

![Amplify Deploy](/images/5-Workshop/9-frontend/amplify-deploy.png)

---

## Step 5: Update Cognito Callback URLs

After getting Amplify URL:

```bash
aws cognito-idp update-user-pool-client \
  --user-pool-id ap-southeast-1_xxxxxxx \
  --client-id xxxxxxxxxx \
  --callback-urls "http://localhost:5173" "https://main.xxxxx.amplifyapp.com" \
  --logout-urls "http://localhost:5173" "https://main.xxxxx.amplifyapp.com"
```

---

## Step 6: Create Test Users

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

## Error Handling

| Error | Solution |
|-------|----------|
| CORS error | Check FastAPI CORS config |
| "User pool does not exist" | Check VITE_COGNITO_POOL_ID |
| Build failed | Check Amplify environment variables |

---

## Checklist

- [ ] `.env` configured
- [ ] Local dev server running
- [ ] Amplify deploy successful
- [ ] Cognito callback URLs updated
- [ ] Login/Register working