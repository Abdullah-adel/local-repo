Below is a **Google-style, interview-ready Networking & Web-Tech study guide for Linux**, written the same way we’ve been preparing your **TSE / Senior TSE** topics:
**simple → precise → observable → debuggable**.
For every topic you’ll see:

* **How it works (mental model)**
* **Linux commands to inspect**
* **What interviewers expect you to say**
* **Trap questions Google likes**
* **Real troubleshooting scenarios**

---

# 1️⃣ DNS in Linux (How it REALLY works)

## Mental model (simple & correct)

> DNS translates **names → IPs**, but Linux does **not** “just ask DNS”.

**Resolution order (glibc / systemd systems):**

1. `/etc/hosts`
2. Local DNS cache (systemd-resolved / nscd)
3. Configured DNS servers (`/etc/resolv.conf`)
4. Upstream DNS (recursive → authoritative)

➡ Controlled by:

```bash
/etc/nsswitch.conf
```

Example:

```text
hosts: files dns
```

---

## Key Linux components

* `/etc/resolv.conf` → DNS servers
* `systemd-resolved` → local stub resolver (127.0.0.53)
* `dig` / `nslookup` → raw DNS queries
* `getent hosts` → **actual libc resolution path**

---

## Commands interviewers EXPECT

```bash
cat /etc/resolv.conf
resolvectl status
getent hosts google.com
dig google.com
dig +trace google.com
tcpdump -i eth0 port 53
```

---

## Common interview questions (DNS)

### Q: Why does `ping google.com` fail but `dig google.com` works?

**Expected answer:**

> `dig` bypasses libc and queries DNS directly. `ping` uses libc → NSS → may be broken at `/etc/nsswitch.conf`, systemd-resolved, or `/etc/hosts`.

---

### Q: Why is DNS slow even though servers respond fast?

**Answer:**

* No local cache
* Broken IPv6 (AAAA timeout)
* Multiple search domains
* systemd-resolved fallback delays

---

### Q: What does `/etc/hosts` override?

**Answer:**

> Everything. It is checked before DNS.

---

## DNS troubleshooting flow (Google-style)

1. `getent hosts example.com`
2. `dig example.com`
3. `resolvectl status`
4. `tcpdump port 53`
5. Check latency & retries

---

# 2️⃣ TCP/IP Basics (What matters in interviews)

## TCP vs UDP (one line)

* **TCP** → reliable, ordered, congestion-controlled
* **UDP** → fast, no guarantees

---

## TCP 3-Way Handshake (Connection setup)

```
Client → SYN
Server → SYN-ACK
Client → ACK
```

**Purpose:**

* Agree on sequence numbers
* Ensure both sides are reachable

---

## TCP 4-Way Termination (Connection close)

```
FIN → ACK → FIN → ACK
```

**Why 4 steps?**

> TCP is full-duplex; each side closes independently.

---

## Interview traps

### Q: Why is there TIME_WAIT?

**Correct answer:**

> To prevent delayed packets from a previous connection being misinterpreted as a new one.

---

### Q: What causes SYN backlog overflow?

**Answer:**

* SYN flood
* Slow accept()
* Low kernel limits

Check:

```bash
ss -s
netstat -s
```

---

# 3️⃣ HTTP (What Google expects you to know)

## HTTP mental model

> HTTP is **stateless**, layered on TCP, extended by headers & cookies.

---

## HTTP Methods

* GET → retrieve
* POST → submit
* PUT → replace
* PATCH → modify
* DELETE → remove

---

## Status Codes (interview minimum)

* **200** OK
* **301/302** redirect
* **400** client error
* **401** unauthenticated
* **403** unauthorized
* **404** not found
* **500** server error
* **502** bad gateway
* **503** unavailable
* **504** timeout

---

## Headers (high-signal only)

* `Host` → virtual hosting
* `Authorization` → credentials
* `Set-Cookie` → server sets cookie
* `Cookie` → client sends cookie
* `Content-Type`
* `User-Agent`
* `X-Forwarded-For`

---

## Cookies & Auth Cookies

* Cookie = key/value stored client-side
* **Auth cookie** → session identifier
* Flags:

  * `HttpOnly`
  * `Secure`
  * `SameSite`

Example:

```http
Set-Cookie: sessionid=abc123; Secure; HttpOnly
```

---

## Authentication types

* Basic Auth
* Bearer tokens (OAuth/JWT)
* Session cookies
* mTLS

---

## Virtual Hosting (very common trap)

> Multiple domains on one IP → **Host header decides**

```http
Host: example.com
```

Without correct Host → wrong website served.

---

## Inspect HTTP traffic

```bash
curl -v https://example.com
curl -I https://example.com
tcpdump -i eth0 port 80 or port 443
```

---

# 4️⃣ Common Protocols (What to know + how to debug)

| Protocol  | Port | Debug            |
| --------- | ---- | ---------------- |
| DNS       | 53   | dig, tcpdump     |
| HTTP      | 80   | curl, tcpdump    |
| HTTPS     | 443  | curl -v, openssl |
| SSH       | 22   | ssh -vv          |
| FTP       | 21   | tcpdump          |
| SMTP      | 25   | telnet           |
| HTTPS TLS | 443  | openssl s_client |

Example:

```bash
openssl s_client -connect google.com:443
```

---

# 5️⃣ Client / Server Troubleshooting Scenarios

## Scenario 1: Client can’t connect

Checklist:

1. DNS resolution
2. Routing (`ip route`)
3. Port open (`ss -lntp`)
4. Firewall (`iptables`)
5. App listening

---

## Scenario 2: Works locally but not remotely

Likely:

* Listening on 127.0.0.1
* Firewall blocking
* Security group / ACL

---

## Scenario 3: Slow responses

Check:

* TCP retransmits
* DNS delay
* TLS handshake
* Backend latency

---

# 6️⃣ Authentication (Interview-level clarity)

## Key principle

> Authentication = identity
> Authorization = permissions

---

## Common failures & meaning

| Error         | Meaning                |
| ------------- | ---------------------- |
| 401           | Missing/invalid auth   |
| 403           | Auth OK, access denied |
| Login loop    | Cookie not set         |
| Random logout | Session expiration     |

---

## Inspect auth issues

```bash
curl -v -H "Authorization: Bearer TOKEN" https://api
browser devtools → cookies
```

---

# 7️⃣ Google-Style Interview Trap Questions

### “DNS works, but browser fails”

Expected answer:

* HTTPS cert mismatch
* SNI / Host header
* Proxy issue
* Cached DNS vs system DNS

---

### “HTTP works, HTTPS fails”

Answer:

* TLS version mismatch
* Expired cert
* Wrong SNI
* Firewall blocking 443

---

### “Service works sometimes”

Answer:

* Load balancer
* Sticky sessions
* DNS round-robin
* Cache inconsistency

---

# 8️⃣ One-line Google Interview Summary

> “I verify DNS resolution path, confirm TCP connectivity, inspect HTTP semantics, validate authentication, and correlate client-server behavior using observable Linux tools.”

---

Below is a **deep-clarification + interview-grade expansion** of every item you asked for, written exactly in the way **Google TSE / Senior TSE interviewers expect answers**:
**what it is → why it exists → what breaks → how you prove it on Linux**.

---

# 1️⃣ `getent hosts google.com` — what it is & why we use it

## What `getent` actually does

`getent` queries **glibc’s Name Service Switch (NSS)** — the *real* resolution path used by applications.

```bash
getent hosts google.com
```

This respects:

* `/etc/nsswitch.conf`
* `/etc/hosts`
* systemd-resolved / DNS
* LDAP / mDNS (if configured)

➡ **It answers the question:**

> “What IP would a real Linux program get?”

---

## Why interviewers LOVE `getent`

Because:

* `ping`, `curl`, `ssh`, browsers → all use **glibc**
* `dig` **does NOT**

So `getent` = **truth from the app’s perspective**

---

## Interview comparison (important)

| Command        | Uses NSS? | Use case             |
| -------------- | --------- | -------------------- |
| `getent hosts` | ✅ YES     | Real app behavior    |
| `ping`         | ✅ YES     | Real connectivity    |
| `dig`          | ❌ NO      | DNS server debugging |

---

## Google-style line

> “I use `getent` to validate the full libc resolution path, not just DNS.”

---

# 2️⃣ Why `ping google.com` fails but `dig google.com` works

## This is a **classic interview trap**

### Why it happens

`dig`:

* Sends raw DNS queries
* Ignores `/etc/nsswitch.conf`
* Ignores `/etc/hosts`
* Ignores systemd-resolved logic

`ping`:

* Uses libc
* Obeys NSS order
* May be blocked by:

  * bad `/etc/hosts`
  * broken IPv6
  * misconfigured systemd-resolved

---

## Real example

```bash
# dig works
dig google.com

# ping fails
ping google.com
```

Cause:

```text
/etc/hosts:
127.0.0.1 google.com
```

---

## Google expected explanation

> “`dig` proves DNS works. `ping` failing means the libc resolution path is broken.”

---

## How you prove it

```bash
getent hosts google.com
cat /etc/nsswitch.conf
resolvectl status
```

---

# 3️⃣ How `ss -s` shows SYN backlog overflow

## What SYN backlog is

When TCP receives `SYN`:

* Connection is **half-open**
* Stored in **SYN backlog**
* If backlog fills → new SYNs dropped

---

## Command

```bash
ss -s
```

Example output:

```text
TCP:
  1024 connections active
  300 SYN_RECV
```

---

## Red flags

* High `SYN_RECV`
* Clients report timeouts
* CPU not necessarily high

---

## Confirm with

```bash
netstat -s | grep -i syn
```

---

## Google-style fix proposal

> Increase backlog, enable SYN cookies, or fix slow accept() in the app.

---

# 4️⃣ “HTTP is stateless, layered on TCP, extended by headers & cookies”

## Break it down correctly

### Stateless

Each request is independent:

* Server remembers **nothing**
* No memory between requests

### Layered on TCP

HTTP:

* Relies on TCP for delivery
* Does not handle packet loss
* Connection ≠ session

### Extended by headers

Headers:

* Add metadata
* Control behavior
* Enable routing, auth, caching

### Cookies simulate state

Cookies:

* Client stores session ID
* Server maps ID → session data

---

## Interview sentence

> “HTTP itself is stateless; cookies and headers are how applications add state.”

---

# 5️⃣ HTTP headers — what matters (and WHY)

## Categories interviewers expect

### Routing

* `Host` → virtual hosting
* `X-Forwarded-For` → real client IP

### Auth

* `Authorization`
* `WWW-Authenticate`

### Content

* `Content-Type`
* `Content-Length`

### Security

* `Cookie`
* `Set-Cookie`
* `Origin`

---

## High-signal example

```http
Host: api.example.com
Authorization: Bearer TOKEN
Cookie: sessionid=abc
```

---

## Common header bug

Missing `Host`:

```bash
curl http://1.2.3.4
```

→ wrong website served

---

# 6️⃣ CORS (explained like Google wants)

## What CORS actually is

A **browser-enforced security policy**, NOT a server one.

---

## Problem CORS solves

> Prevents malicious websites from reading private API responses.

---

## How it works

Browser sends:

```http
Origin: https://evil.com
```

Server must reply:

```http
Access-Control-Allow-Origin: https://trusted.com
```

---

## Why curl works but browser fails

Because:

* curl ignores CORS
* browser enforces it

---

## Google interview line

> “CORS failures are browser-side policy errors, not server outages.”

---

# 7️⃣ HTTP Authentication types (interview-ready)

## Core types

### Basic Auth

```http
Authorization: Basic base64(user:pass)
```

* Simple
* Insecure without HTTPS

---

### Bearer Token (OAuth / JWT)

```http
Authorization: Bearer eyJhbGci...
```

* Stateless
* Scales well

---

### Session Cookies

```http
Set-Cookie: sessionid=abc
```

* Stateful
* Common in web apps

---

### mTLS

* Client cert authentication
* Used internally at Google

---

## Expected comparison

> “Tokens scale better than sessions; cookies simplify browsers.”

---

# 8️⃣ “Without correct Host → wrong website served”

## Why this happens

Multiple domains share:

* Same IP
* Same load balancer
* Same web server

Server chooses site by:

```http
Host: example.com
```

---

## Demo

```bash
curl http://IP
curl -H "Host: correct.com" http://IP
```

---

## Google line

> “Virtual hosting relies entirely on the Host header.”

---

# 9️⃣ `curl -I` vs `curl -v`

## `curl -I`

* Headers only
* Fast
* Health checks

```bash
curl -I https://example.com
```

---

## `curl -v`

* Full request/response
* TLS handshake
* Headers + flow

```bash
curl -v https://example.com
```

---

## Interview usage

* `-I` → availability
* `-v` → debugging

---

# 🔟 Inspecting authentication issues (deep)

## Cookie debugging

```bash
curl -v -c cookies.txt -b cookies.txt https://site
```

---

## Token debugging

```bash
curl -v -H "Authorization: Bearer TOKEN" https://api
```

---

## Browser tools

* Application → Cookies
* Network → Headers

---

## Common auth failures

| Symptom            | Cause                |
| ------------------ | -------------------- |
| Login loop         | Cookie not stored    |
| 401                | Missing/invalid auth |
| 403                | Permission denied    |
| Works in curl only | CORS                 |

---

# 11️⃣ 10 Full Mock Networking Interview Scenarios (Google style)

1. DNS works via `dig`, app fails
2. HTTPS fails, HTTP works
3. SYN flood causing timeouts
4. Wrong site served on same IP
5. Browser fails, curl works
6. Random 502 from load balancer
7. Login loop after success
8. High latency but low CPU
9. Service reachable locally only
10. mTLS handshake failures

(Each expects DNS → TCP → HTTP → Auth reasoning)

---

# 12️⃣ One-Page Networking Cheat Sheet

```
DNS: getent, dig, resolvectl
TCP: ss, netstat, tcpdump
HTTP: curl -I, curl -v
TLS: openssl s_client
Auth: headers, cookies
```

---

# 13️⃣ tcpdump → curl → strace correlation (REAL)

## Step 1: curl hangs

```bash
curl https://example.com
```

## Step 2: tcpdump

```bash
tcpdump -nn port 443
```

See SYN retransmits → network issue

---

## Step 3: strace

```bash
strace -e connect curl https://example.com
```

See `ETIMEDOUT`

---

## Google explanation

> “tcpdump shows packet loss; strace confirms the app is blocked on connect().”

---

## Final Google-style summary

> “I validate DNS via libc, confirm TCP health, inspect HTTP semantics, and debug authentication using observable Linux tooling.”

---

Here’s the **short, interview-ready explanation**, in very simple terms 👇

---

## 1️⃣ What is **NSS** actually used for? (in short)

**NSS = Name Service Switch**

👉 It tells Linux **where to look** when a program asks questions like:

* “What is the IP of this hostname?”
* “Does this user exist?”
* “What groups does this user belong to?”

It is configured in:

```
/etc/nsswitch.conf
```

Example:

```
hosts: files dns
```

Meaning:

1. Check `/etc/hosts`
2. If not found → ask DNS

**In one line (Google-style):**

> NSS controls the order and sources Linux uses to resolve names (hosts, users, groups).

---

## 2️⃣ What does it mean that **ping** uses glibc? (simple)

### What is **glibc**?

* The **standard C library** on Linux
* Provides common functions programs use (networking, files, users, DNS, etc.)

When a program wants to resolve a hostname, it **does NOT talk to DNS directly**.

Instead, it does this:

```
Program → glibc → NSS → DNS / files / other sources
```

---

## 3️⃣ How **ping** uses glibc + NSS

When you run:

```
ping google.com
```

What happens:

1. `ping` calls a glibc function like:

   ```
   getaddrinfo("google.com")
   ```
2. **glibc checks NSS rules** in `/etc/nsswitch.conf`
3. NSS decides:

   * `/etc/hosts`?
   * DNS?
4. glibc returns the IP to `ping`
5. `ping` sends ICMP packets to that IP

🔑 **ping does NOT implement DNS itself**

---

## 4️⃣ How **getent** uses glibc + NSS

When you run:

```
getent hosts google.com
```

What happens:

1. `getent` asks glibc:

   > “Resolve this name using NSS”
2. glibc follows `/etc/nsswitch.conf`
3. Output shows exactly what NSS returns

That’s why:

```
getent hosts google.com
```

✅ reflects **real system resolution**
❌ unlike `dig`, which bypasses NSS

---

## 5️⃣ Why **ping fails but dig works** (key interview trap)

| Command | Uses glibc + NSS | Talks to DNS directly |
| ------- | ---------------- | --------------------- |
| ping    | ✅ Yes            | ❌ No                  |
| getent  | ✅ Yes            | ❌ No                  |
| dig     | ❌ No             | ✅ Yes                 |

So:

* Broken `/etc/nsswitch.conf` → `ping` fails
* Broken `/etc/resolv.conf` → both fail
* DNS reachable but NSS misconfigured → `dig` works, `ping` fails

---

## 6️⃣ One-sentence **Google-level answer**

> NSS defines where name lookups come from; glibc implements it; tools like ping and getent rely on glibc, while dig bypasses NSS and queries DNS directly.

---

Here’s a **short, clean, interview-ready answer** 👇

---

## Most common networking protocols: **TCP vs UDP (and why)**

### 🟢 **TCP (reliable, ordered, connection-based)**

Used when **correctness matters more than speed**.

| Protocol         | Why TCP                                     |
| ---------------- | ------------------------------------------- |
| **HTTP / HTTPS** | Web pages must load completely and in order |
| **SSH**          | Commands must arrive exactly once           |
| **FTP**          | File integrity is critical                  |
| **SMTP**         | Emails must not be corrupted                |
| **IMAP / POP3**  | Mail consistency required                   |
| **HTTPS APIs**   | Requests/responses must be reliable         |

**Why TCP:**

* Guaranteed delivery
* Ordered packets
* Retransmissions on loss

---

### 🔵 **UDP (fast, connectionless, best-effort)**

Used when **speed matters more than perfection**.

| Protocol                     | Why UDP                         |
| ---------------------------- | ------------------------------- |
| **DNS**                      | Fast lookup, small queries      |
| **DHCP**                     | Broadcast discovery             |
| **NTP**                      | Time sync, tiny packets         |
| **VoIP**                     | Delay is worse than packet loss |
| **Video streaming / gaming** | Low latency preferred           |
| **SNMP**                     | Lightweight monitoring          |

**Why UDP:**

* No connection setup
* Lower latency
* Packet loss acceptable

---

## 🔑 Interview one-liner (Google-style)

> TCP is used when reliability and ordering matter; UDP is used when low latency and speed matter and occasional loss is acceptable.

---

## ⚠️ Common follow-up traps (know these)

* **DNS uses UDP by default**, but **TCP for large responses** (zone transfers, truncation)
* **HTTP/3 uses UDP** (via QUIC)
* **SNMP traps** use UDP, retries handled by app
* **Streaming apps** may add reliability on top of UDP

---

Below is a **one-page, interview-ready TCP vs UDP cheat sheet**, followed by **real `tcpdump` examples** exactly how Google expects you to reason about them.

---

# 🧠 TCP vs UDP — One-Page Cheat Sheet

| Feature            | **TCP**                               | **UDP**          |
| ------------------ | ------------------------------------- | ---------------- |
| Connection         | Connection-oriented (3-way handshake) | Connectionless   |
| Reliability        | Guaranteed delivery                   | Best effort      |
| Ordering           | In-order                              | No ordering      |
| Retransmission     | Yes                                   | No               |
| Flow control       | Yes (windowing)                       | No               |
| Congestion control | Yes                                   | No               |
| Latency            | Higher                                | Lower            |
| Overhead           | Higher                                | Minimal          |
| Use when           | Correctness matters                   | Speed matters    |
| Typical protocols  | HTTP(S), SSH, SMTP                    | DNS, DHCP, VoIP  |
| Failure symptom    | Retries, slowness                     | Packet loss      |
| App responsibility | Minimal                               | Must handle loss |

---

## 🟢 Common TCP protocols (why)

* **HTTP/HTTPS** → Page must load fully
* **SSH** → Commands must not be lost
* **SMTP / IMAP** → Mail integrity
* **FTP** → Exact file transfer

---

## 🔵 Common UDP protocols (why)

* **DNS** → Fast lookups
* **DHCP** → Broadcast discovery
* **NTP** → Tiny packets
* **VoIP / streaming** → Delay > loss

---

## 🔑 Interview one-liner

> TCP trades latency for reliability; UDP trades reliability for speed.

---

# 🔍 Real `tcpdump` Examples (TCP vs UDP)

---

## 1️⃣ TCP example — HTTPS traffic

Command:

```bash
tcpdump -n -i eth0 tcp port 443
```

Sample output:

```
IP 10.0.0.5.54321 > 142.250.74.14.443: Flags [S], seq 12345
IP 142.250.74.14.443 > 10.0.0.5.54321: Flags [S.], seq 67890, ack 12346
IP 10.0.0.5.54321 > 142.250.74.14.443: Flags [.], ack 67891
```

### How to interpret (important)

* `[S]` → SYN (start connection)
* `[S.]` → SYN-ACK
* `.` → ACK
  ✅ **3-way handshake → TCP confirmed**

---

## 2️⃣ TCP data transfer example

```
IP 10.0.0.5.54321 > 142.250.74.14.443: Flags [P.], seq 1:517, ack 1
```

* `[P.]` → Push data + ACK
* Sequence numbers present
  ✅ **Ordered, reliable data**

---

## 3️⃣ UDP example — DNS query

Command:

```bash
tcpdump -n -i eth0 udp port 53
```

Sample output:

```
IP 10.0.0.5.45678 > 8.8.8.8.53: 1234+ A? google.com.
IP 8.8.8.8.53 > 10.0.0.5.45678: 1234 1/0/0 A 142.250.74.14
```

### How to interpret

* No SYN / ACK
* Single request → single response
* Query ID (`1234`) matches reply
  ✅ **Fast, connectionless UDP**

---

## 4️⃣ UDP packet loss example

```
IP 10.0.0.5.45678 > 8.8.8.8.53: A? example.com.
(no response)
```

Meaning:

* UDP packet lost
* **Application must retry**
* Kernel does NOT retransmit

---

## 5️⃣ DNS switching to TCP (interview favorite)

```
IP 10.0.0.5.56789 > 8.8.8.8.53: Flags [S]
```

Why?

* DNS response too large
* Truncated UDP reply (`TC=1`)
* Client retries using TCP

---

# 🧪 How Google expects you to analyze tcpdump

| Observation        | Conclusion          |
| ------------------ | ------------------- |
| SYN / ACK present  | TCP                 |
| No handshake       | UDP                 |
| Retransmissions    | TCP congestion/loss |
| Silent packet loss | UDP                 |
| Slow but reliable  | TCP behavior        |
| Fast but lossy     | UDP behavior        |

---

## 🧠 Final interview script (perfect answer)

> “From tcpdump, I see a TCP handshake with retransmissions, so reliability is handled by the kernel. For UDP, there’s no handshake or retry, so any loss must be handled by the application.”

---

Below is a **clear, practical, Google-style explanation**, focused on **how each thing helps in troubleshooting** and **what kind of issues it exposes**. No fluff.

---

# 🔐 Auth / HTTP / CORS — What helps with what (and why)

## 1️⃣ **Auth** (Authentication)

**What it is:**
Who are you?

**How it helps in troubleshooting:**

* Confirms **identity verification** is happening
* Explains **401 Unauthorized** errors

**Typical issues:**

* Missing token
* Expired token
* Wrong auth method (Basic vs Bearer)

---

## 2️⃣ **Authorization**

**What it is:**
What are you allowed to do?

**How it helps:**

* Explains **403 Forbidden**
* Confirms role/permission issues

**Typical issues:**

* User authenticated but lacks permission
* Wrong role mapping

---

## 3️⃣ **WWW-Authenticate**

**What it is:**
Server telling client **how to authenticate**

**Why useful:**

* Shows **expected auth method**
* Explains why login fails

**Example:**

```
WWW-Authenticate: Bearer realm="api"
```

**Issue exposed:**
Client using wrong auth scheme.

---

## 4️⃣ **Content**

**What it is:**
The actual response body

**How it helps:**

* Confirms API returned expected data
* Helps spot **HTML error pages returned to APIs**

**Issue exposed:**

* Backend error masked as 200 OK

---

## 5️⃣ **Content-Type**

**What it is:**
What format the content is in

**How it helps:**

* Confirms client/server agree on data format

**Common issues:**

* `application/json` expected, `text/html` received
* Browser works, API client fails

---

## 6️⃣ **Content-Length**

**What it is:**
Size of response body

**How it helps:**

* Detects **truncated responses**
* Debugs **proxy/load balancer issues**

**Red flag:**
Browser hangs → wrong `Content-Length`.

---

## 7️⃣ **Security (Headers)**

Examples:

```
Strict-Transport-Security
X-Frame-Options
Content-Security-Policy
```

**How it helps:**

* Explains browser-only failures
* Explains blocked scripts / mixed content

**Issue exposed:**
Browser enforces rules curl ignores.

---

## 8️⃣ **Cookie**

**What it is:**
Client-stored state

**How it helps:**

* Confirms session persistence
* Debugs login issues

**Issue exposed:**
Cookie missing → user appears logged out.

---

## 9️⃣ **Set-Cookie**

**What it is:**
Server instructing browser to store cookie

**How it helps:**

* Confirms session creation
* Debugs login loops

**Common issues:**

* Missing `Secure`
* Wrong `Domain`
* Wrong `Path`

---

## 🔟 **Origin**

**What it is:**
Where the browser request came from

**How it helps:**

* Used in **CORS enforcement**
* Explains browser-only failures

---

# 🌐 How CORS actually works (simple + correct)

### 🔑 Key fact:

**CORS is enforced by the browser, NOT the server**

### What the server does:

Sends headers like:

```
Access-Control-Allow-Origin: https://trusted.com
```

### What the browser does:

* Compares request `Origin`
* Blocks response if origin not allowed

---

### ❌ Example:

```
Origin: https://evil.com
Access-Control-Allow-Origin: https://trusted.com
```

👉 Browser **blocks access to response**, even though:

* Server processed request
* Network request succeeded

---

### 🎯 What CORS solves

> Prevents malicious websites from reading private API responses using your browser credentials.

---

# 🧪 11️⃣ 10 Full Mock Networking Interview Scenarios (2 lines each)

### 1. **DNS works via dig, app fails**

DNS resolver bypassed; app uses NSS/system resolver → misconfigured `nsswitch.conf`.

---

### 2. **HTTPS fails, HTTP works**

TLS issue: cert, SNI, cipher mismatch, or time skew.

---

### 3. **SYN flood causing timeouts**

SYN backlog full → connections dropped before handshake completes.

---

### 4. **Wrong site served on same IP**

Missing or wrong `Host` header → virtual hosting issue.

---

### 5. **Browser fails, curl works**

Browser enforces CORS, cookies, CSP; curl does not.

---

### 6. **Random 502 from load balancer**

Backend intermittently failing or timing out; LB healthy checks incomplete.

---

### 7. **Login loop after success**

Cookie not stored/sent (SameSite, Secure, Domain mismatch).

---

### 8. **High latency but low CPU**

I/O wait, network latency, or lock contention—not compute bound.

---

### 9. **Service reachable locally only**

Bound to `127.0.0.1` instead of `0.0.0.0` or firewall blocking.

---

### 10. **mTLS handshake failures**

Client cert missing, expired, or CA mismatch.

---

# 🧭 How `resolvectl` helps with DNS troubleshooting

`resolvectl` shows **real system DNS state**, not just config files.

### What it tells you:

* Active DNS servers
* Per-interface DNS
* Search domains
* DNSSEC status

### Example:

```bash
resolvectl status
```

### Why important:

* Systemd-resolved may override `/etc/resolv.conf`
* Explains why `dig` works but apps fail

---

## 🎯 Google-style closing line

> HTTP headers explain *what happened*, DNS tools explain *where resolution failed*, and CORS explains *why browsers block what the network allows*.

---

Below is a **clear, correct, interview-level explanation**, split cleanly between **client side**, **server side**, then **time skew**.

---

# 🍪 How cookies are actually used (client vs server)

## 1️⃣ Client side (browser behavior)

**What the browser does:**

1. Receives a cookie from the server via:

   ```
   Set-Cookie: session_id=abc123; Secure; HttpOnly
   ```
2. **Stores the cookie** according to its attributes:

   * `Domain`
   * `Path`
   * `Expires / Max-Age`
   * `Secure`
   * `SameSite`
3. **Automatically sends the cookie back** on future requests:

   ```
   Cookie: session_id=abc123
   ```

   …only if:

   * Domain matches
   * Path matches
   * HTTPS if `Secure`
   * SameSite rules allow it

🔑 **Important:**
The browser decides **when to send cookies** — not the server.

---

## 2️⃣ Server side (what the server actually uses cookies for)

**What the server does:**

1. Sends a cookie to the client:

   ```
   Set-Cookie: session_id=abc123
   ```
2. Stores session state **server-side**, usually:

   * In memory (Redis)
   * In DB
   * In cache

Example:

```
session_id=abc123 → user_id=42, role=admin
```

3. On each request:

   * Reads `Cookie` header
   * Looks up session
   * Restores user context

🔑 **Critical point:**
Cookies usually store **an identifier**, not sensitive data.

---

## 3️⃣ Typical real-world cookie flow (login)

1. User logs in
2. Server validates credentials
3. Server responds:

   ```
   Set-Cookie: session_id=abc123; HttpOnly; Secure
   ```
4. Browser stores cookie
5. Browser sends cookie on every request
6. Server uses it to identify the user

---

## 4️⃣ Common cookie-related bugs (interview favorites)

| Symptom                      | Root cause                    |
| ---------------------------- | ----------------------------- |
| Login loop                   | Cookie not stored or not sent |
| Works in curl, not browser   | SameSite / Secure enforced    |
| Logged out randomly          | Cookie expiration             |
| Works locally, fails in prod | Domain mismatch               |
| HTTPS works, HTTP fails      | Secure cookie                 |

---

# ⏱️ What is **time skew**?

**Time skew = clock difference between systems**

Example:

* Client clock: **10:00**
* Server clock: **09:55**

Skew = **5 minutes**

---

## Why time skew breaks things

### 1️⃣ TLS / HTTPS

* Certificates have **valid from / valid to**
* If system clock is wrong → cert appears invalid

👉 Error:

```
certificate not yet valid
certificate expired
```

---

### 2️⃣ Authentication tokens (JWT, OAuth)

Tokens include:

```
iat (issued at)
exp (expiration)
```

If clocks differ:

* Token appears expired
* Token appears not yet valid

---

### 3️⃣ Cookies with expiration

```
Expires=Wed, 20 Dec 2025 12:00:00 GMT
```

Wrong clock →

* Cookie expires immediately
* Or never expires

---

## How time skew shows up in real incidents

| Symptom                | Likely cause      |
| ---------------------- | ----------------- |
| HTTPS suddenly fails   | Clock drift       |
| Auth randomly fails    | Token time checks |
| Works on one host only | NTP broken        |
| Reboot fixes issue     | Clock resync      |

---

## How to troubleshoot time skew (short)

```bash
date
timedatectl status
chronyc tracking
```

Look for:

* NTP synchronized = yes
* Large offset values

---

## Google-style closing answer (perfect)

> Cookies carry identity between requests while servers store the state; time skew breaks security because modern auth and TLS rely on accurate clocks.








Just tell me which one.
