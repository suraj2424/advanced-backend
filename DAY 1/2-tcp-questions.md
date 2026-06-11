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