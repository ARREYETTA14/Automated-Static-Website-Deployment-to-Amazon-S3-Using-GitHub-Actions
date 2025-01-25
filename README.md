# Automated-Static-Website-Deployment-to-Amazon-S3-Using-GitHub-Actions
This project automates the deployment of a static website to Amazon S3 using GitHub Actions. It features a workflow that syncs updates to the S3 bucket upon repository changes, enabling scalable and cost-effective static content hosting via a public URL.

# 1. Prepare Your Files
- Create the main website page (```index.html```) with the following content:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Welcome</title>
</head>
<body>
    <h1>Welcome to Arrey's Data Center</h1>
    <p>This is the main page of the static website.</p>
</body>
</html>
```
- Create the error page (```error.html```) with the following content:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Error</title>
</head>
<body>
    <h1>Oops! Something went wrong.</h1>
    <p>The website is currently unavailable. Please try again later.</p>
</body>
</html>
```

# 2. Create and Configure the S3 Bucket
- **Log in to AWS Management Console** and navigate to the **S3** service.
- **Create a new bucket**:
  - Click **Create bucket**.
  - Enter a unique bucket name (e.g., ```arrey-static-website```).
  - Select the **Region** closest to your audience.
  - Uncheck **Block all public access**, then confirm you want to make the bucket public.
  - Click **Create bucket**.

# 3. Enable Static Website Hosting
- Go to the **Properties** tab of your bucket.
- Scroll down to **Static website hosting** and click **Edit**.
- **Enable** static website hosting.
- Set:
  - Index document to ```index.html```.
  - Error document to ```error.html```.
- Save the changes.

# 4. Store your AWS Secrets in GitHub
- Go to your GitHub repository.
- Navigate to **Settings** > **Secrets and variables** > **Actions** > **New repository secret**.
- Add the following secrets:
  - ```AWS_ACCESS_KEY_ID```: Your IAM user’s Access Key ID.
  - ```AWS_SECRET_ACCESS_KEY```: Your IAM user’s Secret Access Key.
  - ```S3_BUCKET_NAME```: Your S3 bucket name.
  - ```AWS_REGION```: The region where your bucket is hosted (e.g., ```sa-east-1```)

# 5. Create Github Repository add Files to Your Repository
- Upload your ```index.html``` and ```error.html``` files to your repo. You can create a folder structure or directly add the files at the root of the repository

# 6. Create a GitHub Actions Workflow
- clone the **clone** the github repo in yout **vs code**
- Create the following folder structure in the repository you created: ```.github/workflows/```.
- Inside this folder, create a file named ```deploy.yml```.

# 7. Define the Workflow
- Add the following YAML code to automate the deployment:
```yml
name: Deploy Static Website to S3

on:
  push:
    branches:
      - main  # Trigger deployment on pushes to the main branch

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      # Step 1: Checkout code from the repository
      - name: Checkout Code
        uses: actions/checkout@v3

      # Step 2: Configure AWS Credentials
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      # Step 3: Deploy files to S3
      - name: Deploy to S3
        run: |
          aws s3 sync . s3://${{ secrets.S3_BUCKET_NAME }} --delete
```

# 8. Commit and Push Changes
- After adding the workflow file, commit it to your GitHub repository. You can do this with Git commands:
```bash
git add .github/workflows/deploy.yml
git commit -m "Add GitHub Actions workflow for S3 deployment"
git push origin main
```

# 9. Watch the Action Run
Once the code is pushed to GitHub, go to the Actions tab of your repository.
You'll see the workflow trigger automatically because you pushed changes to the main branch.
If everything is set up correctly, the workflow will upload your website files to your S3 bucket.

# 10. Testing 
- Check if the GitHub files where uploaded in your **s3**
- In the bucket details, under the **Properties** tab, scroll down to the **Static website hosting** section.
- You should see a **Bucket website endpoint** URL, which will look like:
```php
http://<your-bucket-name>.s3-website-<AWS-region>.amazonaws.com/
```
