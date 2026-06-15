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


#### 2. Why isn't a 2-way handshake enough?

Example:
```text
Client → SYN
Server → ACK
```
Why do we need the third ACK from the client?


**ANSWER**

<h6>Step 1</h6>

```text
Client → SYN
```

Client says:
```text
I can send packets to you.
```
At this point the server knows:

✅ Client → Server path works

But it doesn't know:

❌ Server → Client path works

<h6>Step 2</h6>

```text
Server → SYN + ACK
```

Server says:
```text
I received your SYN.
Here is my own SYN.
```
Now the client knows:

✅ Server → Client path works

because it received the SYN-ACK.

But the server still doesn't know:

❌ Did the client receive my SYN-ACK?
❌ Is the Client ← Server path fully working?

<h6>Step 3</h6>

```text
Client → ACK
```

Client says:

```text
I received your SYN-ACK.
```

Now the server finally knows:

✅ Client → Server works
✅ Server → Client works
✅ Client received my response

Now both directions are verified.

#### 3. Why 2-Way Handshake can fail?

Imagine:
```text
Client → SYN
```
reaches server.

Server replies:
```text
ACK
```
But the ACK gets lost.

Client ❌ never receives ACK

Now:
```text
Client thinks:
Connection not established

Server thinks:
Connection established
```
The two sides disagree.

This is called a `half-open connection`.

TCP's third ACK prevents this ambiguity.



<h4>Production Analogy</h4>

Imagine calling someone.

Two-way version
```text
You: "Hello?"
Friend: "Hello."
```
Then the line cuts.

Did they hear your response?

Did you hear theirs?

Not clear.

---

Three-way version
```text
You: "Hello?"
Friend: "Hello, I hear you."
You: "Great, I hear you too."
```
Now both sides know communication works in both directions.


#### 4. QUICK CHECK
WHICH HAPPENS FIRST?

Suppose a browser requests:
```text
https://api.company.com/users
```

A)
```text
HTTP Request
↓
TCP Handshake
↓
TLS Handshake
```

B) 
```text
TCP Handshake
↓
TLS Handshake
↓
HTTP Request
```

And explain why that order is required. This question is asked surprisingly often in backend and system design interviews.



<h5>ANSWER</h5>

**COMPLETE FLOW**

When you open:
```text
https://api.company.com/users
```

The company roughly does:
```text
1. DNS Lookup
2. TCP Handshake
3. TLS Handshake
4. HTTP Request
5. HTTP Response
```


**Why TCP Must Come Before TLS**

TLS is not a transport protocol.

TLS doesn't move packets across the network.

TCP does.

Think:
```text
TLS needs a communication channel.
TCP provides that channel.
```


So:
```text
TCP Connection
      ↓
TLS Handshake
```
not
```text
TLS Handshake
      ↓
TCP Connection
```
because TLS has nowhere to send messages without TCP.

---

**Why TLS Must Come Before HTTP**

Suppose we skip TLS.

Browser immediately sends:
```http
GET /users
Authorization: Bearer xyz
```

Anyone between client and server could read:
```text
WiFi Router
ISP
Malicious Proxy
```

including:
```text
JWT Tokens
Passwords
Cookies
```

TLS establishes:
```text
Encryption Keys
```

first.

Only then can HTTP traffic be encrypted.

---

**What Actually Happens**

```text
Browser
   ↓
DNS
   ↓
52.10.20.30
   ↓
TCP Handshake
   ↓
Connection Established
   ↓
TLS Handshake
   ↓
Encryption Established
   ↓
GET /users
   ↓
HTTP Response
```

This is the mental model you should keep.


**Incident Scenario**

A senior engineer sees:

```text
Total API Latency = 900ms
```

They immediately ask:
```text
DNS = ?
TCP = ?
TLS = ?
Application = ?
Database = ?
```

Example:
```text
DNS = 50ms
TCP = 80ms
TLS = 200ms
Node = 100ms
Database = 470ms
```

Total:
```text
900ms
```

Now you know the database is the main bottleneck.

Without understanding the layers, many developers jump straight into Express code.

---

#### 5. Packets Sequencing Matters?

Suppose:
```text
Client
   ↓
TCP Connection
   ↓
Server
```

Client sends:
```text
Packet 1
Packet 2
Packet 3
```

Network delivers:
```text
Packet 2
Packet 3
Packet 1
```

**Questions:**
1. Will the application receive:
```text
2,3,1
```
or
```text
1,2,3
```

2. How do you think TCP is able to guarantee that order?


**ANSWERS**
QUESTION 1

If network delivers:
```text
Packet 2
Packet 3
Packet 1
```

Will your Node.js application receive:
```text
2
3
1
```
❌ No.

TCP guarantees:
```text
1
2
3
```

This is one of TCP's biggest promises:

> Ordered delivery.


**How Does TCP Do That?**

Every packet contains a sequence number.

Imagine:
```text
Packet 1 → Seq 1000
Packet 2 → Seq 2000
Packet 3 → Seq 3000
```

The receiver knows:
```text
1000 comes first
2000 comes second
3000 comes third
```

**What If Packet 1 Arrives Last?**

Network delivers:
```text
Packet 2
Packet 3
```

Receiver sees:
```text
Expected: Packet 1
Received: Packet 2
```

So it doesn't immediately hand data to your application.

Instead it waits.

```text
Buffer Packet 2
Buffer Packet 3
```

Then:
```text
Packet 1 arrives
```

Now:
```text
1
2
3
```
can be delivered to Node.js.

**What If Packet 1 Never Arrives?**

Receiver notices:

Missing Sequence Number

and requests retransmission.

Simplified:

```text
I got:
2
3

Where is:
1 ?
```

Sender resends Packet 1.

Only after it arrives can TCP complete the ordered stream.

---

**Why This Matters**

Suppose you're downloading:
```text
10 MB image
```

Without ordering guarantees:
```text
Middle arrives first
End arrives second
Beginning arrives last
```

The file becomes corrupted.

TCP prevents that.

---

