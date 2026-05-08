Here’s a short summary of the FastAPI + IIS deployment flow you completed:

---

# FastAPI on IIS Notes

## 1. Created FastAPI App

```python id="w9xt9f"
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def home():
    return {"hello": "world"}
```

Run locally:

```bash id="8vb5e4"
uvicorn main:app --host 127.0.0.1 --port 8000
```

---

# 2. Installed IIS

Enabled:

* IIS
* IIS Management Console

---

# 3. Installed IIS Extensions

Installed:

* [URL Rewrite](https://www.iis.net/downloads/microsoft/url-rewrite?utm_source=chatgpt.com)
* [Application Request Routing (ARR)](https://www.iis.net/downloads/microsoft/application-request-routing?utm_source=chatgpt.com)

Purpose:

* ARR = reverse proxy support
* URL Rewrite = route requests to FastAPI

---

# 4. Created IIS Website

In IIS:

```text id="rfmjlwm"
Sites → Add Website
```

Settings used:

```text id="2bqjj6"
Site Name: FastAPIApp
Port: 81
Physical Path: project folder
```

Port 81 was used because port 80 was already occupied by Default Web Site.

---

# 5. Enabled Proxy

At server level:

```text id="wt4g8j"
Application Request Routing Cache
→ Server Proxy Settings
→ Enable Proxy
```

---

# 6. Added Reverse Proxy Rule

Inside site:

```text id="1jhz1l"
FastAPIApp
→ URL Rewrite
→ Add Rule(s)
→ Reverse Proxy
```

Forwarded requests to:

```text id="kqm0u3"
127.0.0.1:8000
```

---

# 7. Final Architecture

```text id="6f3u5u"
Browser
   ↓
IIS (:81)
   ↓
Reverse Proxy
   ↓
Uvicorn (:8000)
   ↓
FastAPI
```

---

# 8. Final Result

Accessed app successfully via:

```text id="z0c5g9"
http://localhost:81
```

IIS now acts as the public-facing web server while Uvicorn runs the FastAPI application internally.


Because Internet Information Services does not natively understand how to serve an ASGI app like FastAPI.

FastAPI is actually being served by Uvicorn on port 8000.

IIS is just standing in front of it.

---

# ARR (Application Request Routing)

ARR gives IIS the ability to act as a:

```text id="g7sjq6"
Reverse Proxy
```

Meaning:

```text id="2dyzzg"
Client Request
    ↓
IIS
    ↓
Forward request to another server/app
```

Without ARR:

* IIS cannot forward requests to Uvicorn
* IIS only serves its own content

So ARR is the “proxy engine”.

---

# URL Rewrite

URL Rewrite creates the actual forwarding rules.

Example:

```text id="n30ecl"
Incoming:
http://localhost:81/

Rewrite to:
http://127.0.0.1:8000/
```

Without URL Rewrite:

* IIS would receive the request
* but wouldn’t know where to send it

So URL Rewrite is the “routing instruction”.

---

# Together

```text id="vwr0jo"
ARR        = ability to proxy
URL Rewrite = rules for where to proxy
```

---

# Real Flow

```text id="a1brsj"
Browser
   ↓
IIS receives request on :81
   ↓
URL Rewrite rule says:
"send this to 127.0.0.1:8000"
   ↓
ARR performs proxy forwarding
   ↓
Uvicorn receives request
   ↓
FastAPI executes route
   ↓
Response goes back through IIS
```

---

# Why Not Access Uvicorn Directly?

You technically could:

```text id="lgukmb"
http://127.0.0.1:8000
```

But IIS gives you:

* HTTPS management
* Windows integration
* reverse proxying
* logging
* hosting multiple sites
* domain bindings
* firewall friendliness
* enterprise compatibility

That’s why IIS sits in front.
