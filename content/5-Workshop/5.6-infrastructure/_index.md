---
title : "Deploy Infrastructure"
date :  "`r Sys.Date()`" 
weight : 6
chapter : false
pre : " <b> 5.6 </b> "
---

In this section, we will clone the repository and deploy the entire AWS infrastructure for the ARC Chatbot system.

---

## Step 1: Clone Repository

Clone the repository from GitHub:

```bash
git clone https://github.com/CrystalJohn/ARC-project.git
cd ARC-project
```

---

## Step 2: Build Dashboard

Before deploying the application, we need to build the frontend dashboard.

### Navigate to the frontend folder

```bash
cd frontend
```

### Install dependencies

Run the following command to install the necessary libraries:

```bash
npm install
```

### Build Dashboard

After installation is complete, run the build command:

```bash
npm run build
```

After the process is complete, a `dist` folder will be created. Verify the `index.html` file and the `assets` folder:

```bash
ls dist/
# index.html  assets/
```

### Return to the project root directory

```bash
cd ..
```

---

## Step 3: Deploy Terraform Application

Deploy the Terraform application. The process will take approximately **20-30 minutes** to deploy all resources.

```bash
cd terraform
terraform init
terraform apply --auto-approve
```

> ⚠️ **Note**: If you encounter an error at this step, make sure **Docker is running** on your computer.

> 💡 **Info**: Replace `<account_id>` with your actual AWS Account ID.

---

## Step 4: Verify Deployment

After completing all the steps above, your environment has been successfully deployed.

You can verify the deployment by checking:

1. **AWS Console**: Check the resources that have been created (EC2, S3, Cognito, DynamoDB, etc.)
2. **Terraform State**: Run `terraform state list` to see the list of resources
3. **S3 Buckets**: Buckets for documents and frontend have been created
4. **EC2 Instance**: Instance for backend has been launched

### Check Outputs

```bash
terraform output
```

Important outputs:

| Output | Description |
|--------|-------------|
| `api_endpoint` | Backend API URL |
| `cognito_user_pool_id` | Cognito User Pool ID |
| `cognito_client_id` | Cognito App Client ID |
| `s3_bucket_name` | S3 bucket for documents |
| `cloudfront_url` | Frontend URL |

---

## Next Steps

Now you can proceed to:

- [Set up Backend](../7-backend/)
- [Set up IDP Pipeline](../8-idp-pipeline/)
- [Set up Frontend](../9-frontend/)
