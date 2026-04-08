
Mar 22:
Introduction to System Design - https://studyalgorithms.com/system-design/introduction-system-design/#
Scaling Concepts
Client Server Model - https://studyalgorithms.com/system-design/client-server-model/
Load Balancer - https://studyalgorithms.com/system-design/load-balancer/

Mar 23:
Caching - https://studyalgorithms.com/system-design/caching-system-design/

Mar 25:
Database - https://studyalgorithms.com/system-design/databases-system-design/

Mar 29:

![[Screenshot 2026-03-29 at 7.27.04 PM.png]]

## Shared Storage
```
## 🧠 Problem Without Shared Storage

If you have **multiple data centers (DC1, DC2)**:

- Each DC has:
    - Its own app servers
    - Its own DB (master + replica)
- Data is **independent per DC**

### Issues:

1. **Data inconsistency**
    - User writes in DC1 → DC2 doesn’t see it immediately
2. **Failover problems**
    - If DC1 goes down → DC2 might have stale data
3. **Complex replication**
    - DB-to-DB cross-region replication is slow + tricky

---

## ✅ What “Shared Datasource” Means

A **shared datasource/storage** is a **central system accessible by all data centers**.

Examples:

- Distributed storage (S3, GCS, HDFS)
- Globally replicated DB (Spanner, DynamoDB, CockroachDB)
- Shared cache layer (Redis cluster across regions)

---

## 🔥 How It Helps

### 1. **Single Source of Truth**

All DCs read/write to the same system

DC1 ─┐  
     ├── Shared Storage  
DC2 ─┘

✔ No mismatch  
✔ No “which DB is correct?” problem

---

### 2. **Cross-DC Consistency**

- When DC1 updates data → DC2 sees it (almost instantly depending on system)
- Avoids:
    - replication lag
    - stale reads

---

### 3. **Easier Failover**

If one data center fails:

- Traffic shifts to another DC
- Data is still available (because it's shared)

👉 This is critical for **high availability (HA)**

---

### 4. **Reduced Replication Complexity**

Instead of:

DB1 → DB2 → DB3 (complex sync)

You get:

All → Shared system

✔ simpler architecture  
✔ fewer sync bugs

---

### 5. **Scalability Across Regions**

- Add a new DC → just connect to shared storage
- No need to setup replication pipelines again

---

## ⚠️ Trade-offs (Important)

Shared datasource is NOT magic:

### ❌ Latency

- Cross-region access can be slower

### ❌ Network dependency

- If shared system has issues → all DCs impacted

### ❌ Cost

- Global distributed systems are expensive

---

## 💡 Real-world Pattern

### Hybrid Approach (Most common)

- Local DB (fast reads/writes)
- Shared storage for:
    - metadata
    - files
    - global state

Example:

- Local DB → user sessions
- Shared storage → user profile / assets

---

## 🧩 Mapping to Your Diagram

From your image:

- Each DC still has:
    - Master + Slave DB (local performance)
- Shared storage is used for:
    - syncing
    - global data
    - consistency checks

---

## 🚀 Simple Analogy

Think of it like:

- Each DC = branch office
- Shared datasource = Google Drive

Without it:

- Everyone keeps local files → mismatch

With it:

- Everyone edits the same doc → consistent
```

Indexing in System Design: https://studyalgorithms.com/system-design/indexing-system-design/
Learn more about database indexing, look into finternet project which indexes that we are using

Apr 1:

Forward proxy and reverse proxy: - https://studyalgorithms.com/system-design/proxy-system-design/

```
Forward Proxy vs Reverse Proxy
Forward Proxy
A forward proxy sits between clients and the internet, acting on behalf of the clients.
How it works: The client sends requests to the proxy, which forwards them to the destination server. The server sees the proxy's IP, not the client's.
Use cases:

Anonymizing client identity (hide user IPs)
Bypassing geo-restrictions or firewalls
Content filtering in corporate networks
Caching to reduce bandwidth

Example: A company routes all employee internet traffic through a proxy to block certain websites.
Client → Forward Proxy → Internet → Server





Reverse Proxy
A reverse proxy sits between the internet and your servers, acting on behalf of the servers.
How it works: Clients send requests to the proxy, which routes them to the appropriate backend server. The client doesn't know which server actually handled the request.
Use cases:

Load balancing across multiple servers
SSL termination
Caching and compression
DDoS protection / hiding backend infrastructure
API gateways

Example: Nginx or Cloudflare sitting in front of your web servers, distributing traffic.
Client → Internet → Reverse Proxy → Server(s)

Key Difference
Forward ProxyReverse ProxyRepresentsThe clientThe serverHidesClient identityServer identityUsed byUsers / clientsServer operatorsDirectionOutbound trafficInbound trafficExamplesSquid, VPNsNginx, HAProxy, Cloudflare
```


Apr 4:

Message Queue - https://studyalgorithms.com/system-design/message-queue-system-design/


## CAP
```
  
what is partition tolerance

  

Partition tolerance is one of the three properties in the **CAP theorem** (Consistency, Availability, Partition tolerance) used in distributed systems.

### Simple definition

**Partition tolerance** means a system continues to operate even if there is a **network failure that splits the system into parts (partitions)** that cannot communicate with each other.

---

### What is a “partition”?

A partition happens when:

- Network links fail
    
- Nodes cannot talk to each other
    
- The system is split into isolated groups
    

Example:  
Imagine a distributed database with servers in two data centers. If the connection between them breaks, you now have two **partitions**.

---

### What partition tolerance guarantees

If a system is partition-tolerant:

- It **keeps running despite communication failures**
    
- Each partition can still process requests (depending on design)
    

---

### Trade-off (CAP theorem)

When a partition happens, a system must choose between:

- **Consistency (C):** all nodes see the same data
    
- **Availability (A):** every request gets a response
    

You **cannot guarantee both** during a partition.

So systems choose:

- **CP systems:** prioritize consistency, may reject requests
    
- **AP systems:** prioritize availability, may return stale data
    

---

### Quick analogy

Think of a team split into two rooms with no communication:

- Partition tolerance = both rooms keep working anyway
    
- But they might:
    
    - Stay consistent by stopping work until they reconnect (CP)
        
    - Or keep working independently but risk inconsistencies (AP)
        

---

If you want, I can give real-world examples (like MongoDB, Cassandra, etc.) to make it clearer.

  

why we cant have all 3

  

Short answer: **because of network failures (partitions), you’re forced to choose between consistency and availability—you can’t guarantee both at the same time.**

---

### First, recall the three (CAP theorem)

- **Consistency (C):** all nodes return the latest, same data
    
- **Availability (A):** every request gets a response
    
- **Partition Tolerance (P):** system keeps working despite network splits
    

---

### Why not all 3 together?

The problem starts when a **partition happens** (and in real systems, it _will_ happen).

Imagine this scenario:

- You have 2 nodes: **Node A** and **Node B**
    
- Network breaks → they can’t talk to each other (partition)
    
- A user writes data to Node A
    

Now another user reads from Node B.

---

### The conflict

At this point, Node B has two choices:

#### Option 1: Return old data

- System is **available** (it responds)
    
- But data is **inconsistent**
    

👉 You chose **AP (Availability + Partition tolerance)**

---

#### Option 2: Refuse the request

- System is **consistent** (no wrong data shown)
    
- But not **available** (request fails)
    

👉 You chose **CP (Consistency + Partition tolerance)**

---

### Why you can't have both C and A during partition

Because:

- Nodes **cannot coordinate** (network is broken)
    
- So they cannot ensure:
    
    - “latest data” (**Consistency**)
        
    - AND “always respond” (**Availability**)
        

At least one must be sacrificed.

---

### Important insight

- In real distributed systems, **Partition Tolerance is NOT optional**
    
- So the real choice is always:
    

👉 **Consistency vs Availability**

---

### Intuition (very practical)

Think of:

- **Consistency = correctness**
    
- **Availability = responsiveness**
    

During a network split:

- You can either **be correct**
    
- Or **be responsive**
    
- But not both, because communication is broken
    

---

### One-line takeaway

👉 **You can’t have all 3 because when the network breaks, you must choose between giving a correct answer or giving any answer at all.**

---

If you want, I can relate this to real systems like Cassandra vs PostgreSQL replication—those examples make it click instantly.

  

**Partition tolerance** means a system continues to operate even if there is a **network failure that splits the system into parts (partitions)** that cannot communicate with each other.

this is availbulity right?

  

Good catch — it _sounds_ like availability, but it’s **not the same thing**. This confusion is very common 👍

---

### 🔹 What you’re pointing at

> “System continues to operate even if network splits”

You’re thinking:  
👉 _“That means it’s available, right?”_

---

### 🔹 The subtle but important difference

#### ✅ **Partition Tolerance (P)**

- The system **survives network failures**
    
- It **keeps running as a whole system**
    
- It does **not guarantee every request succeeds**
    

👉 Focus: _system survives the split_

---

#### ✅ **Availability (A)**

- **Every request gets a response**
    
- Even if the response is wrong/stale
    

👉 Focus: _user always gets a reply_

---

### 🔥 Key distinction

During a partition:

|Situation|Partition Tolerance|Availability|
|---|---|---|
|System doesn’t crash|✅|❓|
|Every request gets response|❌ (not guaranteed)|✅|

---

### ⚡ Example to make it click

Network splits into A and B.

- If system is **partition tolerant**:
    
    - Both sides keep running
        
    - But some requests may fail ❗
        
- If system is **available**:
    
    - Every request gets a response
        
    - Even if data is outdated ❗
        

---

### 🧠 Why they feel similar

Because:

- Both involve “system still working”
    
- But:
    

👉 **Partition tolerance = system-level survival**  
👉 **Availability = request-level guarantee**

---

### 💡 One-line clarity

👉 Partition tolerance means _“system doesn’t break”_  
👉 Availability means _“user never gets an error”_

---

If you want, I can map this to real systems (like why Cassandra is AP and why some databases sacrifice availability) — that usually makes this distinction crystal clear.

  

give the proper notes for CAP theorem and examples explanation

  

Here are **clean, interview-ready notes** for the **CAP Theorem**, with clear explanations and real-world examples 👇

---

# 🧠 CAP Theorem – Proper Notes

### 📌 Definition

The **CAP theorem** states that:

> A distributed system can guarantee **at most 2 out of 3**:

- **Consistency (C)**
    
- **Availability (A)**
    
- **Partition Tolerance (P)**
    

---

# 🔹 The Three Properties

## 1. Consistency (C)

- Every read returns the **latest written value**
    
- All nodes see the **same data at the same time**
    

👉 Similar to a **single database view**

---

## 2. Availability (A)

- Every request gets a **response**
    
- Response can be **success or failure**, but not timeout
    

👉 System is always responsive

---

## 3. Partition Tolerance (P)

- System continues to work even if:
    
    - Network fails
        
    - Nodes cannot communicate
        

👉 System survives **network splits**

---

# ⚠️ Important Insight

👉 In real-world distributed systems:  
**Partition tolerance is mandatory**

Because:

- Networks **will fail**
    
- Distributed systems **must handle it**
    

---

# 🔥 The Real Trade-off

When a partition happens:

👉 You must choose between:

- **Consistency (C)**
    
- **Availability (A)**
    

---

# 📊 CAP Combinations

## 🟢 1. CP (Consistency + Partition Tolerance)

- Prioritizes **correct data**
    
- May reject requests (reduced availability)
    

### ✔️ Behavior:

- If data is uncertain → **fail the request**
    

### 📦 Examples:

- HBase
    
- MongoDB
    
- Zookeeper
    

### 🧠 Use cases:

- Banking systems
    
- Critical transactions
    
- Systems where **wrong data is unacceptable**
    

---

## 🟡 2. AP (Availability + Partition Tolerance)

- Prioritizes **always responding**
    
- May return **stale or inconsistent data**
    

### ✔️ Behavior:

- Always respond, even if data is outdated
    

### 📦 Examples:

- Cassandra
    
- DynamoDB
    
- CouchDB
    

### 🧠 Use cases:

- Social media feeds
    
- Product catalogs
    
- Systems where **availability > accuracy**
    

---

## 🔵 3. CA (Consistency + Availability)

- Works only when **no partitions exist**
    

### ✔️ Behavior:

- Always consistent and available
    
- But breaks during network failures
    

### 📦 Examples:

- Traditional relational databases:
    
    - MySQL
        
    - PostgreSQL
        

### 🧠 Use cases:

- Single-node systems
    
- Systems with **no network distribution**
    

---

# ⚡ Example Scenario (Very Important)

### Setup:

- Two nodes: A and B
    
- Network partition occurs
    

---

### Case 1: Choose Availability (AP)

- A updates data
    
- B doesn’t know about it
    
- User reads from B → gets **old data**
    

👉 ✔ Available  
👉 ❌ Not consistent

---

### Case 2: Choose Consistency (CP)

- A updates data
    
- B cannot verify latest state
    
- B **rejects read request**
    

👉 ✔ Consistent  
👉 ❌ Not available

---

# 🧠 Key Intuition

👉 During partition:

- Nodes **cannot communicate**
    
- So they cannot ensure:
    
    - “latest data” (Consistency)
        
    - AND “always respond” (Availability)
        

---

# 💡 One-line Summary

👉 **You can’t have C + A during partition because nodes cannot coordinate.**

---

# 🎯 Real-World Mapping

|System Type|CAP Choice|Why|
|---|---|---|
|Banking|CP|Accuracy is critical|
|Social media|AP|Availability matters more|
|Single DB|CA|No partitions|

---

# 🚀 Interview Tips

- Always say:  
    👉 “In real systems, P is mandatory, so we choose between C and A”
    
- Mention:
    
    - **CP → may fail requests**
        
    - **AP → may return stale data**
        

---

If you want, I can give a **diagram + memory trick** so you never forget CAP in interviews.

  

🔵 3. CA (Consistency + Availability) ------------------------------------- * Works only when **no partitions exist** ### ✔️ Behavior: * Always consistent and available * But breaks during network failures ### 📦 Examples: * Traditional relational databases: * MySQL * PostgreSQL ### 🧠 Use cases: * Single-node systems * Systems with **no network distribution**

you said CA is not possible?

  

Great question — this is where most people get confused, so let’s clear it properly 👇

---

# ❓ Is CA (Consistency + Availability) possible?

### 👉 Short answer:

**Yes — but only when there is NO partition.**  
And in real distributed systems, partitions are unavoidable.

---

# 🔹 Why people say “CA is not possible”

When we talk about the **CAP theorem**, we assume:

> 👉 The system is **distributed** and **network failures can happen**

In that context:

- Partition tolerance (**P**) becomes **mandatory**
    
- So the real choice becomes:  
    👉 **C vs A**
    

That’s why people say:

> ❌ “CA is not possible”  
> What they really mean is:  
> 👉 **“CA is not possible in the presence of partitions”**

---

# 🔹 When CA _is_ possible

CA works when:

- There is **no network partition**
    
- System is:
    
    - Single-node OR
        
    - Perfectly reliable network (theoretical)
        

### 📦 Examples:

- MySQL (single instance)
    
- PostgreSQL (single instance)
    

👉 In these:

- Data is always consistent
    
- System always responds
    

✔ Consistency  
✔ Availability  
❌ No partition handling needed

---

# 🔥 Why CA breaks in distributed systems

Once you distribute:

- Nodes can **lose connection**
    
- Now a partition happens
    

At that moment:

- If you try to keep **C + A**
    
- You hit an impossible situation:
    

👉 Nodes cannot:

- Agree on latest data (**Consistency**)
    
- AND respond to every request (**Availability**)
    

Because:  
👉 **They cannot communicate**

---

# 🧠 Clean mental model

### Think like this:

|Situation|Possible?|
|---|---|
|No partition|✅ CA possible|
|Partition happens|❌ CA impossible|

---

# 💡 Final clarity

👉 CAP theorem is really about **what happens during failure**

- Before failure → you can have CA
    
- During failure → you must choose CP or AP
    

---

# 🎯 One-line takeaway

👉 **CA is possible only in ideal conditions, but CAP theorem focuses on failure scenarios where CA breaks.**

---

If you want, I can give a **super simple diagram or story** that makes this 100% intuitive (the “two friends with phones” example is really good).

  

lets come from basics what is distributed systems, since I am beginner

  

Great, starting from basics is the right way 👍 — CAP makes much more sense once you understand distributed systems first.

---

# 🧠 What is a Distributed System?

### 📌 Definition

A **distributed system** is:

> A group of multiple computers (nodes) that work together and appear as a **single system** to the user.

---

# 🔹 Simple Example

Instead of:

- One powerful server handling everything ❌
    

You have:

- Many machines working together ✅
    

---

### 📦 Real-world examples:

- Google Search
    
- Amazon
    
- Netflix
    

When you use them:  
👉 You feel like it’s one system  
👉 But internally, **hundreds/thousands of servers** are involved

---

# 🔹 Why do we need Distributed Systems?

## 1. Scalability 📈

- Handle millions of users
    
- Add more machines instead of upgrading one
    

## 2. Fault Tolerance 🛠️

- If one machine fails → others take over
    

## 3. Performance ⚡

- Work is divided across machines → faster responses
    

---

# 🔹 Basic Components

A distributed system usually has:

- **Nodes (Servers)** → machines doing work
    
- **Network** → connects nodes
    
- **Communication** → APIs, RPC, messaging
    

---

# 🔥 Key Challenge (Very Important)

Unlike a single system:

👉 In distributed systems, **things can go wrong between machines**

---

### Problems you face:

## 1. Network failures 🌐

- Machines can’t talk to each other
    

## 2. Latency ⏱️

- Communication takes time
    

## 3. Partial failures ⚠️

- One node fails, others are fine
    

## 4. Data inconsistency 🔄

- Different nodes may have different data
    

---

# ⚡ Example to Understand

Imagine WhatsApp-like system:

- You send a message
    
- It goes to Server A
    
- Server B should also get it
    

But:

- Network breaks between A and B
    

👉 Now:

- A has new message
    
- B doesn’t
    

👉 System becomes **inconsistent**

---

# 🔗 This leads to CAP Theorem

Because of these problems, especially:

👉 **Network failures (partitions)**

We get the **CAP theorem**

---

# 🧠 Core Idea

In distributed systems:

- You cannot assume network is reliable
    
- Nodes may not communicate
    

So you must decide:

👉 Should system:

- Always be correct (Consistency)?
    
- Always respond (Availability)?
    

---

# 💡 Simple Analogy

Think of distributed system like:

👥 A team working in different rooms

- They communicate via phone (network)
    
- If phone breaks:
    
    - They may give wrong answers
        
    - Or stop answering
        

👉 That’s exactly the CAP problem

---

# 🎯 One-line Summary

👉 **Distributed system = multiple machines working together as one, but facing network-related challenges.**

---

If you want, next step I can:

- Build a **step-by-step bridge from distributed systems → CAP theorem (very intuitive)**
    
- Or explain with diagrams (super helpful for interviews)
    

  

Unlike a single system:

what is single system then

  

Got it — you’re asking what I meant by **“single system”** in contrast to distributed systems. Let’s make it very clear 👇

---

# 🧠 What is a Single System?

### 📌 Definition

A **single system (non-distributed system)** is:

> A system where **all computation and data live on one machine**

---

# 🔹 Simple View

Everything runs on:

- One server (or one computer)
    
- One memory space
    
- One database instance
    

👉 No coordination between multiple machines

---

# 🔹 Example

Think of:

- A local app running on your laptop
    
- A backend + database on one server
    

### 📦 Example tools:

- MySQL
    
- PostgreSQL
    

If installed on one machine:  
👉 That’s a **single system**

---

# 🔹 How it works

User → Server → Database (same machine)

Everything is:

- Local
    
- Fast
    
- No network dependency (internally)
    

---

# 🔥 Why it's simpler

In a single system:

## ✅ No network issues

- No communication between machines
    

## ✅ No data inconsistency

- Only one source of truth
    

## ✅ Easier to manage

- No synchronization needed
    

---

# ⚠️ Limitations

## ❌ Not scalable

- Can’t handle massive traffic easily
    

## ❌ Single point of failure

- If system crashes → everything down
    

## ❌ Limited performance

- Bound by one machine’s capacity
    

---

# 🔄 Compare with Distributed System

|Feature|Single System|Distributed System|
|---|---|---|
|Machines|1|Multiple|
|Data|One place|Multiple nodes|
|Network issues|❌|✅|
|Complexity|Low|High|
|Scalability|Limited|High|

---

# 🧠 Key Intuition

👉 Single system = **everything in one place**  
👉 Distributed system = **spread across many machines**

---

# 🔗 Why this matters for CAP

- In a **single system**:  
    👉 You can easily have **Consistency + Availability (CA)**
    
- In a **distributed system**:  
    👉 Network issues introduce **Partition tolerance (P)**  
    👉 Now you must choose **C vs A**
    

---

# 💡 One-line takeaway

👉 **Single system = no network, no partition problem → simple and consistent**  
👉 **Distributed system = network involved → complex trade-offs (CAP)**

---

If you want, next I can show:  
👉 how a single system evolves into a distributed one step-by-step (this makes CAP click instantly)
```


## Rate Limiter in System Design
```
A rate limiter controls how many requests a client can make to a server within a given time window. It protects services from abuse, DoS attacks, and ensures fair resource usage.

---

### Why Use a Rate Limiter?

- **Prevent abuse / DDoS** — stop bad actors from overwhelming your service
- **Fair usage** — ensure one client can't starve others
- **Cost control** — limit expensive downstream API calls
- **Reliability** — protect backend services from being overloaded

---

### Common Algorithms

**1. Token Bucket**

- A bucket holds tokens (capacity = max burst). Tokens refill at a fixed rate.
- Each request consumes one token. If the bucket is empty, the request is rejected.
- ✅ Allows bursting up to bucket capacity
- Used by: AWS, Stripe

**2. Leaky Bucket**

- Requests enter a queue (the "bucket") and are processed at a fixed output rate.
- If the queue is full, incoming requests are dropped.
- ✅ Smooths out bursty traffic, constant output rate
- Used by: Nginx

**3. Fixed Window Counter**

- Divide time into fixed windows (e.g., 1-minute slots). Count requests per window.
- If count exceeds limit, reject until the next window.
- ⚠️ Edge case: a burst at the boundary of two windows can double the allowed rate

**4. Sliding Window Log**

- Store a timestamp log of each request. On each request, evict timestamps older than the window and count the rest.
- ✅ Very accurate, no boundary problem
- ⚠️ Memory-intensive for high traffic

**5. Sliding Window Counter**

- Hybrid: combines fixed window counts with a weighted estimate for the overlap.
- ✅ Memory-efficient and approximately accurate
- Used by: Cloudflare

---

### Where to Place the Rate Limiter

Client → [Rate Limiter] → API Gateway → Backend Services

|Placement|Pros|Cons|
|---|---|---|
|Client-side|No network cost|Easily bypassed|
|API Gateway|Centralized, easy to manage|Single point of failure|
|Middleware|Per-service control|Distributed complexity|


### Distributed Rate Limiting Considerations

When you have **multiple servers**, each server can't maintain its own local counter — that would allow `N × limit` requests total. The solution is **centralized shared state**:

- **Redis** is the standard choice — atomic operations like `INCR` with `EXPIRE` ensure consistency
- Use **Lua scripts** on Redis for atomic read-modify-write to avoid race conditions
- **Sticky sessions** are an alternative (route same client to same server), but less robust

### Headers to Return

When rate limiting, always communicate state back to the client:

|Header|Meaning|
|---|---|
|`X-RateLimit-Limit`|Max requests allowed in window|
|`X-RateLimit-Remaining`|Requests left in current window|
|`X-RateLimit-Reset`|UTC timestamp when window resets|
|`Retry-After`|Seconds to wait (on 429 response)|

### Key Design Decisions

**What to key on?** You can rate limit by: IP address, user ID, API key, or endpoint. A good system does **all of them at different thresholds** (e.g. 10 req/s per IP, 100 req/s per user, 1000 req/s per API key).

**Hard vs. soft limits:** A hard limit rejects immediately. A soft limit allows a brief burst then throttles — better user experience.

**Where to store state?** Redis with `INCR` + `TTL` is the go-to for production. For serverless or edge deployments, Cloudflare's Durable Objects or similar edge KV stores work well.
```

## API Gateway
```
An API Gateway is a **single entry point** that sits between clients and your backend services. Think of it as a "front door" — all requests go through it, and it handles cross-cutting concerns so your individual services don't have to.

### What an API Gateway Does

Without a gateway, every client would need to know the address of every microservice — and every service would need to re-implement auth, rate limiting, logging, etc. The gateway centralizes all of that.

Here's what it handles:

**Authentication & Authorization** — Validates JWT tokens or API keys before the request even reaches a service. Services behind the gateway can trust that requests are already authenticated.

**Request Routing** — Maps incoming URLs to the right backend service. `GET /api/orders` goes to the Order service, `POST /api/payments` goes to the Payment service, and so on.

**Rate Limiting** — Enforces request limits per client (as we discussed earlier). The gateway is the perfect place to do this since it sees all traffic.

**SSL Termination** — Handles HTTPS encryption/decryption at the gateway level. Internal service-to-service communication can then happen over plain HTTP on a private network.

**Load Balancing** — Distributes requests across multiple instances of the same service.

**Logging & Monitoring** — One central place to log every request, track latency, and catch errors — rather than adding logging to every service.

**Request/Response Transformation** — Can modify headers, translate between protocols (e.g. REST to gRPC), or aggregate responses from multiple services into one.

---

### Popular API Gateway Solutions

|Tool|Best For|
|---|---|
|AWS API Gateway|AWS-native serverless/microservices|
|Kong|Self-hosted, highly customizable|
|Nginx|Lightweight, high performance|
|Traefik|Kubernetes / Docker environments|
|Apigee|Enterprise, Google Cloud|

---

### API Gateway vs Load Balancer

A common confusion — they're related but different:

- A **load balancer** distributes traffic across identical instances of the _same_ service (horizontal scaling).
- An **API gateway** routes to _different_ services based on the path/method, and adds cross-cutting logic like auth and rate limiting.

In practice, you often have both — the gateway routes to a service, and a load balancer behind it distributes across that service's instances.
```