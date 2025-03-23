# CI/CD from GitHub to AWS S3 Website Hosting

This project demonstrates how to set up a CI/CD pipeline to automatically deploy a static website from a GitHub repository to an AWS S3 bucket.

## Steps

### 1. Create an S3 Bucket
1. Navigate to **Amazon S3** → **Buckets** → **Create bucket**.
2. Assign a **Bucket Name**.
3. Under **Object Ownership**, check **ACLs enabled**.
4. Under **Block Public Access settings for this bucket**, uncheck **Block all public access** and click the **I acknowledge** button.
5. Navigate to the new bucket → **Properties** → **Static website hosting** → **Edit**.
6. Select **Enable**, and specify the name of your HTML home directory file (e.g., `index.html`).

### 2. Create an IAM User
1. Navigate to **IAM** → **Users** → **Create user**.
2. Assign a **Username**.
3. Select **Attach policies directly** → **Permissions policies** → `AmazonS3FullAccess`.
4. Navigate to **Security Credentials** → **Create Access Key** → **Other**.

### 3. Create a GitHub Repository
1. Create a new GitHub repository and assign a **Repository Name**.
2. Check **Add a README file** and select **Create repository**.
3. Navigate to **Settings** → **Secrets and variables** → **Actions** → **New repository secret**.
4. Create the following repository secrets:
   - `AWS_ACCESS_KEY_ID`: Assign the public access key of the IAM user.
   - `AWS_SECRET_ACCESS_KEY`: Assign the secret access key of the IAM user.
   - `S3_BUCKET_NAME`: Assign the name of your S3 bucket.
   - `AWS_REGION`: Assign the AWS region where your S3 bucket is hosted.

### 4. Create an S3 Bucket Policy
1. Navigate to your **S3 Bucket** → **Permissions** → **Bucket Policy** → **Edit**.
2. Select **Policy Generator**.
   - **Type of Policy**: S3 Bucket Policy.
   - **Principal**: Paste the ARN of the IAM user you created.
   - **Actions**: Select **All Actions**.
   - **Amazon Resource Name (ARN)**: Paste the ARN of your S3 bucket.
3. Click **Generate Policy** and copy the generated policy.
4. Navigate to **Bucket** → **Permissions** and paste the policy.
5. Click **Save Changes**.

### 5. Create a GitHub Actions Workflow
1. In your GitHub repository, create a new file: `.github/workflows/deploy-to-s3.yml`.
2. Paste the following code:

```yaml
on:
  push:
    branches:
      - main  # Change to your main branch

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Sync Website Files to S3 with Public Read Access
        run: |
          aws s3 sync . s3://${{ secrets.S3_BUCKET_NAME }} --delete --acl public-read
```

3. Click **Commit changes**, ensuring that **Commit to the main branch** is selected.

## Conclusion
Once these steps are completed, every push to the `main` branch of your GitHub repository will automatically trigger a deployment to your AWS S3 bucket. Your static website will be live and continuously updated with each push! 🚀
