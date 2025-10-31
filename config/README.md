# Configuration Management

## Security Best Practices

### ✅ Safe to Commit (Non-Sensitive)
- Configuration structure
- Default values
- Example configurations

### 🔒 Must Use Secrets (Sensitive)
- AWS Account IDs (if organization policy requires)
- IAM Role ARNs (best practice: use secrets)
- ECR Registry URLs

**Note:** CI/CD has ECR-only permissions. S3 config is not used by CI/CD.

### 🔐 Logging & Masking in CI/CD
- CI/CD workflow masks `account_id`, `iam_role_arn`, và `ecr_registry` và không in ra logs.
- Chỉ hiển thị `region` và nguồn cấu hình (Env/Secrets hoặc `config/aws.json`).
- Tránh rò rỉ danh tính tài khoản/ARN trên logs công khai.

## Configuration Priority

The system loads configuration in this order (highest to lowest priority):

1. **GitHub Secrets** (recommended for production)
2. **Environment Variables**
3. **config/aws.json** (fallback for development)

> Region không có giá trị mặc định trong CI/CD. Nếu thiếu `AWS_REGION` ở Secrets/ENV và `config/aws.json`, workflow sẽ fail sớm.

## Setup Options

### Option 1: GitHub Secrets (Recommended for Production)

1. Go to Repository Settings → Secrets and variables → Actions
2. Add the following secrets (ECR-only permissions):
   - `AWS_ACCOUNT_ID`: Your AWS account ID
   - `AWS_REGION`: AWS region (e.g., ap-southeast-2)
   - `AWS_IAM_ROLE_ARN`: IAM role ARN for GitHub Actions (ECR permissions only)
   - `AWS_ECR_REGISTRY`: ECR registry URL

3. The workflow will automatically use these secrets if available

### Option 2: config/aws.json (For Local Development)

1. Copy `config/aws.example.json` to `config/aws.json`
2. Fill in your actual values
3. **Important**: Add `config/aws.json` to `.gitignore` if it contains real values

```bash
# Add to .gitignore
echo "config/aws.json" >> .gitignore
```

### Option 3: Environment Variables

Set environment variables before running scripts:

```bash
export AWS_ACCOUNT_ID="123456789012"
export AWS_REGION="ap-southeast-2"
export AWS_IAM_ROLE_ARN="arn:aws:iam::123456789012:role/github-actions-role"
export AWS_ECR_REGISTRY="123456789012.dkr.ecr.ap-southeast-2.amazonaws.com"
```

## Files

- `aws.json`: Actual configuration (should not be committed if contains real values)
- `aws.example.json`: Example configuration (safe to commit)
- `aws.json.template`: Template with environment variable placeholders
- `aws.schema.json`: JSON schema for validation

## Migration Guide

### From JSON-only to Secrets

1. Add values to GitHub Secrets
2. Keep `config/aws.json` as fallback for local development
3. Remove real values from `config/aws.json` if committed
4. Use `config/aws.example.json` for documentation

## Validation

Config được tự động validate trong CI/CD workflow (JSON Schema `config/aws.schema.json`). 

Để validate JSON syntax local:
```bash
jq empty config/aws.json
```

