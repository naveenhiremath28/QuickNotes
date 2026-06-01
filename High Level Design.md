
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
================================================================
   CAP THEOREM — DISTRIBUTED SYSTEMS (Beginner to Interview Ready)
================================================================

----------------------------------------------------------------
1. WHAT IS A SINGLE SYSTEM?
----------------------------------------------------------------
A system where ALL computation and data live on ONE machine.

  User → Server → Database (all on same machine)

  Pros:
    - No network issues (everything is local)
    - No data inconsistency (only one source of truth)
    - Simple to manage — no synchronization needed

  Cons:
    - Not scalable (bound by one machine's capacity)
    - Single point of failure (crashes = everything down)
    - Limited performance

----------------------------------------------------------------
2. WHAT IS A DISTRIBUTED SYSTEM?
----------------------------------------------------------------
A group of multiple computers (nodes) that work together and
appear as a SINGLE system to the user.

  Instead of:   One powerful server handling everything
  You have:     Many machines working together

Real-world examples:
  - Google Search, Amazon, Netflix
  → You feel like it's one system
  → But internally, hundreds/thousands of servers are involved

WHY do we need distributed systems?
  - Scalability    : Handle millions of users, add more machines
  - Fault Tolerance: If one machine fails → others take over
  - Performance    : Work divided across machines → faster

COMPARISON:
  ┌─────────────────┬───────────────────┬──────────────────────┐
  │ Feature         │ Single System     │ Distributed System   │
  ├─────────────────┼───────────────────┼──────────────────────┤
  │ Machines        │ 1                 │ Multiple             │
  │ Data            │ One place         │ Multiple nodes       │
  │ Network issues  │ No                │ Yes                  │
  │ Complexity      │ Low               │ High                 │
  │ Scalability     │ Limited           │ High                 │
  └─────────────────┴───────────────────┴──────────────────────┘

KEY CHALLENGES in distributed systems:
  - Network failures   : Machines can't talk to each other
  - Latency            : Communication takes time
  - Partial failures   : One node fails, others are fine
  - Data inconsistency : Different nodes may have different data

EXAMPLE:
  You send a WhatsApp message → goes to Server A
  Server B should also get it
  But network breaks between A and B
  → A has new message, B doesn't → system is INCONSISTENT

This problem of network failure leads us to the CAP Theorem.

----------------------------------------------------------------
3. WHAT IS A NETWORK PARTITION?
----------------------------------------------------------------
A partition happens when:
  - Network links fail between nodes
  - Nodes cannot talk to each other
  - System is split into isolated groups

Example:
  Data center 1 ── (connection breaks) ── Data center 2
  Now you have two partitions that can't communicate.

----------------------------------------------------------------
4. CAP THEOREM
----------------------------------------------------------------
A distributed system can guarantee AT MOST 2 out of 3:

  C — Consistency
  A — Availability
  P — Partition Tolerance

THE THREE PROPERTIES:

  Consistency (C):
    - Every read returns the LATEST written value
    - All nodes see the SAME data at the same time
    - Like a single database view — no stale data ever

  Availability (A):
    - Every request gets a RESPONSE
    - Response can be success or failure, but NOT a timeout
    - System is always responsive

  Partition Tolerance (P):
    - System continues to work even if network fails
    - Nodes may lose connection — system keeps running
    - Focus: system-level survival (not request-level guarantee)

IMPORTANT DISTINCTION — P vs A:
  ┌──────────────────────────────┬─────────────────────────────┐
  │ Partition Tolerance          │ Availability                │
  ├──────────────────────────────┼─────────────────────────────┤
  │ System doesn't crash         │ Every request gets response │
  │ System survives the split    │ User always gets a reply    │
  │ Some requests may still fail │ Even if data is stale       │
  └──────────────────────────────┴─────────────────────────────┘

  P = system-level survival
  A = request-level guarantee

----------------------------------------------------------------
5. WHY CAN'T WE HAVE ALL 3?
----------------------------------------------------------------
Because in real distributed systems, PARTITION TOLERANCE IS
MANDATORY. Networks WILL fail. You cannot opt out of P.

So the REAL choice is always:
  → Consistency (C) vs Availability (A)

WHEN A PARTITION HAPPENS — the conflict:

  Setup: Two nodes A and B. Network breaks. User writes to A.
         Another user reads from Node B.

  Node B has two choices:

  Option 1 — Return old data:
    → System is AVAILABLE (it responds)
    → But data is INCONSISTENT (stale)
    → This is AP (Availability + Partition Tolerance)

  Option 2 — Refuse the request:
    → System is CONSISTENT (no wrong data shown)
    → But NOT AVAILABLE (request fails)
    → This is CP (Consistency + Partition Tolerance)

  Nodes CANNOT coordinate (network is broken)
  So they cannot ensure "latest data" AND "always respond"
  At least one must be sacrificed.

ONE-LINE TAKEAWAY:
  You can't have all 3 because when the network breaks,
  you must choose between giving a CORRECT answer
  or giving ANY answer at all.

----------------------------------------------------------------
6. THE THREE CAP COMBINATIONS
----------------------------------------------------------------

CP — Consistency + Partition Tolerance
  - Prioritizes CORRECT data
  - May reject requests during partition (reduced availability)
  - If data is uncertain → FAIL the request

  Examples  : HBase, MongoDB, Zookeeper
  Use cases : Banking systems, critical transactions,
              systems where wrong data is unacceptable

AP — Availability + Partition Tolerance
  - Prioritizes ALWAYS RESPONDING
  - May return stale or inconsistent data during partition
  - Always respond, even if data is outdated

  Examples  : Cassandra, DynamoDB, CouchDB
  Use cases : Social media feeds, product catalogs,
              systems where availability > accuracy

CA — Consistency + Availability
  - Works ONLY when there is NO partition
  - Always consistent and available in normal conditions
  - But BREAKS during network failures

  Examples  : MySQL, PostgreSQL (single instance)
  Use cases : Single-node systems, no network distribution

  NOTE: CA is NOT possible in distributed systems.
        The moment a partition happens, CA is impossible.
        CA only exists in single-node / ideal conditions.
        CAP theorem focuses on failure scenarios → CA breaks.

  ┌──────────────┬────────────┬──────────────────────────────┐
  │ System Type  │ CAP Choice │ Why                          │
  ├──────────────┼────────────┼──────────────────────────────┤
  │ Banking      │ CP         │ Accuracy is critical         │
  │ Social media │ AP         │ Availability matters more    │
  │ Single DB    │ CA         │ No partitions to worry about │
  └──────────────┴────────────┴──────────────────────────────┘

----------------------------------------------------------------
7. EXAMPLE SCENARIO (Interview-Ready)
----------------------------------------------------------------

Setup:
  Node A ── (network fails) ── Node B

Case 1: Choose Availability (AP)
  - A updates data
  - B doesn't know about it
  - User reads from B → gets OLD data
  → Available ✓   |   Consistent ✗

Case 2: Choose Consistency (CP)
  - A updates data
  - B cannot verify latest state
  - B REJECTS the read request
  → Consistent ✓   |   Available ✗

----------------------------------------------------------------
8. KEY INTUITION SUMMARY
----------------------------------------------------------------

  Partition tolerance = "system doesn't break"
  Availability        = "user never gets an error"
  Consistency         = "user always gets correct data"

  Before partition → CA is possible
  During partition → must choose CP or AP

  In distributed systems → P is NOT optional
  So real trade-off is always → C vs A
================================================================

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


UniquId: https://www.youtube.com/watch?v=ogOKDhOa_cs&list=PLFdAYMIVJQHOWJgRrjv_RH-ng95B2h3ON&index=12


Apr-13
## Notification System

```

================================================================
   SYSTEM DESIGN: NOTIFICATION SYSTEM (SMS, EMAIL & PUSH)
================================================================

----------------------------------------------------------------
1. WHAT IS A NOTIFICATION SYSTEM?
----------------------------------------------------------------
A system that delivers messages/alerts to users across different
channels — push, SMS, and email.

Types of Notifications:
  - Push Notifications : Sent to mobile devices (iOS / Android)
                         Real-time alerts (reminders, updates)
  - SMS                : Text messages via telecom providers
                         Used for OTPs, alerts, marketing
  - Email              : Rich content (images, banners, links)
                         Used for promotions, newsletters

----------------------------------------------------------------
2. BASIC HIGH-LEVEL DESIGN
----------------------------------------------------------------
Your system does NOT deliver notifications directly.
It delegates delivery to external third-party providers.

FLOW:
  Service (Producer) → Notification System → Third-party → User Device

Third-party Providers by channel:
  - Email : SMTP servers, SendGrid, Amazon SES
  - SMS   : Telecom APIs
  - Push  : APNS (Apple), FCM (Firebase/Android)

PROBLEM with this basic design:
  - Tight coupling between services
  - No buffering — spikes can crash the system
  - No retry handling on failure
  - Not scalable

----------------------------------------------------------------
3. DATA GATHERING & STORAGE
----------------------------------------------------------------
To send notifications, you need user contact info and
device tokens — collected and stored during user activity.

DATA FLOW:
  User → Load Balancer → API Servers → Database

Data Models:

  User Table:
  ┌───────────┬────────────────┬───────┬─────────┬────────────┐
  │  user_id  │     email      │ phone │ country │ created_at │
  └───────────┴────────────────┴───────┴─────────┴────────────┘

  Device Table:
  ┌──────────────┬─────────┬───────────┐
  │ device_token │ user_id │ last_used │
  └──────────────┴─────────┴───────────┘

Important Notes:
  - Device tokens are dynamic — must be updated frequently
  - One user can have multiple devices
  - Store mapping: user ↔ multiple device tokens

----------------------------------------------------------------
4. IMPROVED HIGH-LEVEL ARCHITECTURE
----------------------------------------------------------------
Multiple services (orders, payments, reminders) all produce
notification events. A central Notification System handles
routing, formatting, and sending.

FLOW:
  Multiple Services → Notification System → Third-party → Devices

Components:
  1. Producer Services   : Any service generating events
                           (Orders, Payments, Reminders)
  2. Notification System : Core — handles routing, formatting,
                           sending to right channel
  3. Third-party Services: Email / SMS / Push providers
  4. Clients             : Mobile devices, Web apps

----------------------------------------------------------------
5. IMPROVED DESIGN (With Queue + Workers + Retries)
----------------------------------------------------------------
Add separate queues per channel between the Notification Service
and delivery workers to decouple, buffer, and enable retries.

----------------------------------------------------------------
IMPROVED DESIGN FLOW
----------------------------------------------------------------

  Service 1 ──┐
  Service 2 ──┤
  Service N ──┘
              │
              ▼
   ┌──────────────────────────┐
   │    Notification Service  │
   │   Cache          DB      │
   └──────────────────────────┘
              │
              ├──► Email Queue ──► Worker ──► Email Provider ──┐
              │                                                 │
              ├──► SMS Queue   ──► Worker ──► SMS Provider   ──┤──► Client
              │                                                 │
              ├──► Push Queue  ──► Worker ──► Push Provider  ──┘
              │
              └──► Slack Queue ──► Worker ──► Slack Provider ──► Client


----------------------------------------------------------------
RETRY FLOW (on delivery failure)
----------------------------------------------------------------

worker reads from queue

  Worker ──► Third-party
                 │
            (fails ✗)
                 │
                 ▼
          Retry Queue
                 │
                 ▼
            Worker ──► Third-party
                            │
                       (success ✓)
                            │
                            ▼
                          Client


Cache stores:
  - User preferences
  - Device tokens
  - Reduces repeated DB hits

DB stores:
  - User info
  - Device info
  - Notification history

WHY MULTIPLE QUEUES?
  - Each channel (Email, SMS, Push) has its own queue + worker
  - One channel failing does NOT block others
  - Each can scale independently based on load
  - Retry is isolated — only failed messages are retried

Improvements over basic design:
  - Better scalability (workers scale independently)
  - Fault isolation (one channel failure doesn't affect others)
  - Faster processing (parallel queues)
  - Retry handling on failure

----------------------------------------------------------------
6. GENERIC / PRODUCTION-READY DESIGN
----------------------------------------------------------------
Adds Authentication, Rate Limiting, Analytics, Notification
Logs, and Templates on top of the improved design.

FULL FLOW:

  Service → Auth + Rate Limiter → Message Queue → Workers → Third-party → User
                    │                   ▲               │
              device/settings           │          Notification Logs
              user_info (DB)       Analytics             │
                 Cache              Service          Template DB

Step by step:
  1. Service sends a notification request
  2. Auth verifies the service identity
  3. Rate Limiter prevents spam / abuse / overload
  4. Request buffered in Message Queue (handles traffic spikes)
  5. Workers pick up from queue, apply Template, send to provider
  6. Third-party delivers to user device
  7. Delivery result logged in Notification Logs
  8. Analytics Service tracks open / click / delivery rates

Components Explained:

  Authentication:
    - Verify which service is sending the request
    - Prevent unauthorized usage of notification system

  Rate Limiter:
    - Cap how many notifications a service can send
    - Protect downstream systems from overload
    - Prevent spam / abuse

  Message Queue:
    - Decouples producers from workers
    - Absorbs traffic spikes without dropping requests
    - Separate queues per channel (Email / SMS / Push)

  Workers:
    - Channel-specific (Email worker, SMS worker, Push worker)
    - Fetch templates, format message, deliver via provider
    - Handle retry logic on failure

  Notification Logs:
    - Stores delivery status, failures, retry attempts
    - Used for debugging and auditing

  Template Service:
    - Predefined formats for each channel
      e.g. Email HTML layout, SMS text format
    - Dynamic placeholders: {{name}}, {{order_id}}

  Analytics Service:
    - Tracks delivery rate, open rate, click rate
    - Feeds data back from third-party responses
    - Provides insights for optimization and business decisions

----------------------------------------------------------------
FULL FLOW SUMMARY — ALL 3 DESIGNS
----------------------------------------------------------------

BASIC:
  Producer → Notification System → Third-party → User



----------------------------------------------------------------
IMPROVED DESIGN
----------------------------------------------------------------

  Service 1 ──┐
  Service 2 ──┤
  Service N ──┘
              │
              ▼
   ┌─────────────────────┐
   │  Notification Service│
   │   ┌──────┐ ┌──────┐ │
   │   │Cache │ │  DB  │ │
   │   └──────┘ └──────┘ │
   └─────────────────────┘
              │
              ├──► Email Queue ──► Email Worker ──► Email Provider ──┐
              │                         │                             │
              ├──► SMS Queue   ──► SMS Worker   ──► SMS Provider   ──┤──► User
              │                         │                             │
              ├──► Push Queue  ──► Push Worker  ──► Push Provider  ──┘
              │                         │
              │                    (on failure)
              │                         │
              └─────────────────► Retry Queue
                                        │
                                        ▼
                                  Worker retries




----------------------------------------------------------------
PRODUCTION (Generic) DESIGN
----------------------------------------------------------------

  Service
     │
     ▼
  Auth + Rate Limiter ──────────────► Analytics Service
     │                                       ▲
     │                                       │
     ▼                                       │
  Message Queue                        (tracks results)
     │                                       │
     ▼                                       │
  Workers ──► Template Service               │
     │             (formats message)         │
     │                                       │
     ├──► Email Provider ──┐                 │
     ├──► SMS Provider   ──┤──► User ────────┘
     └──► Push Provider  ──┘
     │
     ▼
  Notification Logs
  (delivery status, failures, retries)
     │
     ▼
  DB + Cache
  (user info, device tokens, preferences)
  
  

----------------------------------------------------------------
COMPONENT SUMMARY TABLE
----------------------------------------------------------------
  Component           Purpose
  ─────────────────── ─────────────────────────────────────────
  Auth                Verify sender identity
  Rate Limiter        Prevent spam, protect from overload
  Message Queue       Buffer requests, smooth traffic spikes
  Workers             Process & deliver per channel
  Cache (Redis)       Store tokens/preferences, reduce DB load
  Database            Persist users, devices, history
  Notification Logs   Track delivery status & failures
  Template Service    Reusable message formats with placeholders
  Analytics Service   Measure delivery, open, click rates
  Third-party         APNS, FCM, SendGrid, Twilio etc.

================================================================

```



## DROPBOX / CLOUD STORAGE (Google Drive, OneDrive)

```md
================================================================
   SYSTEM DESIGN: DROPBOX / CLOUD STORAGE (Google Drive, OneDrive)
================================================================

----------------------------------------------------------------
1. WHAT IS A SERVICE LIKE DROPBOX?
----------------------------------------------------------------
A cloud storage service that lets users store, sync, and share
files across multiple devices.
Examples: Dropbox, Google Drive, OneDrive

Client (Mobile) ⟷ Cloud Service ⟷ Client (Desktop)

Advantages:
  - Availability   : Data accessible anywhere with internet
  - Reliability    : Data secured and stays on cloud
  - Scalability    : Storage expands/reduces as needed

----------------------------------------------------------------
2. REQUIREMENTS & CONSIDERATIONS
----------------------------------------------------------------
Functional:
  - Users can upload & download from all configured clients
  - Users can share files with others
  - Offline editing of files
  - Synchronize files across different clients

Non-Functional:
  - Versioning: Restore previous versions of files
  - Premium subscriptions with more features
  - Very heavy on bandwidth (high uploads/downloads)

----------------------------------------------------------------
3. SCALE OF THE SYSTEM
----------------------------------------------------------------
  Total Users      : 500 million
  Active Users     : 100 million
  Devices per User : 3 (average)
  Files per User   : 200 (average)

  Total Files    => 200 × 500M         = 100 Billion files
  Total Storage  => 100B × 100 KB      = 10 Petabytes (PB)

----------------------------------------------------------------
HIGH LEVEL DESIGN
----------------------------------------------------------------

                        ┌──────────────────┐
                        │   Block Server   │──────────────────┐
                        └──────────────────┘                  │
                               ▲        |                     │
                               │        |                     ▼
  Clients ─────────────────────┤        |           ┌──────────────────┐
  (Mobile + Desktop)           │        |           │  Backend Storage │
        ▲                      │        |           │    (Amazon S3)   │
        │                      ▼        |__________ └──────────────────┘
        │               ┌──────────────────┐       |           
        │               │  Metadata Server │───────▼──────────|
        │               └──────────────────┘                  │
        │                      │    │                         ▼
        │                      │    └──────────────► ┌──────────────────┐
        │                      │                     │ Metadata Storage │
        │                      ▼                     └──────────────────┘
        │               ┌──────────────────┐
        └───────────────│   Sync Server    │
          (notifies     └──────────────────┘
           on change)

Components:
  - Block Server      : Handles file uploads, splits into blocks
  - Metadata Server   : Manages file info (name, size, owner, etc.)
                        Talks to both Backend Storage & Metadata Storage
  - Sync Server       : Notifies clients when files change
  - Backend Storage   : Blob/object store (Amazon S3) — actual file data
  - Metadata Storage  : Database for file metadata

----------------------------------------------------------------
5. UPLOADING FILES
----------------------------------------------------------------

Step 1 — Client requests upload:
  Client → Load Balancer → Block Server (Auth + Rate Limiter)

Step 2 — Block Server issues presigned URL:
  Block Server → presigned URL → Client

Step 3 — Client uploads directly, metadata updated:
  Client → (presigned URL) → Amazon S3 → upload complete → Metadata DB

  NOTE: Client uploads DIRECTLY to S3 using presigned URL.
        Large file data never passes through app servers.
        Saves bandwidth + improves performance.

----------------------------------------------------------------
6. DOWNLOADING FILES
----------------------------------------------------------------

Step 1 — Client requests file:
  Client → Load Balancer → Block Server (Auth + Rate Limiter) → Metadata DB

Step 2 — File is served:
  Amazon S3 → (caches hot files) → CDN → Client

  NOTE: CDN caches frequently accessed files at edge locations.
        Reduces latency for users + reduces load on S3.

----------------------------------------------------------------
7. SHARING FILES
----------------------------------------------------------------
No separate system needed. A "sharelist" field is added to the
file's metadata record with IDs of users who can access it.

Before sharing:                 After sharing:
{                               {
  "id": "123",                    "id": "123",
  "name": "file.txt",             "name": "file.txt",
  "size": 1000,                   "size": 1000,
  "mimeType": "text/plain",       "mimeType": "text/plain",
  "uploadedBy": "user1"           "uploadedBy": "user1",
}                                 "sharelist": ["user2","user3"]
                                }
                                

----------------------------------------------------------------
8. SYNCING FILES
----------------------------------------------------------------
When one client uploads/edits a file, all other clients of that
same user must be notified and updated automatically.

HOW IT WORKS:

  Each client sends changes to a shared Request Queue.
  The Sync Server reads from that queue and updates Metadata.
  Each client has its OWN Individual Response Queue to receive
  updates — so notifications are isolated per device.

FLOW:

  Client (Device 1) ──┐
  Client (Device 2) ──┼──► Request Queue ──► Sync Server ──► Metadata Storage
  Client (Device 3) ──┘                           │
	                                              │ pushes updates
                                                  ▼
                              ┌─────────────────────────────────┐
                              │   Individual Response Queues    │
                              │   [Queue A] ──► Device 1(laptop)│
                              │   [Queue B] ──► Device 2(mobile)│
                              │   [Queue C] ──► Device 3(tab)   │
                              └─────────────────────────────────┘

WHY INDIVIDUAL RESPONSE QUEUES?
  - If one shared queue was used for responses, all clients would
    receive ALL notifications — even ones not meant for them.
  - Each device gets only its own relevant updates.
  - Decouples clients — one slow client doesn't block others.

KEY POINTS:
  - Sync Server is the brain — coordinates what changed & who
    needs to know.
  - Metadata Storage is updated first, then clients are notified.
  - Uses async messaging (queue-based) — not direct connection —
    so the system stays reliable even if a client is offline.

----------------------------------------------------------------
9. FULL PICTURE (Complete Architecture)
----------------------------------------------------------------
Putting everything together — Upload, Download, and Sync
all happening through one unified architecture.

FULL ARCHITECTURE FLOW:

                          Amazon S3 (Blob Storage)
                          ▲              │
    upload via            │              │ upload
    presigned URL ────────┘              │ complete
                                         ▼
  Clients ──► Load Balancer ──► Application Server ──► Metadata Storage
    │              │             (Auth + Rate Limiter)       ▲
    │              │                                         │
    │         Message Queue ◄──► Synchronization Server ────-┘
    │         ┌──────────────┐
    │         │ Request Queue│ (client → server)
    │         │Response Queue│ (server → client)
    │         └──────────────┘
    │
    ▼
   CDN (download via cache)
    │
    ▼
  Client receives file

----------------------------------------------------------------
FULL FLOW SUMMARY — ALL 3 OPERATIONS
----------------------------------------------------------------

UPLOAD:
  Client → LB → App Server (Auth + Rate Limit)
                     │
                     └──► presigned URL → Client
                                           │
                                           ▼
                                       Amazon S3 → Metadata Storage

DOWNLOAD:
  Client → LB → App Server → Metadata Storage + Amazon S3
                                        │
                                    CDN (caches hot files)
                                        │
                                        ▼
                                      Client

SYNC (notify other devices):
  App Server → Sync Server ◄──► Message Queue ──► Client(s)
                    │
                    ▼
             Metadata Storage (updated first, then push to queues)

----------------------------------------------------------------
COMPONENTS SUMMARY
----------------------------------------------------------------
  Load Balancer       : Distributes incoming requests evenly
  Application Server  : Auth, Rate Limiting, business logic
  Amazon S3           : Stores actual file data (blob storage)
  Metadata Storage    : Stores file info (name, size, owner, etc.)
  Sync Server         : Coordinates file change notifications
  Message Queue       : Async communication between components
    - Request Queue   : Client changes going TO the server
    - Response Queue  : Server updates coming BACK to each client
  CDN                 : Caches frequently downloaded files at edge

================================================================
```


## NEWSFEED (Facebook, Instagram, Reddit Timeline)
```
================================================================
   SYSTEM DESIGN: NEWSFEED (Facebook, Instagram, Reddit Timeline)
================================================================

----------------------------------------------------------------
1. REQUIREMENTS & CONSIDERATIONS
----------------------------------------------------------------
Functional:
  - Available on both mobile and web
  - Publish updates and push to friends (most important feature)
  - Updates should appear in near real-time
  - Posts shown in reverse chronological order (newest first)

Clarifying Questions to Ask:
  - What traffic volume is expected?
  - What is the maximum number of friends a user can have?
  - What do posts contain? (Images / Text / Video)

----------------------------------------------------------------
2. IDEA OF THE SYSTEM (High Level)
----------------------------------------------------------------
User publishes a post → system fans it out to all friends' feeds.
User opens app → system fetches their pre-built or fresh feed.

FLOW:

  User (Client)
       │
       ▼
  Load Balancer
       │
       ▼
  Web Servers (Auth + Proxy + Rate Limiter)
       │
       ├──► Post Service      ──► Post Cache ──► Post DB
       │
       ├──► Fanout Service    ◄──► News Feed Cache
       │
       └──► Notification Service

Components:
  - Post Service       : Handles creating/storing new posts
  - Post Cache         : Fast access to recent posts
  - Post DB            : Persistent storage for all posts
  - Fanout Service     : Distributes post to all friends' feeds
  - News Feed Cache    : Stores pre-built feeds per user
  - Notification Svc   : Alerts friends of new posts

----------------------------------------------------------------
3. FANOUT SERVICE — TWO MODELS
----------------------------------------------------------------
Fanout = pushing one user's post to all their friends' feeds.

  FANOUT ON WRITE (Push Model)
  ─────────────────────────────
  News Feed cache is pre-populated at write time.
  When user posts → immediately pushed to all friends' caches.

  + Feed is generated in real-time, pushed immediately
  + Fetching news feed is fast (pre-computed, ready to read)

  - If user has many friends, generating feed gets very slow
  - Wastes resources for inactive users (pushed but never read)

  FANOUT ON READ (Pull Model)
  ─────────────────────────────
  News Feed cache is populated when user opens the app.
  Feed is built fresh on each read request.

  + Works better for inactive users (only built when needed)
  + Data is not pushed to everyone — saves bandwidth

  - Process is slow (built on demand)
  - User can have a bad experience (loading delay)

  BEST PRACTICE:
  Use a hybrid — Push model for regular users,
                  Pull model for users with many followers
                  or inactive users.

----------------------------------------------------------------
4. DESIGN OF THE FANOUT SERVICE
----------------------------------------------------------------
Step-by-step flow when a user publishes a post:

  Fanout Service
       │
       ├──(1)──► Graph Database
       │          (fetch list of all friends of the poster)
       │
       │    Graph DB ──(2)──► User Cache ◄──► User DB
       │                      (get device tokens, settings)
       │
       ├──(3)──► Message Queue
       │          (queue tasks for each friend)
       │
       ▼
  (4) Fanout Workers
       │
       └──(5)──► Newsfeed Cache
                  (write post_id + user_id into each friend's feed)

Step by step explained:
  1. Fanout Service queries Graph DB to get all friends
  2. Graph DB returns friend list → look up user info from
     User Cache / User DB (tokens, preferences)
  3. Push fan-out tasks into Message Queue (one per friend)
  4. Fanout Workers pick up tasks from queue
  5. Workers write {post_id, user_id} entry into Newsfeed Cache
     for each friend

Newsfeed Cache stores:
  ┌─────────┬─────────┐
  │ post_id │ user_id │
  ├─────────┼─────────┤
  │ post_id │ user_id │
  │ post_id │ user_id │
  │ post_id │ user_id │
  └─────────┴─────────┘
  (a list of post IDs mapped to the user they belong to)

NOTE: Cache stores only IDs, not full post content.
      Actual post data fetched from Post DB when user reads feed.

----------------------------------------------------------------
5. RETRIEVING / UPDATING POSTS
----------------------------------------------------------------
When user opens the app and wants to read their feed:

  Client sends GET request
       │
       ▼
  Web Server → Fanout Service → Newsfeed Cache
                                     │
                                     │ (get list of post_ids)
                                     ▼
                                Post DB / Post Cache
                                     │
                                     │ (fetch actual post content)
                                     ▼
                                  Client (feed displayed)

When user creates a new post:

  Client sends POST request
       │
       ▼
  Web Server → Post Service → Post Cache → Post DB
                    │
                    ▼
              Fanout Service → (fans out to all friends as above)

----------------------------------------------------------------
6. FULL PICTURE (Complete Architecture)
----------------------------------------------------------------

  User
   │
   ├── GET (read feed) ◄─────────────────────────────┐
   └── POST (publish)                                 │
         │                                            │
         ▼                                            │
   Load Balancer                                      │
         │                                            │
         ▼                                   Newsfeed Cache
   Web Servers                                (pre-built feeds)
   (Auth + Proxy + Rate Limiter)                      ▲
         │                                            │
         ├──► Post Service ──► Post Cache ──► Post DB │
         │                                            │
         ├──► Fanout Service ──► Message Queue ──► Fanout Workers
         │          │                                 │
         │     Graph DB                               │
         │    User Cache ◄──► User DB          Newsfeed Cache
         │
         └──► Notification Service ──► Friends notified

----------------------------------------------------------------
COMPONENT SUMMARY
----------------------------------------------------------------
  Component            Purpose
  ──────────────────── ────────────────────────────────────────
  Web Servers          Auth, Proxy, Rate Limiting
  Post Service         Create and store new posts
  Post Cache           Fast reads for recent posts
  Post DB              Persistent post storage
  Fanout Service       Distribute posts to all friends' feeds
  Graph Database       Stores friend relationships
  User Cache / DB      Stores user info and device tokens
  Message Queue        Buffers fanout tasks per friend
  Fanout Workers       Process queue, write to Newsfeed Cache
  Newsfeed Cache       Pre-built feed (post_id + user_id pairs)
  Notification Svc     Alerts friends when new post is published

================================================================
```



## CONSISTENT HASHING

```
================================================================
   WHY DO WE NEED MULTIPLE DATABASES?
================================================================

----------------------------------------------------------------
START WITH ONE DATABASE
----------------------------------------------------------------
In the beginning, every system starts simple:

  Client ◄──► Web Server ◄──► Single Database

This works perfectly fine when:
  - Users are few (hundreds or thousands)
  - Data is small
  - Traffic is low

----------------------------------------------------------------
WHAT HAPPENS WHEN YOUR APP GROWS?
----------------------------------------------------------------
Imagine you built an app like Instagram.

Day 1   →      1,000 users   → one DB handles it fine
Month 1 →    100,000 users   → DB getting slower
Month 6 →  1,000,000 users   → DB struggling
Year 1  → 10,000,000 users   → DB completely overwhelmed

One database has LIMITS:
  - Only one CPU
  - Limited RAM
  - Limited disk speed
  - Can only handle X reads/writes per second

----------------------------------------------------------------
THE PROBLEMS WITH ONE DATABASE
----------------------------------------------------------------

1. PERFORMANCE
  - Too many requests hit the same DB
  - DB becomes a BOTTLENECK
  - Every user experiences slowness
  - Queries start timing out

2. STORAGE LIMIT
  - One machine has limited disk space
  - You can't store infinite data on one server
  - Example: Instagram stores billions of photos
    → impossible on one machine

3. SINGLE POINT OF FAILURE
  - If that one DB goes down → entire app goes down
  - No backup, no fallback
  - Every user is affected immediately

  Client ──► Web Server ──► DB (crashed) ✗
                               ↑
                          Everything stops

4. GEOGRAPHIC LATENCY
  - DB is in one location (say, US)
  - User in India makes a request
  - Data has to travel US → India every single time
  - Very slow for international users

----------------------------------------------------------------
THE SOLUTION — MULTIPLE DATABASES
----------------------------------------------------------------
Spread the load, storage, and responsibility across many DBs.

  Client ──► Web Server ──► DB [0]  (handles some users)
                        ──► DB [1]  (handles some users)
                        ──► DB [2]  (handles some users)

Now:
  - Each DB handles only a FRACTION of the total load
  - Storage is split across machines
  - If one DB fails → others still work
  - Can place DBs in different regions (closer to users)

----------------------------------------------------------------
REAL WORLD NUMBERS
----------------------------------------------------------------
  Instagram  → billions of photos → needs 1000s of DB servers
  WhatsApp   → 100B messages/day  → one DB would melt instantly
  Amazon     → millions of orders → needs parallel DB processing

----------------------------------------------------------------
TYPES OF MULTIPLE DB SETUPS
----------------------------------------------------------------

Replication (copies):
  Master DB ──► Slave DB [1]  (exact copy)
            ──► Slave DB [2]  (exact copy)
  - Reads spread across slaves
  - Writes go to master
  - If master fails → slave takes over

Sharding (splitting):
  DB [0] → stores users 0–33%
  DB [1] → stores users 33–66%
  DB [2] → stores users 66–100%
  - Data is DIVIDED not copied
  - Each DB owns a portion
  - This is where consistent hashing comes in

----------------------------------------------------------------
SO THE CHAIN IS:
----------------------------------------------------------------

  App grows → one DB not enough
      ↓
  Need multiple DBs (sharding)
      ↓
  Need a way to decide which DB stores which data
      ↓
  Use hashing: event_id % no_of_servers
      ↓
  Servers added/removed → formula breaks, all data remaps
      ↓
  Need a smarter solution
      ↓
  CONSISTENT HASHING

----------------------------------------------------------------
ONE-LINE ANSWER
----------------------------------------------------------------
One database has limits on speed, storage, and reliability.
As your app grows, you MUST split data across multiple databases.
And once you have multiple DBs, you need a smart way to decide
which data goes where — that's why consistent hashing exists.

================================================================
```

```
================================================================
   SYSTEM DESIGN: CONSISTENT HASHING
================================================================

----------------------------------------------------------------
1. THE PROBLEM — HOW TO DECIDE WHICH DATABASE?
----------------------------------------------------------------
When you have multiple databases, how do you decide which DB
stores or retrieves a particular piece of data?

Basic approach (Naive Hashing):
  db_index = event_id % no_of_servers

FLOW (basic setup):
  Client ◄──► Web Servers ◄──► DB [0] / DB [1] / DB [2]

Example with 3 servers:
  1234 % 3 = 1  → goes to DB [1]
  6666 % 3 = 0  → goes to DB [0]
  5612 % 3 = 2  → goes to DB [2]

This works fine — UNTIL you add or remove a server.

----------------------------------------------------------------
2. PROBLEM: REMOVING A DB
----------------------------------------------------------------
If DB [1] goes down, no_of_servers changes from 3 to 2.

  db_index = event_id % no_of_servers

  Before removal (3 servers):   After removal (2 servers):
  6666 % 3 = 0 → DB [0]         6666 % 2 = 0 → DB [0]  (same)
  1234 % 3 = 1 → DB [1]         1234 % 2 = 0 → DB [0]  (WRONG!)
  5612 % 3 = 2 → DB [2]         5612 % 2 = 0 → DB [0]  (WRONG!)

FLOW:
  Client ◄──► Web Servers ──► DB [2]
                          ──► DB [1] (REMOVED ✗)
                          ──► DB [0]

Problem:
  - Almost ALL data mappings change
  - System tries to find data in wrong DB
  - Massive cache misses + data loss risk
  - Every key needs to be remapped

----------------------------------------------------------------
3. PROBLEM: ADDING A NEW DB
----------------------------------------------------------------
If you add DB [3], no_of_servers changes from 3 to 4.

  Before adding (3 servers):    After adding (4 servers):
  6666 % 3 = 0 → DB [0]         6666 % 4 = 2 → DB [2]  (WRONG!)
  1234 % 3 = 1 → DB [1]         1234 % 4 = 1 → DB [1]  (same)
  5612 % 3 = 2 → DB [2]         5612 % 4 = 0 → DB [0]  (WRONG!)

Problem:
  - Adding one DB reshuffles most of the data
  - Same massive remapping issue
  - Not scalable

ROOT CAUSE of both problems:
  Naive hashing uses % no_of_servers
  → any change in server count = almost full reshuffle

----------------------------------------------------------------
4. SOLUTION — CONSISTENT HASHING
----------------------------------------------------------------
Instead of hashing to a server index directly, we map both
SERVERS and KEYS onto a fixed circular ring (hash ring).

HOW THE RING WORKS:
  - Ring has positions 0 to 99 (or 0 to 2^32 in real systems)
  - Each DB is placed at a position on the ring using a hash
  - Each key (event_id) is also hashed to a position on the ring
  - Key goes to the FIRST DB found going CLOCKWISE from its position

RING LAYOUT (example with 4 DBs):

              DB [0]  (position ~0)
           /                        \
     (94) /                          \ (6)
         /                            \
   DB[2]                               DB[1]
   (~75)                               (~25)
         \                            /
     (63) \                          / (37)
           \                        /
              DB [3]  (position ~50)

Key assignment examples:
  1234 % 100 = 34  → goes clockwise → hits DB [1] at ~25... 
                      wait, 34 > 25, so next clockwise = DB [3] at 50
  6666 % 100 = 66  → goes clockwise → hits DB [2] at 75
  5612 % 100 = 12  → goes clockwise → hits DB [1] at 25

RULE: Key always goes to the next DB clockwise on the ring.

----------------------------------------------------------------
5. REMOVING A DB WITH CONSISTENT HASHING
----------------------------------------------------------------
When DB [3] is removed from the ring:

  BEFORE:                         AFTER:
  Keys 26–50 → DB [3]             Keys 26–50 → DB [1]
                                  (just move to next clockwise DB)

  All other keys → UNCHANGED

  DB [0] ──────── DB [0]
  DB [1] ──────── DB [1]  ← absorbs DB[3]'s keys
  DB [2] ──────── DB [2]
  DB [3] ──────── REMOVED ✗

Only the keys that WERE on DB [3] need to move.
Everything else stays exactly where it is.

----------------------------------------------------------------
6. ADDING A NEW DB WITH CONSISTENT HASHING
----------------------------------------------------------------
When DB [4] is added between DB [1] and DB [3]:
And DB [5] is added between DB [2] and DB [0]:

  DB [4] placed at ~42 on ring
    → Takes keys between 26–42 from DB [1]
    → Only those keys move, rest unchanged

  DB [5] placed at ~87 on ring
    → Takes keys between 76–87 from DB [0]
    → Only those keys move, rest unchanged

Only the keys in the NEW DB's segment are affected.
All other keys stay on their current DB.

----------------------------------------------------------------
7. VIRTUAL NODES
----------------------------------------------------------------
Problem with basic consistent hashing:
  - DBs may get uneven key distribution
  - One DB could get many more keys than others
  - Not truly balanced

Solution — Virtual Nodes:
  - Each physical DB is represented by MULTIPLE positions on ring
  - Each position = one virtual node

RING WITH VIRTUAL NODES (each DB has 3 virtual spots):

              DB [0]  (positions: 0, 94, ...)
           /                                  \
          /                                    \
   DB[2]                                        DB[1]
   (positions: 75, 63, ...)      (positions: 25, 31, ...)
          \                                    /
           \                                  /
              DB [3]  (positions: 50, 42, ...)

  Numbers inside ring = 0, 6, 12, 17, 25, 31, 37, 42,
                        50, 56, 63, 69, 75, 81, 87, 94

Virtual node labels around ring (which DB owns each slot):
  Position 0  → DB[0]     Position 50 → DB[3]
  Position 6  → DB[0]     Position 56 → DB[3]
  Position 12 → DB[1]     Position 63 → DB[2]
  Position 17 → DB[1]     Position 69 → DB[2]
  Position 25 → DB[1]     Position 75 → DB[2]
  Position 31 → DB[1]     Position 81 → DB[2]
  Position 37 → DB[3]     Position 87 → DB[0]
  Position 42 → DB[3]     Position 94 → DB[0]

Benefits:
  - Keys are distributed more evenly across all DBs
  - When a DB is added/removed, its virtual nodes spread
    the impact evenly across ALL other DBs (not just one neighbor)
  - More virtual nodes = better balance

----------------------------------------------------------------
SUMMARY — NAIVE HASHING vs CONSISTENT HASHING
----------------------------------------------------------------
  ┌─────────────────────┬──────────────────┬──────────────────┐
  │ Feature             │ Naive Hashing    │ Consistent Hash  │
  ├─────────────────────┼──────────────────┼──────────────────┤
  │ Formula             │ key % n_servers  │ Hash ring        │
  │ Add server          │ Full reshuffle   │ Only 1 segment   │
  │ Remove server       │ Full reshuffle   │ Only 1 segment   │
  │ Keys remapped       │ Almost all       │ Minimal (1/n)    │
  │ Balance             │ Even             │ Uneven (fix with │
  │                     │                  │ virtual nodes)   │
  └─────────────────────┴──────────────────┴──────────────────┘

ONE-LINE TAKEAWAY:
  Consistent hashing maps servers and keys onto a ring.
  Adding/removing a server only affects its neighbouring keys —
  not the entire dataset. Virtual nodes ensure even distribution.

================================================================
```



```
================================================================
   API GATEWAY — COMPLETE STRUCTURED NOTES
================================================================

----------------------------------------------------------------
1. THE PROBLEM — CHAOS OF MICROSERVICES
----------------------------------------------------------------
Modern apps are built with multiple microservices.
Without a gateway, clients talk directly to each service.

BASIC SETUP (no gateway):

  Clients (Web/Mobile/Tablet)
       │
       ├──► Payment Service
       ├──► Subscription Service
       ├──► Users Service
       └──► Notification Service

PROBLEMS this creates:

  - Chaotic Direct Connections
    Every client knows about every service
    Tightly coupled — change one service = update all clients

  - Redundant Logic in Services
    Every service must implement its own:
    Auth, rate limiting, logging, SSL — duplicated everywhere

  - No Unified Monitoring
    No single place to see all traffic and errors

  - Security Vulnerabilities
    Every service is exposed directly to the internet
    Each one is an attack surface

  - Scaling Nightmares
    Hard to scale individual services independently
    No central control point

  - Inefficient Data Aggregation
    Client needs data from 3 services?
    → Makes 3 separate calls, waits for all 3

----------------------------------------------------------------
2. WHAT IS AN API GATEWAY?
----------------------------------------------------------------
A single entry point that sits between clients and
all your microservices.

All requests go THROUGH the gateway.
Gateway decides what to do with them.

FLOW WITH GATEWAY:

  Clients (Web / Mobile / Tablet)
       │
       │  GET / POST
       ▼
  ┌─────────────────────────────┐
  │         API Gateway         │
  │                             │
  │   1. Validate Request       │
  │   2. Run Middleware         │
  │   3. Reroute                │
  │   4. Transform Response     │
  └─────────────────────────────┘
       │
       ├──► Payment Service
       ├──► Subscription Service
       ├──► Users Service
       └──► Notification Service

Clients only know about ONE address — the API Gateway.
Services are hidden from the outside world.

----------------------------------------------------------------
3. HOW API GATEWAY FILTERS REQUESTS (Funnel)
----------------------------------------------------------------
Every request passes through layers before reaching a service.
Think of it as a funnel — only valid requests get through.

FILTERING LAYERS (top to bottom):

  All incoming requests (many)
         │
         ▼
  TLS Termination       ← handles SSL/HTTPS
         │
         ▼
  Authentication        ← verify JWT / API keys
         │
         ▼
  Routing               ← which service handles this?
         │
         ▼
  Transformation        ← modify request/response format
         │
         ▼
  Aggregation           ← combine responses from services
         │
         ▼
  Services (fewer, valid requests only)

Inside the gateway at each step:
  Validate request → Run Middleware → Reroute → Transform response

----------------------------------------------------------------
4. REQUEST VALIDATION PROCESS
----------------------------------------------------------------
Every incoming request goes through this validation pipeline:

  Receive Request
       │
       ▼
  Validate URL          ← is the endpoint correct?
       │
       ▼
  Check Headers         ← auth token present? content-type?
       │
       ▼
  Validate Body         ← required fields? correct format?
       │
       ├── Valid ──────────────────► Forward to Service
       │
       └── Invalid ────────────────► Reject Invalid Request
                                            │
                                            ▼
                                     Send Error Message
                                     (400 / 401 / 403)

----------------------------------------------------------------
5. MIDDLEWARE — WHAT GATEWAY DOES IN THE MIDDLE
----------------------------------------------------------------
The gateway runs middleware on every request.
These are cross-cutting concerns handled ONCE centrally.

Middleware responsibilities:

  Security:
    - Authenticate requests using JWT tokens
    - Whitelist / blacklist IPs
    - Terminate SSL connections (TLS termination)
    - Handle CORS headers

  Performance:
    - Compress responses
    - Validate request sizes
    - Handle response timeouts
    - Throttle traffic

  Monitoring:
    - Log and monitor all traffic
    - Unified visibility across all services

  Compliance:
    - Version APIs (v1, v2, v3)
    - Integrate with service discovery
    - Limit request rates to prevent abuse

Without gateway → each service implements ALL of this itself.
With gateway    → done ONCE, applied to EVERY service.

----------------------------------------------------------------
6. PROS AND CONS OF API GATEWAY
----------------------------------------------------------------

  PROS:
    - Centralized security
      Auth, SSL, IP filtering in one place
    - Reduced coupling
      Clients don't know about individual services
    - Improved performance
      Caching, compression, aggregation
    - Scalability
      Services scale independently behind the gateway
    - Fault tolerance
      Gateway can reroute if a service goes down

  CONS:
    - Operational complexity
      One more system to deploy, manage, and monitor
    - Cost
      Running and maintaining a gateway adds infrastructure cost
    - Latency
      Every request goes through an extra hop
      Adds a small but real delay

  ┌──────────────────────┬──────────────────────────────────┐
  │ PROS                 │ CONS                             │
  ├──────────────────────┼──────────────────────────────────┤
  │ Centralized security │ Operational complexity           │
  │ Reduced coupling     │ Added cost                       │
  │ Better performance   │ Extra latency per request        │
  │ Scalability          │                                  │
  │ Fault tolerance      │                                  │
  └──────────────────────┴──────────────────────────────────┘

----------------------------------------------------------------
7. FULL PICTURE — COMPLETE FLOW
----------------------------------------------------------------

  Client (Web / Mobile / Tablet)
       │
       │ GET or POST request
       ▼
  ┌─────────────────────────────┐
  │         API Gateway         │
  │                             │
  │  ┌───────────────────────┐  │
  │  │   Validate Request    │  │ ← check URL, headers, body
  │  ├───────────────────────┤  │
  │  │    Run Middleware      │  │ ← auth, rate limit, logging
  │  ├───────────────────────┤  │
  │  │       Reroute         │  │ ← decide which service
  │  ├───────────────────────┤  │
  │  │  Transform Response   │  │ ← format response for client
  │  └───────────────────────┘  │
  └─────────────────────────────┘
       │
       ├──────────────────► Payment Service
       ├──────────────────► Subscription Service
       ├──────────────────► Users Service
       └──────────────────► Notification Service

Response travels back:
  Service → Gateway (transforms) → Client

----------------------------------------------------------------
COMPONENT SUMMARY
----------------------------------------------------------------
  Component           Purpose
  ─────────────────── ──────────────────────────────────────────
  API Gateway         Single entry point for all clients
  TLS Termination     Handle HTTPS at gateway, not each service
  Authentication      Verify JWT / API keys centrally
  Rate Limiter        Prevent abuse and overload
  Router              Direct request to correct microservice
  Transformer         Convert request/response formats
  Aggregator          Combine responses from multiple services
  Middleware          Cross-cutting logic run on every request
  Request Validator   Check URL, headers, body before forwarding

================================================================
```


Atomic Operations?


```
================================================================
   QUICK-COMMERCE SYSTEM DESIGN — (Blinkit, Zepto, GoPuff)
   COMPLETE STRUCTURED NOTES
================================================================

----------------------------------------------------------------
1. THE PROBLEM — WHY QUICK-COMMERCE?
----------------------------------------------------------------
Traditional e-commerce (Amazon, Flipkart) delivers in 1-7 days.
Modern users want EVERYDAY ITEMS in MINUTES, not days.

PROBLEMS with traditional e-commerce for groceries:

  - Slow Delivery
    1-2 days is too long for milk, bread, snacks

  - Centralized Warehouses
    One huge warehouse far from customer = long delivery time

  - Inventory Mismatch
    No real-time stock view per location

  - No Geographic Awareness
    Cannot route to the NEAREST stock location

QUICK-COMMERCE SOLUTION:
  Small warehouses (dark stores) placed CLOSE to users.
  Promise: Delivery in 10-30 minutes.

Famous examples:
  - Blinkit (India)
  - Zepto  (India)
  - GoPuff (USA)
  - Getir  (Europe)

----------------------------------------------------------------
2. THE 30-MINUTE MAGIC (HIGH-LEVEL FLOW)
----------------------------------------------------------------
Every quick-commerce order follows this lifecycle:

  User (at home)
       │
       │ Opens app, picks items
       ▼
  ┌─────────────────────────────┐
  │       Mobile App            │
  │   (Browse + Add to cart)    │
  └─────────────────────────────┘
       │
       │ Sends request
       ▼
  ┌─────────────────────────────┐
  │       Cloud Server          │
  │    (Backend services)       │
  └─────────────────────────────┘
       │
       ├──► Nearest Warehouse #1  ◄── stock check
       ├──► Nearest Warehouse #2  ◄── stock check
       └──► Delivery Agent        ◄── pickup + drop

  Result: Order delivered in ~30 minutes.

KEY INSIGHT:
  Unlike Amazon, Q-commerce places MANY small warehouses
  (dark stores) close to customers — not one giant warehouse
  far away. This is what makes 30-min delivery possible.

----------------------------------------------------------------
3. SYSTEM REQUIREMENTS
----------------------------------------------------------------
Before designing, we list what the system MUST do.

FUNCTIONAL + NON-FUNCTIONAL REQUIREMENTS:

  ┌────────────────────────┬─────────────────────────────────┐
  │ Requirement            │ Description                     │
  ├────────────────────────┼─────────────────────────────────┤
  │ Availability Connection│ Connect to any DC within 1 hour │
  │ Ordering Consistency   │ Strong consistency across nodes │
  │ Order Volume           │ Handle 1 million orders/day     │
  │ Order Items            │ Users can order any catalog item│
  │ Availability Speed     │ 100ms response time             │
  │ System Size            │ 10,000 DCs, 100,000 items each  │
  │ Query Availability     │ Stock query by location < 1 hr  │
  └────────────────────────┴─────────────────────────────────┘

BACK-OF-ENVELOPE MATH:
  1,000,000 orders/day ÷ 86,400 sec ≈ 12 orders/sec average
  Peak load: ~5x average → ~60 orders/sec at peak
  Catalog rows: 100,000 items × 10,000 DCs = 1 billion rows

WHY THESE MATTER:
  - Connection      → user must reach a DC quickly
  - Consistency     → no overselling the same item
  - Volume          → drives scale planning
  - Speed (100ms)   → user won't wait long

----------------------------------------------------------------
4. ESSENTIAL TERMINOLOGY (4 CORE ENTITIES)
----------------------------------------------------------------
Understanding these 4 entities is CRITICAL.

  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐
  │ Inventory  │  │    Item    │  │     DC     │  │   Order    │
  │            │  │            │  │            │  │            │
  │  Physical  │  │   Type of  │  │  Physical  │  │ Collection │
  │  instance  │  │   item     │  │  location  │  │ of items   │
  │   at a DC  │  │ e.g.Cheetos│  │ storing    │  │ by a user  │
  │            │  │            │  │   items    │  │            │
  └────────────┘  └────────────┘  └────────────┘  └────────────┘

KEY DISTINCTION — Item vs Inventory:

  Item       = Abstract product (e.g. "Lays Classic 50g")
  Inventory  = Physical stock of that item at a specific DC

  Same Item can exist as Inventory across MANY DCs.

EXAMPLE:
  Item: "Lays Classic 50g"
    ├── Inventory at DC-Mumbai-01 : 30 packets
    ├── Inventory at DC-Mumbai-02 : 12 packets
    └── Inventory at DC-Delhi-05  : 50 packets

----------------------------------------------------------------
5. HIGH-LEVEL ARCHITECTURE — TYING IT TOGETHER
----------------------------------------------------------------
The system has 4 main layers.

FULL ARCHITECTURE FLOW:

  ┌──────────────────────┐
  │  Client Application  │
  │   (Mobile / Web)     │
  │   iOS/Android/React  │
  └──────────────────────┘
            │
            │ HTTPS / REST
            ▼
  ┌──────────────────────────┐
  │       API Gateway        │
  │                          │
  │  • Authentication & Auth │
  │  • Rate Limiting         │
  │  • Load Balancing        │
  │  • Request Routing       │
  │                          │
  │   Port: 8080 | NGINX     │
  └──────────────────────────┘
            │
            │ Routes to correct microservice
            │
            ├────────────────────┬────────────────────┐
            ▼                    ▼                    ▼
  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
  │   Availability   │ │     Location     │ │      Order       │
  │     Service      │ │     Service      │ │     Service      │
  │                  │ │                  │ │                  │
  │  Real-time       │ │  Find nearest    │ │  Process orders  │
  │  inventory check │ │  DCs             │ │  + payments      │
  │                  │ │                  │ │                  │
  │   Port: 8081     │ │   Port: 8082     │ │   Port: 8083     │
  └──────────────────┘ └──────────────────┘ └──────────────────┘
            │                    │                    │
            │ Query Inventory    │ Location Lookup    │ Write Transaction
            ▼                    ▼                    ▼
  ┌────────────────────────────────────────────────────────────┐
  │              Distribution Center Database                  │
  │              (Regional Fulfillment Hub)                    │
  │                                                            │
  │  DC Properties:                                            │
  │    dc_id, latitude, longitude, address, capacity           │
  │                                                            │
  │  ┌──────────────┐   ┌────────────────┐                     │
  │  │    ITEMS     │   │   INVENTORY    │                     │
  │  │  item_id PK  │   │ inventory_id PK│                     │
  │  │  name        │   │ item_id FK     │                     │
  │  │  description │   │ dc_id FK       │                     │
  │  │  category    │   │ quantity       │                     │
  │  │  price       │   │ reserved_qty   │                     │
  │  └──────────────┘   └────────────────┘                     │
  │                                                            │
  │  ┌──────────────┐   ┌────────────────┐                     │
  │  │   ORDERS     │   │  ORDER_ITEMS   │                     │
  │  │  order_id PK │   │ order_item_id  │                     │
  │  │  customer_id │   │ order_id FK    │                     │
  │  │  dc_id FK    │   │ item_id FK     │                     │
  │  │  status      │   │ quantity       │                     │
  │  │  total_amount│   │ unit_price     │                     │
  │  └──────────────┘   └────────────────┘                     │
  └────────────────────────────────────────────────────────────┘

WHY MICROSERVICES?
  - Each service scales independently
  - Failure in one (e.g. Availability) doesn't kill ordering
  - Different teams own different services
- [ ] 
----------------------------------------------------------------
6. NEARBY (LOCATION) SERVICE
----------------------------------------------------------------
PURPOSE: Given user coordinates, find nearest DCs.

FLOW:

  Input Coordinates    Calculate Distance    Return DC List
       (1)                  (2)                   (3)
        │                    │                     │
        ▼                    ▼                     ▼
  ┌────────────┐      ┌────────────┐         ┌────────────┐
  │  lat/long  │ ───► │ Geospatial │ ──────► │  DC list   │
  │  radius_km │      │   Query    │         │  (sorted)  │
  └────────────┘      └────────────┘         └────────────┘

INPUT PARAMETERS:
  latitude  : DECIMAL(10,8)
  longitude : DECIMAL(11,8)
  radius_km : INTEGER

PROCESSING:
  - Haversine distance formula (great-circle distance)
  - Earth curvature compensation
  - Geospatial indexing for fast lookup

DC_LOCATIONS TABLE:

  ┌──────────────────────┐
  │     DC_LOCATIONS     │
  ├──────────────────────┤
  │  dc_id     (PK)      │
  │  latitude            │
  │  longitude           │
  │  address             │
  │  capacity            │
  │  region              │
  └──────────────────────┘

WHY GEOSPATIAL INDEXING?
  - Naive distance calc across 10,000 DCs → SLOW
  - Geospatial index (PostGIS, Geohash, R-Tree)
  - Reduces query time from O(N) to O(log N)

OUTPUT:
  List of DC IDs sorted by distance from user.

----------------------------------------------------------------
7. AVAILABILITY SERVICE
----------------------------------------------------------------
PURPOSE: Given a list of DCs, return what items are in stock.

FLOW:

  Input DC List      JOIN Query        Return Inventory
       (1)              (2)                  (3)
        │                │                    │
        ▼                ▼                    ▼
  ┌────────────┐  ┌──────────────┐     ┌────────────┐
  │  dc_ids[]  │─►│ JOIN Items   │ ──► │ {item:qty} │
  │ item_filter│  │ + Inventory  │     │  mapping   │
  │  category  │  └──────────────┘     └────────────┘
  └────────────┘

INPUT PARAMETERS:
  dc_ids      : Array<UUID>          (from Nearby Service)
  item_filter : String (optional)
  category    : String (optional)

PROCESSING:
  - JOIN Items table with Inventory table
  - Filter by DC IDs from Nearby Service
  - Return list of {item_id: quantity}

DATABASE TABLES:

  ┌────────────────┐         ┌────────────────────┐
  │     ITEMS      │         │     INVENTORY      │
  ├────────────────┤         ├────────────────────┤
  │  item_id (PK)  │◄────────│  item_id (FK)      │
  │  name          │         │  inventory_id (PK) │
  │  description   │         │  dc_id (FK)        │
  │  category      │         │  quantity          │
  │  price         │         │  reserved_qty      │
  └────────────────┘         │  available_qty     │
                             └────────────────────┘

JOIN LOGIC (SQL):
  SELECT i.name, inv.available_qty
  FROM items i
  JOIN inventory inv ON i.item_id = inv.item_id
  WHERE inv.dc_id IN (dc_id_list)
    AND inv.available_qty > 0;

WHY available_qty AND reserved_qty?
  - quantity       = total physical stock
  - reserved_qty   = locked by in-progress orders
  - available_qty  = quantity - reserved_qty
  → Prevents 2 users buying the last item simultaneously

----------------------------------------------------------------
8. ORDER SERVICE
----------------------------------------------------------------
PURPOSE: Accept an order and atomically commit it to the DB.

FLOW:

  Order Request    Atomic Transaction    Order Confirmed
       (1)               (2)                  (3)
        │                 │                    │
        ▼                 ▼                    ▼
  ┌────────────┐   ┌──────────────┐     ┌────────────┐
  │ customer_id│──►│  PostgreSQL  │ ──► │  order_id  │
  │  items[]   │   │ ACID + Locks │     │  returned  │
  │   dc_id    │   └──────────────┘     └────────────┘
  └────────────┘

ORDER REQUEST:
  customer_id : UUID
  items       : [{item_id, quantity}]
  dc_id       : UUID

ATOMIC TRANSACTION (all-or-nothing):
  1. Check inventory availability
  2. Create order + order_items records
  3. Update inventory (decrement quantity)
  4. Commit or rollback as a single unit

DATABASE: PostgreSQL
  - ACID transactions
  - Row-Level Locking

INSIDE THE TRANSACTION:

  ┌─────────────────────────────────────┐
  │       PostgreSQL Database           │
  │                                     │
  │  ┌─────────────┐ ┌────────────────┐ │
  │  │   ORDERS    │ │  ORDER_ITEMS   │ │
  │  │ order_id PK │ │ order_item_id  │ │
  │  │ customer_id │ │ order_id FK    │ │
  │  │ dc_id FK    │ │ item_id FK     │ │
  │  │ status      │ │ quantity       │ │
  │  │ total_amount│ │ unit_price     │ │
  │  └─────────────┘ └────────────────┘ │
  │                                     │
  │  ┌──────────────────────┐           │
  │  │  INVENTORY (LOCKED)  │ ◄── lock  │
  │  │  inventory_id (PK)   │   during  │
  │  │  item_id (FK)        │   txn     │
  │  │  dc_id (FK)          │           │
  │  │  quantity            │           │
  │  │  reserved_qty        │           │
  │  └──────────────────────┘           │
  └─────────────────────────────────────┘

WHY ROW-LEVEL LOCKING?
  Scenario: 2 users buy the LAST Cheetos packet at the same time
    - Without locking → both succeed → oversold ✗
    - With locking    → only 1 succeeds → consistent ✓

WHY ACID?
  - Atomicity   : Order fully placed OR fully rolled back
  - Consistency : No invalid states (no negative stock)
  - Isolation   : Concurrent orders don't conflict
  - Durability  : Confirmed orders survive crashes

----------------------------------------------------------------
9. END-TO-END REQUEST FLOW (INTERVIEW READY)
----------------------------------------------------------------

  User: "Order 2 packets of Lays from my location"

  Step 1: Client → API Gateway
    ┌─────────────┐
    │  Auth check │ ← JWT token
    │  Rate limit │
    │   Routing   │
    └─────────────┘

  Step 2: Gateway → Location Service
    Input  : user lat/long
    Output : [DC_42, DC_43, DC_17]   ← nearby DCs

  Step 3: Gateway → Availability Service
    Input  : [DC_42, DC_43, DC_17] + "Lays"
    Output : {DC_42: 30, DC_43: 12}  ← stock per DC

  Step 4: User confirms → Gateway → Order Service
    ┌──────────────────────────────────┐
    │  BEGIN TRANSACTION               │
    │    LOCK row for Lays @ DC_42     │
    │    UPDATE inventory: qty -= 2    │
    │    INSERT INTO orders            │
    │    INSERT INTO order_items       │
    │  COMMIT                          │
    └──────────────────────────────────┘

  Step 5: Order Confirmed
    Return order_id → user
    Trigger delivery agent dispatch (async)

----------------------------------------------------------------
10. KEY DESIGN DECISIONS — WHY?
----------------------------------------------------------------

  ┌──────────────────────────┬────────────────────────────────┐
  │ Decision                 │ Reason                         │
  ├──────────────────────────┼────────────────────────────────┤
  │ Microservices            │ Independent scaling & failure  │
  │ PostgreSQL for orders    │ ACID needed for transactions   │
  │ Row-level locking        │ Prevent overselling            │
  │ Regional DC databases    │ Low latency, data locality     │
  │ Geospatial indexing      │ Fast nearest-DC lookups        │
  │ API Gateway              │ Central auth + rate limiting   │
  │ Reserved_qty column      │ Handle in-flight orders        │
  └──────────────────────────┴────────────────────────────────┘

CONSISTENCY CHOICE (CAP Theorem applied):

  ┌─────────────────────┬──────────────────────────────────┐
  │ Service             │ CAP Choice                       │
  ├─────────────────────┼──────────────────────────────────┤
  │ Order Service       │ CP — accuracy > availability     │
  │ Availability/Browse │ AP — availability > accuracy     │
  │ Nearby/Location     │ AP — stale DC list is fine       │
  └─────────────────────┴──────────────────────────────────┘

  Order   = wrong inventory = lost money → MUST be consistent
  Browse  = slightly stale stock is OK   → MUST be available

----------------------------------------------------------------
11. SCALABILITY CONSIDERATIONS
----------------------------------------------------------------

HORIZONTAL SCALING:

  ┌────────┐ ┌────────┐ ┌────────┐
  │ Order  │ │ Order  │ │ Order  │   ← multiple instances
  │ Svc-1  │ │ Svc-2  │ │ Svc-3  │
  └────────┘ └────────┘ └────────┘
        │         │         │
        └─────────┴─────────┘
                  │
            Load Balancer

CACHING (Redis):
  - Hot items (top sellers) cached in Redis
  - Reduces DB load on Availability Service
  - Cache invalidated on inventory update

DATABASE SHARDING:
  - Shard by dc_id (region-based)
  - Each region keeps its own DB
  - Reduces cross-region queries

ASYNC PROCESSING (Kafka):

  Order placed (sync) → user gets confirmation
       │
       └──► Kafka queue (async)
              │
              ├──► Delivery dispatch
              ├──► Notifications
              ├──► Analytics
              └──► Inventory replenishment

----------------------------------------------------------------
12. PROS AND CONS OF THIS DESIGN
----------------------------------------------------------------

  ┌──────────────────────┬──────────────────────────────────┐
  │ PROS                 │ CONS                             │
  ├──────────────────────┼──────────────────────────────────┤
  │ Fast delivery (30min)│ Many small warehouses = expensive│
  │ Scales per region    │ Operational complexity is high   │
  │ Strong consistency   │ Cross-region orders are hard     │
  │   on orders          │                                  │
  │ Independent services │ Extra latency from gateway       │
  │ Real-time inventory  │ Inventory sync across DCs hard   │
  └──────────────────────┴──────────────────────────────────┘

----------------------------------------------------------------
13. FULL PICTURE — COMPLETE SYSTEM
----------------------------------------------------------------

  User (Mobile / Web)
       │
       │ HTTPS / REST
       ▼
  ┌─────────────────────────────┐
  │         API Gateway         │
  │                             │
  │  ┌───────────────────────┐  │
  │  │   Authentication      │  │
  │  ├───────────────────────┤  │
  │  │    Rate Limiting      │  │
  │  ├───────────────────────┤  │
  │  │   Load Balancing      │  │
  │  ├───────────────────────┤  │
  │  │   Request Routing     │  │
  │  └───────────────────────┘  │
  └─────────────────────────────┘
       │
       ├──► Location Service ────► DC_LOCATIONS DB
       │       (find nearby DCs)
       │
       ├──► Availability Service ─► Items + Inventory DB
       │       (check stock)
       │
       └──► Order Service ───────► PostgreSQL (ACID + Locks)
               (place order)         │
                                     ├─► Orders table
                                     ├─► Order_Items table
                                     └─► Inventory (LOCKED)
                                              │
                                              ▼
                                       Async via Kafka
                                              │
                                              ├─► Delivery dispatch
                                              ├─► Notifications
                                              └─► Analytics

----------------------------------------------------------------
14. KEY INTUITION SUMMARY
----------------------------------------------------------------

  Quick-commerce = "Amazon, but in 30 minutes"
  Key trick      = Small warehouses (dark stores) NEAR users

  3 CORE SERVICES:
    Nearby       → "Where is the closest DC?"
    Availability → "What's in stock there?"
    Order        → "Lock it and sell it atomically"

  4 CORE ENTITIES:
    Item, Inventory, Distribution Center, Order

  CRITICAL GUARANTEE:
    No overselling → enforced via DB transactions + locking

----------------------------------------------------------------
COMPONENT SUMMARY
----------------------------------------------------------------
  Component             Purpose
  ───────────────────── ─────────────────────────────────────────
  Client App            Mobile/Web interface for users
  API Gateway           Single entry, auth, rate limit, routing
  Location Service      Find nearest DCs using lat/long
  Availability Service  Check live stock at nearby DCs
  Order Service         Atomically place + confirm orders
  Distribution Center   Physical warehouse with local inventory
  Items Table           Catalog of products (abstract types)
  Inventory Table       Physical stock per DC (real instances)
  Orders Table          Customer order records
  Order_Items Table     Line items per order
  DC_Locations Table    Geospatial index of DCs
  PostgreSQL            ACID-compliant DB for transactions
  Redis Cache           Hot-item caching for fast reads
  Kafka                 Async queue for non-critical workflows
  Geospatial Index      Fast nearest-DC lookups (Haversine)
================================================================
```


