# **AWS S3 + CloudFront Deployment Guide for Portfolio Website**  

This guide details the full process of deploying a **static portfolio website** using **AWS S3, CloudFront, and Route 53**, ensuring **HTTPS security, DNS configuration, and troubleshooting**.  

---

## **🚀 Prerequisites**
Before starting, make sure you have:
- ✅ **An AWS account**
- ✅ **Basic AWS knowledge** (S3, CloudFront, Route 53)
- ✅ **Your website files** (`index.html`, `style.css`, images)
- ✅ **A registered domain** (via AWS Route 53 or an external registrar)

---

## **📌 Step 1: Create an S3 Bucket for Static Website Hosting**  

1. **Log in** to the AWS Management Console.
2. Navigate to **S3**.
3. Click **Create bucket**.
4. Set a **globally unique bucket name**, e.g., `awscg-portfolio.site`.
5. **Region Selection**:
   - **Important:** Choose a region **close to your users**.
   - If using **CloudFront**, region selection is flexible.
   - If using **S3 website hosting only**, performance may be affected by region choice.
6. **Uncheck** `Block all public access` (Required for website access).
7. Click **Create bucket**.

---

## **🌍 Step 2: Enable Static Website Hosting on S3**  

1. Select your **S3 bucket**.
2. Go to the **Properties** tab.
3. Scroll to **Static website hosting**.
4. Choose **Use this bucket to host a website**.
5. Set:
   - **Index document** → `index.html`
   - **Error document** → `index.html` (or `error.html` if needed).
6. Copy the **"Endpoint URL"**:
   - Example:
     ```
     http://awscg-portfolio.site.s3-website.eu-west-2.amazonaws.com
     ```
   - **⚠️ Note:** S3 Website Endpoints **do NOT support HTTPS**, so we will use **CloudFront**.

---

## **📤 Step 3: Upload Website Files to S3**  

1. Open your **S3 bucket**.
2. Go to the **Objects** tab.
3. Click **Upload** → Add your **HTML, CSS, and images**.
4. Under **Permissions**, ensure:
   - **Grant public read access**.
5. Click **Upload**.

---

## **🔑 Step 4: Configure S3 Bucket Policy for Public Access**  

Since you did **not** use `SecureTransport`, here’s the **exact policy** that worked:

1. Navigate to **Permissions** → Click **Bucket Policy**.
2. Paste this **policy** (replace `your-bucket-name`):

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
            "arn:aws:s3:::your-bucket-name/*.jpg",
            "arn:aws:s3:::your-bucket-name/*.jpeg",
            "arn:aws:s3:::your-bucket-name/*.png",
            "arn:aws:s3:::your-bucket-name/*.svg"
          ]
        }
      ]
    }
    ```

3. This allows **public read access** for web files **without SecureTransport**.

---

## **🌐 Step 5: Create a CloudFront Distribution (for HTTPS & Performance)**  

Since **S3 website hosting doesn’t support HTTPS**, we use **CloudFront**.

1. Navigate to **CloudFront**.
2. Click **Create Distribution**.
3. Under **Origin**:
   - **Origin Domain** → Choose **S3 website endpoint** (not the bucket name).
   - **Protocol** → **HTTP only** (CloudFront handles HTTPS).
4. **Default Cache Behavior**:
   - **Viewer Protocol Policy** → **Redirect HTTP to HTTPS**
   - **Allowed HTTP Methods** → `GET, HEAD`
5. **Settings**:
   - **Price Class** → "Use all edge locations (best performance)"
   - **Alternate Domain Names (CNAMEs)**:
     - `awscg-portfolio.site`
     - `www.awscg-portfolio.site`
   - **SSL Certificate**:
     - Use **AWS Certificate Manager (ACM)**
     - If missing, **request a new SSL certificate**.
6. Click **Create Distribution** and wait for it to deploy.

---

## **🔁 Step 6: Configure Route 53 DNS Records**  

### **❌ Deleted Wrong Records? Fix Route 53!**
- You initially had an **A Record** and **CNAME**.
- Route 53 records **must** point to **CloudFront**, not S3.

1. Go to **Route 53** → **Hosted Zones**.
2. Select your domain (`awscg-portfolio.site`).
3. **Delete old A records pointing to S3**.
4. **Create an A Record (Alias)**
   - **Name**: (leave blank for root domain)
   - **Alias**: Yes
   - **Route traffic to** → **CloudFront distribution**.
5. **Create a CNAME Record**
   - **Name**: `www`
   - **Alias**: No
   - **Value**: `dc7omfocll6y.cloudfront.net` (or your CloudFront URL).
6. **Save the records** → Wait **30 minutes** for DNS propagation.

---

## **🛠️ Troubleshooting & Fixing Issues**  

### **🚨 Website Not Loading?**
- **Check Route 53 records**: Ensure **A record points to CloudFront**.
- **Check CloudFront status**: If "In Progress," wait until it's "Deployed."
- **Clear Cache**:
  ```
  Ctrl + Shift + R (Windows)
  Cmd + Shift + R (Mac)
  ```

### **🚨 No HTTPS?**
1. Go to **CloudFront → Edit Distribution**.
2. Ensure:
   - `Redirect HTTP to HTTPS` is enabled.
   - Your **SSL certificate** is active.
   - **Route 53 records point to CloudFront** (not S3!).

### **🚨 NSLookup Troubleshooting**
- Run:
  ```
  nslookup awscg-portfolio.site
  ```
  If it **resolves to CloudFront IPs** (`54.230.x.x`), ✅ **it's correct**.  
  If it **resolves to S3**, ❌ **Fix Route 53**.

### **🚨 SSL Certificate Not Valid?**
- Verify SSL via:
  ```
  https://www.sslshopper.com/
  ```
- Ensure CloudFront **CNAMEs match your domain**.
- Check ACM → The certificate **must be in `us-east-1`**.

### **🚨 Site Still Shows "Not Secure"?**
- Use Chrome DevTools (`F12`) → **Console Tab**.
- If you see **Mixed Content Errors**, fix all **HTTP URLs** in your code.

---

## **🎉 Final Check**
1. Open:
   ```
   https://awscg-portfolio.site
   ```
2. Confirm:
   ✅ Page loads  
   ✅ HTTPS is working  
   ✅ No security warnings  
   ✅ DNS is resolving correctly  

---

## **📌 Conclusion**
This guide covers **exactly** what worked for you:
- ✅ **Bucket policy without SecureTransport**
- ✅ **CloudFront over S3 for HTTPS**
- ✅ **A + CNAME Records in Route 53**
- ✅ **Region-specific considerations**
- ✅ **Step-by-step debugging**

Now, you have a **fully deployed, secure, and scalable website on AWS**! 🎉🚀  
Keep this guide handy for future deployments. Let me know if you'd like any refinements! 🔥
