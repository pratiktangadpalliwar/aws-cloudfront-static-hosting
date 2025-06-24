# 🚀 Step-by-Step Deployment Guide: Secure Static Website with S3 + CloudFront

This guide explains how to deploy a secure static website using Amazon S3, CloudFront, and AWS WAF, including custom error pages and geo-restriction.

---

## 🧱 Prerequisites

- AWS account
- Basic HTML files: `index.html`, `error.html`, `block.html`
- Admin access to S3, CloudFront, and WAF services

---

## 🔹 Step 1: Create an S3 Bucket & Enable Static Website Hosting

1. Go to **Amazon S3** → *Create bucket*
2. Set a name like `<bucket_name>`
3. Enable:
   - Object Ownership → **ACLs enabled**
   - Object Writer as default owner
   - Public access (allow public read/write access for this demo)
4. After creation:
   - Go to **Properties** → Enable **Static Website Hosting**
   - Set:
     - Hosting type: *Host a static website*
     - Index document: `index.html`

---

## 🔹 Step 2: Upload Website Files

1. Upload the following HTML files to the S3 bucket:
   - `index.html` → Main homepage
   - `error.html` → 404 error page
   - `block.html` → 403 block page

---

## 🔹 Step 3: Edit Bucket Policy

1. Go to the **Permissions** tab of the S3 bucket
2. Edit the **Bucket Policy** and paste the following:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": "<resource_arn>"
        },
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "<resource_arn/*>"
        }
    ]
}

---

## 🔹 Step 4: Create a CloudFront Distribution

1. Go to **CloudFront** → *Create Distribution*
2. Set the following options:

- **Origin domain**: Use the **S3 static website endpoint** (not the bucket ARN)
- **Viewer protocol policy**: Redirect HTTP to HTTPS
- **Cache policy**: Use the default caching settings

3. Leave WAF and geo settings off for now
4. Click **Create Distribution**

---

## 🔹 Step 5: Configure Custom Error Pages

1. Go to your CloudFront distribution → *Error Pages* → *Create Custom Error Response*

### For 404 (Not Found):
- **HTTP error**: `404`
- **TTL**: `10` seconds
- **Customize response**: Yes
- **Response page path**: `/error.html`
- **HTTP response code**: `404`

### For 403 (Forbidden):
- **HTTP error**: `403`
- **TTL**: `10` seconds
- **Customize response**: Yes
- **Response page path**: `/block.html`
- **HTTP response code**: `403`

---

## 🔹 Step 6: Create and Attach WAF ACL

1. Go to **AWS WAF & Shield** → *Create Web ACL*
2. Set the **scope** to **CloudFront (Global)**
3. Name the Web ACL and **attach it to your CloudFront distribution**
4. Add the following AWS managed rule:

- **Rule Group**: `AWSManagedRulesSQLiRuleSet`  
  ➜ Protects against common **SQL injection** attempts

5. Review and create the Web ACL

---

## 🔹 Step 7: Enable Geo-Restriction

1. Open your CloudFront distribution → *Edit*
2. Scroll to **Geo restriction**
3. Select: **Block list**
4. Choose countries to block (e.g., **Turkey**, **Pakistan**, etc.)
5. Save changes

---

## 🔹 Step 8: Test Deployment

### ✅ Website Load
- Visit your **CloudFront domain URL**
- You should see the `index.html` homepage loaded through the CDN

### ✅ Error Page Test
- Visit a non-existent path (e.g., `/test404`)  
  → You should see the **custom 404 error page (`error.html`)**

### ✅ SQL Injection Test (WAF)
- Simulate a SQLi attempt in your browser:
    https://<cloudfront-domain>/product?item=securitynumber'+OR+1=1—
  → Should return **403 Forbidden** if WAF is active

### ✅ Geo Restriction Test
- Access the CloudFront domain from a **blocked country** (or use a VPN)
  → Should show the **403 Forbidden** and load the `block.html` page

---

## 📦 Final Output Summary

- ✅ **S3 Bucket**: Hosts public static HTML content
- ✅ **CloudFront**: CDN with HTTPS, caching, and custom error support
- ✅ **AWS WAF**: Secures against SQL injection attempts
- ✅ **Geo Restriction**: Limits access from unauthorized regions