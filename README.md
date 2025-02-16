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
  - Go to **permissions** tab and **create bucket policy** policy:

```json
  {
  "Id": "Policy1737839385249",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1737839383542",
      "Action": "s3:*",
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::arrey-data-center",
      "Principal": "*"
    }
  ]
}
```

# 3. Enable Static Website Hosting
- Go to the **Properties** tab of your bucket.
- Scroll down to **Static website hosting** and click **Edit**.
- **Enable** static website hosting.
- Set:
  - Index document to ```index.html```.
  - Error document to ```error.html```.
- Save the changes.

# 4. Store your AWS Secrets on GitHub
- Create a GitHub repository.
- Navigate to **Settings** tab > **Secrets and variables** > **Actions** > **New repository secret**.

![Image](https://github.com/user-attachments/assets/99e1ac56-2f60-4523-a6a9-de84fe6f365e)

![Image](https://github.com/user-attachments/assets/d13f9781-8106-4ccd-a83e-c340344f27c8)

- Add the following secrets:
  - ```AWS_ACCESS_KEY_ID```: Your IAM user’s Access Key ID.
  - ```AWS_SECRET_ACCESS_KEY```: Your IAM user’s Secret Access Key.
  - ```S3_BUCKET_NAME```: Your S3 bucket name.
  - ```AWS_REGION```: The region where your bucket is hosted (e.g., ```sa-east-1```).

![Image](https://github.com/user-attachments/assets/3cb943f9-5b9f-4ee1-84da-9268a268d012)

# 5. Add Files to Your Repository
- Upload your ```index.html``` and ```error.html``` files to your repo. You can create a folder structure or directly add the files at the root of the repository

![Image](https://github.com/user-attachments/assets/14089f0f-d405-41cd-8842-d1f174507187)

# 6. Create a GitHub Actions Workflow
- **Clone** the GitHub repo in your **vs code**
- Create the following folder structure in the repository you created: ```.github/workflows/```.
- Inside this folder, create a file named ```deploy.yml```.

![Image](https://github.com/user-attachments/assets/98e49c5a-f33a-4d0d-8186-bc02ddb532ec)

# 7. Define the Workflow
- Add the following YAML code in the ```deploy.yml``` file to automate the deployment:

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

      # Step 4: Set public read ACL for index.html and error.html files
      - name: Apply Public Read ACL to index.html and error.html
        run: |
          aws s3api put-object-acl --bucket ${{ secrets.S3_BUCKET_NAME }} --acl public-read --key index.html
          aws s3api put-object-acl --bucket ${{ secrets.S3_BUCKET_NAME }} --acl public-read --key error.html

```

# 8. Commit and Push Changes
- After adding the workflow file, commit it to your GitHub repository. You can do this with Git commands:

```bash
git add .github/workflows/deploy.yml
git commit -m "Add GitHub Actions workflow for S3 deployment"
git push origin main
```

# 9. Watch the Action Run
Once the code is pushed to GitHub, go to the **Actions** tab of your repository.
You'll see the workflow trigger automatically because you pushed changes to the main branch.
If everything is set up correctly, the workflow will upload your website files to your S3 bucket.

![Image](https://github.com/user-attachments/assets/a9dcfd42-d002-46f5-b335-4183a1ea21b3)

![Image](https://github.com/user-attachments/assets/0dd1eb0a-6231-4100-90c0-fe66f1b8f9fb)

# 10. Testing 
- Check if the GitHub files were uploaded in your **s3**

![Image](https://github.com/user-attachments/assets/d5623fb9-4993-454a-bb82-d772aa785e54)

- In the bucket details, under the **Properties** tab, scroll down to the **Static website hosting** section.
- You should see a **Bucket website endpoint** URL, which will look like:

```php
http://<your-bucket-name>.s3-website-<AWS-region>.amazonaws.com/
```
- Open your web browser.
- Paste the S3 bucket website URL (e.g., ```http://<your-bucket-name>.s3-website-<AWS-region>.amazonaws.com/```).
- If everything is working, the ```index.html``` page should load, and you should see your website content.

![Image](https://github.com/user-attachments/assets/40103de8-660d-4adf-946d-0cf143561822)

**Simulate an Error**

- To test if the **error.html** page works, you can try accessing a non-existent page or file in your S3 bucket:
    - In your browser, append a random filename (such as **nonexistentfile.html**) to the end of the URL:
  
```php
http://<your-bucket-name>.s3-website-<AWS-region>.amazonaws.com/nonexistentfile.html
```

![Image](https://github.com/user-attachments/assets/dde6a3dc-75f8-4ac5-aa27-5a4123e2dbbd)

- Any change made on the ```index.html``` and ```error.html``` files will automatically be upates on the website on the browser. Just refresh when **Github Action** is done deploying the change.


**NB**: If after this you have an **ACCESS DENIED**, **Check Permissions for Each Object**. 
Each file (such as ```index.html``` and ```error.html```) must have the correct permissions to be publicly accessible.

- **Go to Your Objects** (e.g., ```index.html```):
    - Select your **index.html file**.
- Verify Object Permissions:
    - Go to the **Permissions** tab for that object.
    - Under **Access control list (ACL)**, you may need to ensure that  **Everyone(public access)** has **Read** permission.
 
![Image](https://github.com/user-attachments/assets/e70c72ee-f368-4c07-83e1-03018eaa74b1)

- Repeat for ```error.html```

- This should resolve the **ACCESS DENIED** issue.



# THANK YOU




