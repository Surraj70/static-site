# 🚀 Automated Static Website Deployment Pipeline

A fully automated CI/CD pipeline that deploys a static portfolio website to AWS S3 + CloudFront on every `git push` to main. No manual uploads. No console clicking. Just push and it's live.

---

## ⚡ How It Works

```
git push origin main
       │
       ▼
GitHub Actions triggered
       │
       ▼
Checkout code → Configure AWS credentials
       │
       ▼
aws s3 sync ./src → S3 Bucket
       │
       ▼
CloudFront cache invalidated
       │
       ▼
✅ Live on the internet in < 60 seconds
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| CI/CD | GitHub Actions |
| Storage | AWS S3 |
| CDN | AWS CloudFront |
| Security | AWS IAM (least privilege) |
| IaC (optional) | Terraform |
| Frontend | Static HTML / CSS |

---

## 📁 Project Structure

```
├── .github/
│   └── workflows/
│       └── deploy.yml       # CI/CD pipeline definition
├── src/
│   └── index.html           # Static website files
├── terraform/
│   └── main.tf              # Infrastructure as code (optional)
├── .gitignore
└── README.md
```

---

## 🔧 Setup & Deployment

### Prerequisites

- AWS account with S3 bucket and CloudFront distribution created
- GitHub repository with Actions enabled

### 1. Clone the repo

```bash
git clone https://github.com/Surraj70/portfolio-cicd-aws.git
cd your-repo-name
```

### 2. Add GitHub Secrets

Go to your repo → **Settings → Secrets and variables → Actions** and add:

| Secret | Description |
|---|---|
| `AWS_ACCESS_KEY_ID` | IAM user access key |
| `AWS_SECRET_ACCESS_KEY` | IAM user secret key |
| `AWS_S3_BUCKET` | Your S3 bucket name |
| `AWS_CLOUDFRONT_DISTRIBUTION_ID` | Your CloudFront distribution ID |

### 3. IAM Permissions Required

Your IAM user needs the following permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject", "s3:DeleteObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::YOUR_BUCKET_NAME",
        "arn:aws:s3:::YOUR_BUCKET_NAME/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": "cloudfront:CreateInvalidation",
      "Resource": "arn:aws:cloudfront::YOUR_ACCOUNT_ID:distribution/*"
    }
  ]
}
```

### 4. Deploy

Just push to main:

```bash
git add .
git commit -m "your message"
git push origin main
```

Watch the pipeline run live under the **Actions** tab.

---

## ⚙️ GitHub Actions Workflow

```yaml
name: Deploy to AWS S3

on:
  push:
    branches:
      - main

env:
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1

      - name: Deploy to S3
        run: |
          aws s3 sync ./src s3://${{ secrets.AWS_S3_BUCKET }} \
            --delete \
            --cache-control "max-age=86400"

      - name: Invalidate CloudFront cache
        env:
          CF_DIST_ID: ${{ secrets.AWS_CLOUDFRONT_DISTRIBUTION_ID }}
        if: env.CF_DIST_ID != ''
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.AWS_CLOUDFRONT_DISTRIBUTION_ID }} \
            --paths "/*"
```

---

## 🌍 Infrastructure as Code (Terraform)

Optionally provision the S3 bucket and CloudFront distribution with code instead of clicking through the console:

```bash
cd terraform
terraform init
terraform plan
terraform apply
```

> See [`terraform/main.tf`](./terraform/main.tf) for the full configuration.

---

## 🐛 Errors I Hit & How I Fixed Them

### ❌ `Unexpected symbol: '${{' `
The `if:` field in GitHub Actions is already an expression context — wrapping it in `${{ }}` breaks the parser. Also, secrets can't be referenced directly in `if` conditions.

**Fix:** Copy the secret into an `env` variable first, then check that.

```yaml
env:
  CF_DIST_ID: ${{ secrets.AWS_CLOUDFRONT_DISTRIBUTION_ID }}
if: env.CF_DIST_ID != ''
```

---

### ❌ `Exit code 254` on `aws s3 sync`
The credentials step showed green but sync failed silently. Root cause: GitHub Actions was running on deprecated Node.js 20, causing `aws-actions/configure-aws-credentials` to fail to export environment variables.

**Fix:** Force Node.js 24 at the workflow level:

```yaml
env:
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true
```

---

### ❌ `AccessDenied` on CloudFront invalidation
The IAM user had S3 access but no CloudFront permissions — classic least-privilege gap.

**Fix:** Added a scoped inline IAM policy granting only `cloudfront:CreateInvalidation`.

---

## 💰 Cost

This setup runs at effectively **$0/month** on AWS Free Tier:

| Service | Free Tier | Expected Usage |
|---|---|---|
| S3 | 5GB storage, 20K requests/month | ~1MB, ~100 requests |
| CloudFront | 1TB transfer, 10M requests/month | Negligible |
| Total | — | ~$0.00/month |

> Set a **$5 billing alert** in AWS just in case: Billing → Budgets → Create budget.

---

## 📄 License

MIT — feel free to use this as a template for your own projects.

---

## 🤝 Connect

- **Portfolio:** https://d32mu49xhctut.cloudfront.net/
- **LinkedIn:** [linkedin.com/in/surraj70](https://linkedin.com/in/surraj70)
- **GitHub:** [github.com/Surraj70](https://github.com/Surraj70)
