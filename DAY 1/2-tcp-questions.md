# TCP
#### 1. When a browser wants to send an HTTP request to a server, why can't it simply send:
```text
GET /users HTTP/1.1
```
immediately?


**ANSWER**
- before even sending a request both party must agree to each other sending request and response

*Imagine TCP Doesn't Exist*
Suppose your browser immediately sends:
```http
GET /users HTTP/1.1
Host: api.company.com
```

Questions arise:
1. Is the server even alive?
```text
Client
   ↓
Send Data
   ↓
???
```
Maybe:
- Server is down
- Server rebooting
- Wrong IP
- Network issue

How does the client know?

--- 

2. Is someone listening on that port?

Suppose:
```text
18.200.50.10:80
```
No process is listening.

If we immediately send data:

```text
GET /users
```
Who receives it?

Nobody.

--- 

3. Can both sides exchange data reliably?

The internet is messy.

Packets can:
```text
Be delayed
Be duplicated
Arrive out of order
Be dropped
```
TCP establishes rules before data transfer begins.

---

**What TCP Actually Provides**

Before HTTP starts, TCP guarantees:

*Connectivity*
```text
Can I reach you?
```
*Readiness*
```text
Are you accepting connections?
```

*Reliability*
```text
Will lost packets be retransmitted?
```

*Ordering*
```text
Will data arrive in the correct order?
```

For example:
```text
Packet 1
Packet 2
Packet 3
```

TCP ensures the application receives:
```text
Packet 1
Packet 2
Packet 3
```

even if the network delivered:
```text
Packet 2
Packet 3
Packet 1
```

**Real Backend Example**
When a browser calls:
```text
https://api.company.com/users
```

The sequence is:
```text
DNS
 ↓
IP Address Found
 ↓
TCP Connection Established
 ↓
TLS Handshake
 ↓
HTTP Request Sent
 ↓
HTTP Response Received
```

Notice:
```text
HTTP comes AFTER TCP
```

This is the fundamental interview question.