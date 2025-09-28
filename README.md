🚀 Build, Scan, and Push to Amazon ECR with GitHub Actions

This project demonstrates a CI/CD pipeline using GitHub Actions to automate the process of building a Docker image, scanning it for vulnerabilities, and pushing it to Amazon Elastic Container Registry (ECR).

It’s designed as a DevOps best-practice project to showcase automation, security scanning, and image versioning in a cloud-native workflow.

📌 Workflow Overview

The GitHub Actions workflow is triggered on every push to the following branches:

main

dev

prod

🔄 Key Stages

Checkout repository – Pulls the source code.

Configure AWS credentials – Authenticates with AWS using GitHub secrets.

Login to Amazon ECR – Gets credentials to push Docker images.

Generate image tags – Creates tags based on:

Branch name

GitHub commit SHA (short)

Timestamp

Build Docker image – Builds the image with a unique tag.

Trivy Scan – Scans the Docker image for vulnerabilities (HIGH, CRITICAL).

Push to ECR – Pushes two tags:

branch-SHA-timestamp

branch-latest

📂 Project Structure

.
├── .github/
│   └── workflows/
│       └── build-scan-push.yml   # GitHub Actions workflow (build, scan, push to ECR)
├── Dockerfile                    # Docker build instructions
├── src/                          # Application source code
│   ├── app.py                    # Example application file
│   └── requirements.txt          # Dependencies list
├── README.md                     # Project documentation

🔎 Explanation

.github/workflows/ → Contains the GitHub Actions pipeline.

Dockerfile → Defines how to build the Docker image for the app.

src/ → Application source code and dependencies.

README.md → Documentation for the project.

⚙️ Example Image Tags

For a commit pushed to the dev branch:

dev-a1b2c3d-20250928130045

dev-latest

🔐 Required Secrets

Add the following secrets in your GitHub repository (Settings > Secrets and variables > Actions):

AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

📂 Workflow File Example

.github/workflows/build-scan-push.yml

name: Build, Scan, and Push to ECR

on:
  push:
    branches: [ "main", "dev", "prod" ]

jobs:
  build-scan-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2
      - name: Set image tags
        run: |
          echo "SHORT_SHA=${GITHUB_SHA::7}" >> $GITHUB_ENV
          echo "TIMESTAMP=$(date +'%Y%m%d%H%M%S')" >> $GITHUB_ENV
          echo "BRANCH=$(echo ${GITHUB_REF_NAME} | tr '/' '-')" >> $GITHUB_ENV
      - name: Build Docker image
        run: |
          docker build -t my-app:${{ env.BRANCH }}-${{ env.SHORT_SHA }}-${{ env.TIMESTAMP }} .
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@0.20.0
        with:
          image-ref: my-app:${{ env.BRANCH }}-${{ env.SHORT_SHA }}-${{ env.TIMESTAMP }}
          format: table
          exit-code: 1
          severity: HIGH,CRITICAL
      - name: Tag and Push image to ECR
        if: success()
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: my-app
        run: |
          IMAGE_TAG=${{ env.BRANCH }}-${{ env.SHORT_SHA }}-${{ env.TIMESTAMP }}
          docker tag my-app:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker tag my-app:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:${{ env.BRANCH }}-latest
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:${{ env.BRANCH }}-latest

🛡 Features & Best Practices

✅ Automated Docker builds per branch

✅ Security scanning with Trivy

✅ Dynamic tagging (branch + commit + timestamp)

✅ Pushes both unique and latest tags per branch

✅ Uses GitHub secrets for AWS credentials

🚀 How to Use

Fork this repository or copy the workflow into your project.

Create an Amazon ECR repository (e.g., my-app).

Add your AWS credentials to GitHub secrets.

Push code to main, dev, or prod branches.

The pipeline will automatically build, scan, and push your image.

📊 Example Run

Push code to dev branch.

GitHub Actions builds Docker image.

Trivy scans vulnerabilities.

Image is pushed to Amazon ECR as:

dev-<short_sha>-<timestamp>

dev-latest

⚡ This project is a hands-on DevOps practice for CI/CD, Docker, AWS ECR, and Security Scanning.