# DNS

#### 1. Why can't browsers directly use domain names to connect to servers?
- because internal hardware and routing doesnt understand English words.
#### 2. What problem does DNS solve?
- its solve such that we can easily navigate to any server with the use of english words
#### 3. What is TTL?
- time to live for a dns cache such that browser will not lookup continously for ip address of a server for any domain name
#### 4. What is the difference between an A record and a CNAME record?
- A record maps IP address with a domain while CNAME record maps another domain which is useful for managing subdomains.
#### 5. If DNS fails but your Node.js server is healthy, can users access your website using the domain name?
- if dns fails, the browser did not find the ip address of server which implies even though server is healthy but i cant access server due to empty ip address
#### 6. In a production architecture, why might api.company.com point to a load balancer instead of directly to a Node.js server?
- for traffic distribution, high availability, health checks, zero down time deployments and horizontal scaling.

ex.
```text
User Requests
      ↓
Load Balancer
      ↓
Server 1
Server 2
Server 3
Server 4
```

when server 2 dies:
```text
Load Balancer
      ↓
Server 1
Server 3
Server 4
```
Users continue working.

This is a critical production concept.


#### 7. Suppose a backend architecture exists:
```text
api.company.com
      ↓
DNS
      ↓
Load Balancer
      ↓
Node Server A
      ↓
PostgreSQL
```
The user says;
> "The API is down"

List at least 5 places where the problem could be occurring before blaming the Node.js application.

1. Client Local Network or ISP (The Request never left)
- ISP outage
- Corporate firewall
- VPN issue
- Local DNS issue

2. DNS Resolution failures or latency
Possible issues:
```text
api.company.com
       ↓
DNS Server Slow
```
or
```text
NXDOMAIN
```
or
```text
DNS timeout
```
Application is perfectly healthy.

User still can't access it.

3. TCP Handshake / Network Routing Issues (ISP to Load Balancer)

Example:
```text
Client
   ↓
Internet
   ↓
Load Balancer
```

A routing problem somewhere in between can cause:
```text
SYN
   ↓
No Response
```

Result:
```text
Connection timeout
```
before HTTP even starts.

4. TLS/SSL Handshake Negotiation Slowness
- Before HTTP data can be sent, the client and the Load Balancer (or Reverse Proxy) must agree on encryption keys.

Examples:
1. Expired certificates
2. Slow TLS negotiation
3. Cipher mismatch
4. Reverse proxy overload

Request never reaches Node.js.

5. Load Balancer Capacity / Misconfigured Target Groups
- The Load Balancer itself is running out of CPU/memory resources, or its configuration is broken. More commonly, the Load Balancer thinks Node Server A is "unhealthy" because it failed a health check, so it has nowhere to send the traffic.
- load balancer until its own gateway timeout triggers (often returning `504 Gateway Timeout`)


Particularly this observation:

> Load balancer thinks Node is unhealthy

That's exactly the kind of thing that happens in production.

Example:
```text
Node Server
     ↓
Health Check Fails
     ↓
Removed From Pool
```

Then:
```text
Load Balancer
      ↓
No Healthy Targets
```

Result:
```text
503
504
Timeouts
```
6. Reverse Proxy

```text
Client
 ↓
Nginx
 ↓
Node
```

Possible issues:

1. Connection limits reached
2. Worker exhaustion
3. Bad configuration
4. Request buffering problems

7. Database

```text
Node
 ↓
Postgres
```

Node is healthy.

Database is not.

Example:
```text
Connection Pool Exhausted
```
Request waits forever.

User sees timeout.

8. Redis

```text
Node
 ↓
Redis
```
If every request depends on Redis:
```text
Redis Slow
```
Entire API becomes slow.

9. External Services
```text
Node
 ↓
Payment Gateway
```
or 
```text
Node
 ↓
Email Provider
```
or
```text
Node
 ↓
Auth Service
```
One dependency slows down.

Your API appears down.


#### The Production Thinking Model

When you hear:

"API is timing out"

Never start with:
```text
The Express route is broken
```
Start with:
```text
Where in the request path is time being spent?
```

```text
Client
 ↓
DNS
 ↓
TCP
 ↓
TLS
 ↓
Load Balancer
 ↓
Reverse Proxy
 ↓
Node
 ↓
Redis
 ↓
Database
 ↓
Third Party Services
```
Every arrow is a potential failure point.


#### 8. DNS Knowledge Check (Final)

Suppose:
```text
api.company.com
      ↓
52.10.20.30
TTL = 3600
```

At 10:00 AM you change DNS:
```text
api.company.com
      ↓
18.200.50.10
```
A user visited the site at 9:55 AM and still has the old record cached.

Question:

1. Until what time might that user continue using the old IP?
2. Why is TTL important during deployments and infrastructure migrations?
3. What problem can occur if TTL is set to 24 hours and you need to move traffic immediately?

Answers: 

1. until 10:55 AM the user will be using old IP.

2.  because it dictates how long your local networks and browsers cache your websites address, it acts as timer for how long the internet takes time to recognize your new servers.

Senior engineers often do:
```text
1 week before migration:
TTL = 86400 (24h)

1 day before migration:
TTL = 300 (5m)

Migration day:
Switch DNS

Wait

Increase TTL again
```
This is a very common production pattern.


3. downtime and routing errors, traffic will be splitted in old traffic and new traffic, old traffic will encounter service down.

```text
Some users
     ↓
Old IP

Other users
     ↓
New IP
```
This can create extremely confusing incidents.

Imagine:
```text
Old Server
     ↓
Already shut down
```
Users with cached records:
```text
api.company.com
      ↓
Old Dead Server
```
Result:
```text
Connection refused
Timeout
503
```
Meanwhile:
```text
New users
      ↓
New Server
```
Now support receives:

> "Website works for me but not for customers."

This is a **classic DNS propagation problem**.

### DNS MENTAL MODEL (REMEMBER THIS)
When a user says:

> "The site is down."

Ask:
```text
Can I resolve the domain?
```

When a user says:

> "It works for some people but not others."

Ask:
```text
Is this DNS caching or propagation?
```

When a user says:

> "We migrated servers and traffic is weird."

Ask:
```text
What is the TTL?
```