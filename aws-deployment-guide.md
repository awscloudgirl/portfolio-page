# AWS S3 Deployment Guide for Portfolio Website

This guide outlines the detailed steps to deploy a static portfolio website on Amazon S3, ensuring it is configured securely and efficiently according to best practices.

## Prerequisites

- An AWS account.
- Basic knowledge of AWS services.
- Your portfolio website files (HTML, CSS, JavaScript, images).

## Steps to Deploy on Amazon S3

### Step 1: Create an S3 Bucket

1. **Log in** to the AWS Management Console.
2. Navigate to the **S3 service**.
3. Click **Create bucket**.
4. Provide a **bucket name** that is globally unique.
5. Select the **Region** that is geographically closest to your expected users to reduce latency. This is important as it affects the performance of your site.
6. Uncheck **Block all public access** settings, as this bucket will need to allow public access to serve your website.
7. Click **Create bucket**.

### Step 2: Enable Static Website Hosting

1. Select your newly created bucket from the bucket list.
2. Go to the **Properties** tab.
3. Scroll down and click on **Static website hosting**.
4. Choose **Use this bucket to host a website**.
5. Enter `index.html` for the index document and also for the error document.
6. Note the **Endpoint URL** provided; it will be used to access your website publicly.

### Step 3: Upload Your Website Files

1. Go to the **Objects** tab within your bucket.
2. Click **Upload**.
3. Add your website files.
4. Ensure that during the upload process, you set the permissions for these files to **Grant public-read access** so they are accessible to visitors.

### Step 4: Set Bucket Policy

1. Navigate to the **Permissions** tab of your bucket.
2. Click **Bucket Policy**.
3. Enter the following policy, replacing `your-bucket-name` with the actual name of your bucket:

    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Sid": "PublicReadForCertainTypes",
          "Effect": "Allow",
          "Principal": "*",
          "Action": "s3:GetObject",
          "Resource": [
            "arn:aws:s3:::your-bucket-name/*.html",
            "arn:aws:s3:::your-bucket-name/*.css",
            "arn:aws:s3:::your-bucket-name/*.js",
            "arn:aws:s3:::your-bucket-name/*.jpg",
            "arn:aws:s3:::your-bucket-name/*.jpeg",
            "arn:aws:s3:::your-bucket-name/*.png",
            "arn:aws:s3:::your-bucket-name/*.svg"
          ],
          "Condition": {
            "Bool": {
              "aws:SecureTransport": "true"
            }
          }
        }
      ]
    }
    ```

4. This policy allows public access to your website files while enforcing that access is over HTTPS, securing data in transit.

### Step 5: Test Your Website

1. Open a browser and navigate to the Endpoint URL noted earlier.
2. Ensure that the website loads properly and all links, images, and pages are accessible as expected.

## Conclusion

Following these steps ensures that your portfolio website is deployed securely on AWS S3. The setup includes public access permissions configured specifically for typical web file formats and mandatory HTTPS for secure access. Regularly review and update your configurations as needed to maintain security and functionality.

