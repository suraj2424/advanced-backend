# DNS
- "**Internet's Phonebook**"


*Humans Prefer*
```text
google.com
github.com
api.company.com
```

*Computers communicate using IP addresses*
```
142.250.193.78
140.82.121.3
104.18.32.47
```

**DNS Translates**
```text
google.com
      ↓
142.250.193.78
```
<u>Without DNS, every website would have to be visited using an IP address.</u>

## Why DNS Exists

Imagine you run a Backend API:
```text
api.mycompany.com
```

Your server IP address is:
```text
35.201.100.20
```

If the server moves to new machine:
```text
52.10.20.30
```

Users would have to remember the new IP.

Instead:
```text
api.mycompany.com
```
always stays the same.

Only the **DNS** record changes.

## What Happens When You Open a Website?

Let's say you type:
```text
https://github.com
```

The browser doesn't know GitHub's IP.

So it starts DNS resolution.

```text
Browser
   ↓
Need IP for github.com
   ↓
DNS Lookup
   ↓
140.x.x.x
   ↓
Connect to server
```

Only after obtaining the IP can the browser start TCP and HTTPS communication.

## DNS Resolution Flow (Simplified)
```text
Browser
   ↓
Local DNS Cache
   ↓
ISP Resolver
   ↓
DNS Servers
   ↓
IP Address Returned
```

Example:
```text
github.com
     ↓
140.82.121.3
```

Then browser connects to:
```text
140.82.121.3:443
```

## DNS Caching

- DNS lookups are expensive.

- Imagine Google receiving billions of lookups every second.

- Instead, results are cached.

```text
github.com
     ↓
140.82.121.3
```

Store for:
```text
TTL = 300 seconds
```

For the next 5 minutes:
```text
No DNS lookup needed.
```
Browser uses cached IP.

## TTL (Time To Live)

TTL defines how long a DNS result can be cached.

Example:
```text
github.com → 140.82.121.3

TTL = 300
```

Meaning:
```text
Cache for 300 seconds
```

After TTL expires:
```text
Perform DNS lookup again
```

## Common DNS Record Types

### A Record

Maps domain → IPv4 address

```text
api.company.com
      ↓
35.201.100.20
```
Most common record.

### AAAA Record

Maps domain → IPv6 address
```text
api.company.com
      ↓
2001:db8::1
```
Same idea as **A record** but IPv6.

### CNAME Record

Alias to another domain.

Example:
```text
www.company.com
      ↓
company.com
```
or
```text
cdn.company.com
      ↓
xyz.cloudflare.net
```
Useful because changing the target updates all aliases automatically.

#### ⚠️ One Golden Rule of CNAMEs
- A CNAME record cannot exist at the root domain. You can make a CNAME for a subdomain (like www.google.com or api.google.com), but you cannot make a CNAME for the root zone (google.com). The root domain must use an A record (or a special type of record like an ALIAS/ANAME depending on your DNS provider).

## Real World Backend Example

Suppose your API is deployed on AWS.

Users call:

```text
api.company.com
```

DNS
```text
api.company.com
      ↓
AWS Load Balancer
```

The load balancer then routes traffic:
```text
User
 ↓
DNS
 ↓
Load Balancer
 ↓
Server 1
Server 2
Server 3
```
This is how most production systems work.

## What Happens If DNS Fails?

If DNS cannot resolve:
```text
api.company.com
```
Browser never gets the IP.
No IP means:
```text
No TCP connection
No HTTPS
No Request
```
Website appears down even though servers may be perfectly healthy.

This is why DNS outages are extremely serious.

## Backend Engineer Perspective

When a user says:
> "The API is down"

A production engineer immediately asks:
1. Is DNS working?
2. Is the IP reachable?
3. Is TCP working?
4. Is TLS working?
5. Is the application working?

Many outages happen before the request even reaches your Node.js application.



# TCP

- **THREE WAY HANDSHAKE**

```text
Client → SYN
Server → SYN-ACK
Client → ACK
```
**MESSAGE OUTPUT**

TCP does not think in terms of messages.

TCP thinks in terms of a continuous byte stream.

For example:
```text
Hello
World
```

might be sent as:
```text
Hel
loWo
rld
```
or
```text
HelloWor
ld
```

TCP reassembles everything in order.

```text
Hello
World
```


## Why TCP Is Slower Than UDP

Imagine:
```text
Packet 1 lost
Packet 2 arrived
Packet 3 arrived
```

TCP says:
```text
Stop.
Need Packet 1.
```
This waiting adds latency.

--- 

But
UDP says:
```text
Cool.
Use whatever arrived.
```
No waiting.

That's why:
```text
Video Calls
Gaming
Live Streaming
```
Often use UDP


While:
```text
HTTP
PostgreSQL
Redis
SSH
```
use TCP.

Reliability > speed.

## The Most Important TCP Concept

A senior backend engineer thinks:
```text
TCP = Reliable
Ordered
Connection-oriented
```
Whenever you see:
```text
HTTP
HTTPS
PostgreSQL
Redis
SSH
```
you should automatically think:

**TCP** underneath.

