# H&M Deep Learning - ML Monorepo

Monorepo chuẩn BigTech cho dự án Machine Learning/Deep Learning.

## Cấu trúc dự án

```
.
├── components/              # Các ML components
│   ├── data_ingestion/     # Đổ data raw từ nhiều nguồn vào S3
│   │   ├── src/           # Source code
│   │   ├── Dockerfile     # Docker image definition
│   │   ├── requirements.txt
│   │   ├── config.yaml    # Component configuration
│   │   └── .dockerignore
│   ├── data_processing/    # Xử lý dữ liệu từ raw sang processed
│   ├── data_eda/           # Exploratory Data Analysis
│   ├── train/              # Training models
│   └── inference/          # Model inference và serving
├── config/                 # Configuration files
│   ├── aws.json           # AWS configuration
│   └── aws.schema.json    # AWS config JSON schema
├── .github/workflows/      # CI/CD pipelines
│   └── build-and-push-ecr.yml
├── Makefile               # Build commands
├── pyproject.toml         # Python project configuration
└── README.md
```

## Components

### 1. Data Ingestion
- **Mục đích**: Đổ data raw từ nhiều nguồn vào S3 raw bucket
- **Output**: Data trong S3 bucket tại `raw/` prefix

### 2. Data Processing
- **Mục đích**: Xử lý dữ liệu từ raw sang processed format
- **Input**: `raw/` prefix trong S3
- **Output**: `processed/` prefix trong S3

### 3. Data EDA
- **Mục đích**: Exploratory Data Analysis
- **Input**: `processed/` prefix trong S3
- **Output**: EDA reports và visualizations

### 4. Train
- **Mục đích**: Train ML models
- **Input**: `processed/` prefix trong S3
- **Output**: Model artifacts tại `artifacts/` prefix trong S3

### 5. Inference
- **Mục đích**: Model inference và serving
- **Input**: Model artifacts từ S3
- **Output**: API endpoints cho predictions

## AWS Configuration

### 🔒 Security-First Approach

**Configuration Priority (Highest to Lowest):**
1. **GitHub Secrets** (recommended for production) 🔐
2. **Environment Variables**
3. **config/aws.json** (fallback for local development)

### Setup Options

#### Option 1: GitHub Secrets (Recommended)

1. Go to **Repository Settings → Secrets and variables → Actions**
2. Add these secrets (ECR-only permissions):
   - `AWS_ACCOUNT_ID`
   - `AWS_REGION`
   - `AWS_IAM_ROLE_ARN`
   - `AWS_ECR_REGISTRY`

3. Workflow tự động sử dụng secrets nếu có

#### Option 2: config/aws.json (Local Development)

```bash
# Copy example file
cp config/aws.example.json config/aws.json

# Fill in your values (file is gitignored)
# For production, use GitHub Secrets instead!
```

#### Option 3: Environment Variables

```bash
export AWS_ACCOUNT_ID="123456789012"
export AWS_REGION="ap-southeast-2"
# ... etc
```

### Config Structure

- `config/aws.json`: Actual config (gitignored if contains real values)
- `config/aws.example.json`: Example config (safe to commit)
- `config/aws.json.template`: Template with env var placeholders
- `config/aws.schema.json`: JSON schema for validation

See [config/README.md](config/README.md) for detailed documentation.

### Validation

Config được tự động validate với:
- ✅ JSON syntax validation
- ✅ Required fields checking
- ✅ Format validation (account ID, region, ARN patterns)
- ✅ Priority-based loading (Secrets > ENV > JSON)

### Local Development

Config được validate và load tự động trong CI/CD workflow. Không cần scripts riêng.

Để test config local, bạn có thể validate JSON syntax:
```bash
jq empty config/aws.json
```

### CI/CD Integration

Workflow tự động:
1. ✅ Load từ GitHub Secrets (nếu có) hoặc config file
2. ✅ Validate tất cả config values
3. ✅ Sử dụng IAM role cho authentication
4. ✅ Create ECR repositories với security best practices
5. ✅ Retry logic với error handling

## CI/CD

CI/CD pipeline với các tính năng:

### Smart Build Detection
- ✅ Chỉ build components có thay đổi (git diff)
- ✅ Build tất cả nếu `config/aws.json` thay đổi
- ✅ Hỗ trợ cả push và pull request events

### Robust Configuration Management
- ✅ Validate AWS config trước khi sử dụng
- ✅ Đọc toàn bộ config từ `config/aws.json` (không hardcode)
- ✅ Error handling và validation chi tiết
- ✅ Auto-create ECR repositories nếu thiếu

### Security & Reliability
- ✅ IAM role-based authentication
- ✅ Retry logic cho push operations (3 attempts)
- ✅ Image scanning enabled trên ECR
- ✅ Encryption enabled (AES256)
- ✅ Role session naming với run ID

### Workflow trigger
- Push vào `main` hoặc `develop` branch
- Pull requests vào `main` hoặc `develop` branch

### Image Tagging (Immutable BigTech ML Standard)

**Quy ước tagging immutable (không dùng `latest`):**

- **Main branch**: `main-<short-sha>` (primary) + `main-latest` (additional)
- **Develop branch**: `develop-<short-sha>` (primary) + `develop-latest` (additional)
- **Pull requests**: `pr-<pr-number>-<short-sha>` (primary) + `pr-<pr-number>` (additional)
- **Feature branches**: `<branch-name>-<short-sha>` (primary) + full SHA (additional)

**Ví dụ:**
- Main: `data_ingestion:main-a1b2c3d` (immutable production tag)
- Develop: `data_processing:develop-x9y8z7w`
- PR #42: `train:pr-42-f5e4d3c`

**Lý do không dùng `latest`:**
- ✅ Immutable tags cho reproducibility
- ✅ Dễ rollback và track versions
- ✅ Tránh race conditions
- ✅ Production-ready best practice

## Development

### Local development
```bash
# Build image cho một component
cd components/data_ingestion
docker build -t data_ingestion:local .

# Run component
docker run --env-file .env data_ingestion:local

# Hoặc sử dụng Makefile
make build-component COMPONENT=data_ingestion
make push-component COMPONENT=data_ingestion TAG=latest
```

### Thêm dependencies
Cập nhật `requirements.txt` trong component tương ứng.

## Notes

- Tất cả components sử dụng Python 3.10
- Images được push lên ECR registry: `465002806239.dkr.ecr.ap-southeast-2.amazonaws.com`
- CI/CD chỉ build và push components có thay đổi (thông minh)

