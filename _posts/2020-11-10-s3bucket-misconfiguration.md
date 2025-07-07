---
title: How Tracking My Dumbbell Delivery Led Me to an S3 Bucket Misconfiguration on a Major E-Commerce Platform
date: 2020-11-10 09:11:05 +0530
categories: [BugBounty Writeup,Web,S3 Bucket Misconfiguration,Cloud Bugs]
tags: [Bugbounty,Writeup]
excerpt: Web Challenge
---

# 🛡️ How Tracking My Dumbbell Delivery Led Me to an S3 Bucket Misconfiguration on a Major E-Commerce Platform

Security bugs often show up in the most unexpected ways. This time, it happened when I was simply tracking a fitness product I had ordered. What started as a routine delivery check turned into the discovery of a **misconfigured Amazon S3 bucket** on one of the top e-commerce websites in the industry—exposing sensitive data.

---

## 📦 The Dumbbell That Opened the Door

I had recently placed an order for a pair of dumbbells. A few days later, I opened the parcel tracking page to check on the delivery progress.

To my surprise, the **tracking page showed the same dumbbell image that was used in the product listing**.

Being curious, I right-clicked the image and opened it in a new tab. The image URL looked like this:

  ```
  https://***.ecommerce.com/www/tracker/service/track-888895.svg
  ```
<br><br>

Out of habit (and a bit of instinct), I started modifying the path.

---

## 🧵 Unraveling the URL

Here’s what I discovered step-by-step:

### 1. **Original URL (200 OK)**  
```
https://***.ecommerce.com/www/tracker/service/track-888895.svg
```

- ✅ Image loaded successfully (public access)
- 🏷️ Reused from the original product listing

### 2. **Trimmed URL – Back to `/tracker/` (200 OK)**  
```
https://***.ecommerce.com/www/tracker/
```

- ✅ Exposed a **listable S3 bucket**
- I could see and access files directly from here

### 3. **Base URL without `/tracker/` (403 Forbidden)**  
```
https://***.ecommerce.com/www/
```

- ❌ Access denied (403 Forbidden)
- Meaning: the `/tracker/` directory was the only publicly exposed path

### 4. **Adding `/service/` without a valid file (403 Forbidden)**  
```
https://***.ecommerce.com/www/tracker/service/
```

- ❌ Forbidden again
- Only specific file paths under `/tracker/service/` were exposed

---

## 🔐 What Was Exposed?

Inside the publicly accessible `/tracker/` path (S3 bucket), I found:

- Sensitive data 🤫

Some of the data, while not extremely critical, **would provide an attacker or competitor with deep insight** into internal systems.

---

## 📢 Responsible Disclosure

I responsibly reported the issue through the e-commerce platform’s security or bug bounty program. My report included:

- A detailed explanation of the path traversal  
- Screenshots and sample file links  
- Suggestions to audit and block all public access unless explicitly required

The security team acknowledged the issue quickly and patched it within 24 hours. They:

- Disabled public access to the exposed S3 path  
- Reviewed all related bucket policies  

---

## 🧠 Takeaways

### For Developers & Cloud Teams:
- **Always audit your public S3 buckets**—especially ones connected to static content delivery  
- Never reuse product media without validating its hosting context  
- Use **S3 Block Public Access** and **least privilege IAM policies**


---

## 🏁 Final Thoughts

This entire find happened by accident. I wasn’t hunting for bugs—I was just excited about my dumbbells. But that curiosity led to a serious misconfiguration that could have been abused by malicious actors.

It’s a reminder that **security is everywhere**, and sometimes, even package tracking pages have stories to tell.

Stay curious, stay ethical. 🐞

---
