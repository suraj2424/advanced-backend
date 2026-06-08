Great. Since your goal is **production-grade backend engineering**, Day 1 should focus on understanding how real backend systems actually work in production rather than writing more Express routes.

# Day 1 — Backend Engineering Foundations & Request Lifecycle

## Objective

Understand what happens from the moment a user clicks a button until a response is returned from a production backend.

By the end of today, you should be able to explain:

* How a request reaches your server
* What DNS does
* How TCP works
* How TLS/HTTPS works
* What a reverse proxy does
* What happens inside Node.js when a request arrives
* Where latency comes from

---

# Part 1: Study (3-4 Hours)

## 1. DNS

Learn:

* Domain names
* DNS resolution
* Recursive resolver
* A record
* CNAME
* TTL

Example:

```text
api.company.com
        ↓
DNS Lookup
        ↓
35.221.10.55
        ↓
Request reaches server
```

Questions you should answer:

* Why do we need DNS?
* What happens if DNS fails?
* What is DNS caching?

---

## 2. TCP

Learn:

* TCP 3-way handshake

```text
Client → SYN
Server → SYN-ACK
Client → ACK
```

Learn:

* Connection establishment
* Reliability
* Retransmission
* Flow control

Questions:

* Why is TCP used for HTTP?
* What happens when packets are lost?

---

## 3. TLS / HTTPS

Learn:

* HTTP vs HTTPS
* SSL/TLS
* Certificates
* Public key cryptography

Understand:

```text
Browser
   ↓
TLS Handshake
   ↓
Encrypted Connection
   ↓
HTTP Requests
```

Questions:

* Why is HTTPS secure?
* What is a certificate authority?

---

## 4. HTTP Fundamentals

Learn:

### Methods

```text
GET
POST
PUT
PATCH
DELETE
```

### Status Codes

```text
200
201
400
401
403
404
500
502
503
```

### Headers

```text
Authorization
Content-Type
Cache-Control
Cookie
```

Questions:

* Difference between PUT and PATCH?
* Difference between 401 and 403?

---

## 5. Reverse Proxy

Learn:

### Nginx

What it does:

```text
Internet
    ↓
Nginx
    ↓
Node.js App
```

Responsibilities:

* SSL termination
* Load balancing
* Rate limiting
* Caching

Question:

Why don't companies expose Node.js directly to the internet?

---

# Part 2: Node.js Internals (2 Hours)

Study:

### Event Loop

Understand:

```text
Call Stack
Callback Queue
Microtask Queue
Event Loop
```

Learn:

* setTimeout
* Promise
* async/await
* process.nextTick

Watch execution order examples until you can predict outputs confidently.

---

## Understand Non-Blocking I/O

Question:

Why can Node handle thousands of connections without thousands of threads?

---

# Part 3: Practical Assignment (2-3 Hours)

Create:

```text
request-tracker/
```

Build an Express server.

Requirements:

### Middleware 1

Generate request ID.

```ts
req.id = uuid()
```

---

### Middleware 2

Measure latency.

```ts
Request Start
Request End
Response Time
```

---

### Middleware 3

Structured logging.

Log:

```json
{
  "requestId": "...",
  "method": "GET",
  "path": "/users",
  "statusCode": 200,
  "duration": 45
}
```

---

### Routes

```text
GET /health
GET /users
POST /users
```

Mock data is fine.

---

# Part 4: Production Thinking Exercise

For every request draw this diagram:

```text
Browser
 ↓
DNS
 ↓
TCP
 ↓
TLS
 ↓
Nginx
 ↓
Node.js
 ↓
Express Middleware
 ↓
Route Handler
 ↓
Database
 ↓
Response
```

Then answer:

1. Where can latency occur?
2. Where can failures occur?
3. Where can caching be added?
4. Where can monitoring be added?

---

# Deliverables

By the end of Day 1 you should have:

* Notes on DNS, TCP, TLS, HTTP
* Understanding of Event Loop
* Express request tracker project
* Ability to explain the complete request lifecycle from browser to database and back

This foundation is critical because every advanced topic later (load balancing, caching, microservices, distributed systems, Kubernetes, system design) builds on understanding this request flow. Day 2 will move into **Linux, networking tools, processes, ports, sockets, and how production servers actually run applications**.
