# 💻🔐 Threshold

> 🚦 **A lightweight IP-based access control system built with FastAPI**

Control who can access your FastAPI application by checking the client's IP address before allowing requests to reach your API endpoints.

The project uses Python's built-in `ipaddress` module to validate IP addresses and FastAPI middleware to create a simple security gate. 🛡️

---

## ✨ Features

* 🔐 **IP Allowlist** — Only approved IP ranges can access the application.
* ⚡ **Fast IP Checking** — IP networks are compiled once at startup.
* 🌐 **IPv4 & IPv6 Parsing** — Supports both IPv4 and IPv6 addresses.
* 🔄 **IPv4-Mapped IPv6 Support** — Handles addresses such as `::ffff:192.168.1.1`.
* 🚪 **Automatic Request Blocking** — Unauthorized clients receive HTTP `403`.
* 🔍 **Proxy-Aware IP Detection** — Supports common proxy/load-balancer headers.
* 🧪 **Pytest Tests** — Includes tests for valid, invalid, malformed, and edge-case IP addresses.
* 🧩 **Reusable Middleware** — The IP gate can be separated from the main application.

---

## 🧠 Why IP Access Control?

Imagine your API is a private office. 🏢

You don't want everyone on the internet walking through the door.

Instead, you create a security rule:

```text
🌍 Incoming Request
        │
        ▼
   🔎 Find Client IP
        │
        ▼
  🛡️ Check Allowlist
     /          \
   ✅ YES        ❌ NO
    │             │
    ▼             ▼
FastAPI App     HTTP 403
    │
    ▼
   🎉 Access
```

This project implements exactly that idea.

### ✅ Allowed IP

```text
Client IP
   ↓
192.168.1.100
   ↓
Matches 192.168.0.0/16
   ↓
✅ Request Allowed
```

### ❌ Blocked IP

```text
Client IP
   ↓
8.8.8.8
   ↓
Doesn't match allowlist
   ↓
❌ HTTP 403
```

---

# 📁 Project Structure

```text
IP_Access_Control/
│
├── 📄 main.py
│   └── FastAPI application
│
├── 🔐 ip_access_control.py
│   └── IP parsing, validation and allowlist logic
│
├── 🧩 midddleware.py
│   └── Reusable IP access-control middleware
│
├── 🧪 test_access_control.py
│   └── Pytest test cases
│
├── 📄 README.md
│   └── Project documentation
│
├── 📦 myenv/
│   └── Local Python virtual environment
│
└── 🗂️ __pycache__/
    └── Python generated cache files
```

> 💡 **Recommendation:** `myenv/` and `__pycache__/` normally should not be committed to Git. Add them to `.gitignore`.

---

# 🔍 How Each File Works

## 1️⃣ `ip_access_control.py` 🔐

This is the **brain of the project**.

It contains three important functions:

### `parse_ip()`

Converts a raw IP string into a Python IP address object.

It also handles:

* IPv4 addresses
* IPv6 addresses
* IPv4-mapped IPv6
* IPv4 addresses containing ports
* Bracketed IPv6 addresses

Example:

```python
parse_ip("192.168.1.10")
```

And:

```python
parse_ip("192.168.1.10:8080")
```

is converted to:

```text
192.168.1.10
```

The implementation also converts:

```text
::ffff:192.168.1.10
```

into its IPv4-mapped equivalent.

---

## 2️⃣ `is_allowed()` 🛡️

This function decides whether an IP address can enter the application.

Current allowlist:

```python
ALLOWED_CIDRS = [
    "10.0.0.0/8",
    "192.168.0.0/16",
    "127.0.0.1/32",
]
```

So the basic rule is:

```text
             IP Address
                  │
                  ▼
             Parse IP
                  │
             ┌────┴────┐
             │         │
          Valid?     Invalid
             │         │
            YES        ❌
             │
             ▼
       Check CIDR list
          /       \
        Match     No Match
         │           │
         ▼           ▼
        ✅           ❌
      Allow         Deny
```

Malformed addresses are rejected instead of being trusted.

---

## 3️⃣ `get_real_ip()` 🌍

This function attempts to determine the client's real IP address.

It checks headers such as:

```text
X-Forwarded-For
CF-Connecting-IP
True-Client-IP
X-Real-IP
```

If no supported header exists, it falls back to the direct connection address.

### ⚠️ Important Security Note

`X-Forwarded-For` should **only be trusted when the application is behind a proxy/load balancer that you control**.

A client can potentially send a fake forwarding header.

So the architecture should look like:

```text
👤 Client
   │
   ▼
🌐 Trusted Proxy / Load Balancer
   │
   ▼
🚦 FastAPI
   │
   ▼
🔐 IP Access Control
```

---

# 4️⃣ `main.py` 🚀

This is the main FastAPI application.

The middleware runs **before the endpoint**.

```text
HTTP Request
     │
     ▼
IP Middleware
     │
     ▼
Get Client IP
     │
     ▼
Check Allowlist
   /       \
  ❌        ✅
  │         │
403       Endpoint
            │
            ▼
         Response
```

The current application returns:

```json
{
  "message": "You are allowed in!"
}
```

when the IP is permitted.

Unauthorized requests receive:

```json
{
  "error": "Access denied",
  "ip": "CLIENT_IP"
}
```

with HTTP status:

```text
403 Forbidden
```

---

# 5️⃣ `midddleware.py` 🧩

This file contains a reusable `ip_gate()` middleware function.

It performs the same basic process:

```text
Request
   ↓
Get Real IP
   ↓
Check IP
   ↓
Allowed? ─── No ──→ 403
   │
  Yes
   ↓
Continue Request
```

This is useful if you want to keep your security logic separate from `main.py`.

> 📝 Small cleanup suggestion: rename `midddleware.py` to `middleware.py` because the current filename contains an extra `d`.

---

# 6️⃣ `test_access_control.py` 🧪

The test file uses **pytest**.

It tests:

* ✅ Allowed IP addresses
* ❌ Denied IP addresses
* 🚫 Malformed IP addresses
* 🌐 IPv4-mapped IPv6
* 🔌 IP addresses containing ports
* 📍 CIDR boundary cases

For example:

```python
assert is_allowed("192.168.1.1:8080") is True
```

and:

```python
assert is_allowed("not_an_ip") is False
```

The test suite is designed to verify both normal behavior and edge cases.

---

# ⚙️ Installation

## 1. Clone the repository

```bash
git clone https://github.com/sreeja7748/IP_Access_Control.git
cd IP_Access_Control
```

## 2. Create a virtual environment

### Windows

```bash
python -m venv myenv
myenv\Scripts\activate
```

### Linux / macOS

```bash
python3 -m venv myenv
source myenv/bin/activate
```

---

## 3. Install dependencies

```bash
pip install fastapi uvicorn pytest
```

---

# 🚀 Run the Application

Start the FastAPI server:

```bash
uvicorn main:app --reload
```

The application will normally be available at:

```text
http://127.0.0.1:8000
```

Open the endpoint:

```text
/
```

If your IP is allowed, you'll receive:

```json
{
  "message": "You are allowed in!"
}
```

If your IP isn't allowed:

```json
{
  "error": "Access denied",
  "ip": "YOUR_IP"
}
```

with:

```text
HTTP 403 Forbidden
```

---

# 🧪 Run Tests

Run:

```bash
pytest
```

For more detailed output:

```bash
pytest -v
```

---

# 🔧 How to Add Your Own IP Range

Open:

```text
ip_access_control.py
```

Find:

```python
ALLOWED_CIDRS = [
    "10.0.0.0/8",
    "192.168.0.0/16",
    "127.0.0.1/32",
]
```

You can add your own CIDR ranges.

For example:

```python
ALLOWED_CIDRS = [
    "10.0.0.0/8",
    "192.168.0.0/16",
    "127.0.0.1/32",
    "172.16.0.0/12",
]
```

Now addresses inside:

```text
172.16.0.0 → 172.31.255.255
```

can be permitted.

---

# 🧩 Example

Suppose the incoming IP is:

```text
192.168.1.50
```

The application performs:

```text
192.168.1.50
      │
      ▼
parse_ip()
      │
      ▼
IPv4Address
      │
      ▼
is_allowed()
      │
      ▼
192.168.0.0/16
      │
      ▼
     MATCH ✅
      │
      ▼
FastAPI Endpoint
```

Result:

```json
{
  "message": "You are allowed in!"
}
```

---

# ❌ Blocked Example

Suppose the client IP is:

```text
8.8.8.8
```

Then:

```text
8.8.8.8
   │
   ▼
parse_ip()
   │
   ▼
is_allowed()
   │
   ▼
No matching CIDR ❌
   │
   ▼
HTTP 403
```

Response:

```json
{
  "error": "Access denied",
  "ip": "8.8.8.8"
}
```

---

# 🏗️ Architecture

```text
                    🌍 INTERNET
                         │
                         ▼
                  📡 HTTP Request
                         │
                         ▼
                ┌─────────────────┐
                │    FastAPI      │
                │    Middleware   │
                └────────┬────────┘
                         │
                         ▼
                  🔎 Get Real IP
                         │
                         ▼
                ┌─────────────────┐
                │  parse_ip()     │
                └────────┬────────┘
                         │
                         ▼
                ┌─────────────────┐
                │  is_allowed()   │
                └────────┬────────┘
                         │
                   ┌─────┴─────┐
                   │           │
                 ❌ DENY      ✅ ALLOW
                   │           │
                   ▼           ▼
               HTTP 403    FastAPI Route
                               │
                               ▼
                            🎉 Response
```

---

# 💡 Why This Project Is Useful

### 🔐 Security

Restrict sensitive APIs to trusted IP ranges.

### 🏢 Internal Applications

Useful for:

* Company dashboards
* Internal tools
* Admin APIs
* Development environments
* Private services

### ⚡ Lightweight

The project doesn't require a database for basic IP allowlisting.

### 🧩 Reusable

The access-control logic is separated into reusable functions.

### 🧪 Testable

IP validation can be tested independently from the web application.

---


# 🛠️ Recommended Improvements

For a stronger production-ready version, consider adding:

* 📄 `requirements.txt`
* 🚫 `.gitignore`
* ⚙️ Environment-based configuration
* 🧪 More middleware integration tests
* 📝 Better logging
* 🔒 Trusted-proxy configuration
* 🌐 Explicit IPv6 CIDR configuration
* 📚 API documentation
* 🐳 Docker support
* 🔄 CI/CD with GitHub Actions

Recommended structure:

```text
IP_Access_Control/
│
├── 📁 app/
│   ├── __init__.py
│   ├── main.py
│   ├── ip_access_control.py
│   └── middleware.py
│
├── 📁 tests/
│   └── test_access_control.py
│
├── 📄 README.md
├── 📄 requirements.txt
├── 📄 .gitignore
└── 🐳 Dockerfile
```

---

# 🎯 Project Flow — Super Simple

```text
👤 User
   ↓
🌐 Sends Request
   ↓
🔎 Application Finds IP
   ↓
🧠 IP Parser Validates It
   ↓
📋 Compare With Allowlist
   ↓
 ┌───────────────┐
 │               │
 ▼               ▼
✅ Allowed      ❌ Not Allowed
 │               │
 ▼               ▼
🚀 API          🚫 403
Response        Forbidden
```

---

# 🏁 Conclusion

**IP Access Control** is a simple FastAPI security layer that demonstrates how IP-based allowlisting can be implemented using Python and middleware.

The core idea is beautifully simple:

> 🔐 **Identify the client → Validate the IP → Allow or deny the request.**

It is a good foundation for learning:

```text
🐍 Python
   +
⚡ FastAPI
   +
🌐 Networking
   +
🔐 Access Control
   +
🧪 Testing
```

⭐ **Simple. Lightweight. Easy to understand. Easy to extend.**

---

## 👩‍💻 Author

**Sreeja Dey**

### 🔗 Repository

https://github.com/sreeja7748/IP_Access_Control

https://github.com/Suchitra-Santra/IP_Access_Control

---

## ⭐ Support

If you found this project helpful, please consider giving it a ⭐ on GitHub.

---

