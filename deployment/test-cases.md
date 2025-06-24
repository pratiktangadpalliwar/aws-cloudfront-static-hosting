# ✅ Test Cases: AWS S3 + CloudFront Static Website Hosting

This document outlines test scenarios to verify that your static website is correctly configured and secured with AWS CloudFront, WAF, custom error pages, and geo-restriction.

---

## 🔹 1. Homepage Load Test

**Action**: Open your CloudFront domain in a browser

**Expected Result**: `index.html` loads and displays correctly

**Notes**:
- Verify that HTTPS is automatically enforced
- Check page rendering and CSS/JS (if included)

---

## 🔹 2. Custom 404 Error Page Test

**Action**: Open a non-existent page  
Example: `https://<cloudfront-domain>/nonexistent.html`

**Expected Result**: `error.html` displays with 404 status

**Notes**:
- CloudFront should return your custom `error.html`
- Response code in dev tools: `404`

---

## 🔹 3. Custom 403 Error Page Test (Geo-Restriction)

**Action**: Try accessing the site from a **blocked country**  
(use VPN set to a blacklisted country like China or Russia)

**Expected Result**: `block.html` appears with a 403 status

**Notes**:
- Response code: `403`
- Content of `block.html` should match what was uploaded

---

## 🔹 4. WAF SQL Injection Test

**Action**: Open a URL with SQL injection-like parameters  
Example: `https://<cloudfront-domain>/?id=1'+OR+1=1--`

**Expected Result**: 403 Forbidden response

**Notes**:
- Response should **not reach S3**
- Verify WAF blocked the request (check WAF logs or CloudWatch if enabled)

---

## 🔹 5. Public Access via S3 Bucket URL

**Action**: Try to access the site using the raw S3 URL (bucket endpoint)

**Expected Result**: Accessible only if public; otherwise, denied

**Notes**:
- If bucket policy allows direct public access, the site loads (intentional in this case)
- For production, consider private buckets with CloudFront OAI

---

## 🔹 6. Cache and Redirect Test

**Action**: Open the site with `http://` instead of `https://`

**Expected Result**: Automatically redirected to `https://` by CloudFront

**Notes**:
- Test multiple endpoints
- Verify cache behavior if you updated static files recently

---