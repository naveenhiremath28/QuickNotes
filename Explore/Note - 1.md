How to design a system (High level)
1. List Functional requirements: System features/ functionalities
2. If required Out of scope requirements like dependency functionalities or not majorly mentioned in the requirements
3. List Non-functional requirements like availability, scalability, latency etc if required also mention out of scope requirements like fault tolerance, backups etc
4. List Core entities of the application eg. if we take leet code as application its core entities are user, problem, solution, leadboard etc
5. List high level api routes for the application
6. write high level flow, schemas


24/02/2026
https://bytebytego.com/guides/the-ultimate-api-learning-roadmap/
## 1️⃣ Public APIs (Open APIs)




**Definition:**  
APIs that are exposed to external developers or the public over the internet.

**Who can access?**  
Anyone (usually with an API key or OAuth token).

**Purpose:**  
To allow third-party developers to build apps, integrations, or services on top of your platform.

### Examples

- Google Maps API    
- Stripe Payments API
- Twitter API
### Characteristics

- Well-documented (OpenAPI/Swagger)
- Strong authentication (OAuth2, API keys, JWT)
- Rate-limited
- Versioned
- Public documentation portal
### Example Use Case

You build a payment service and expose:
POST /api/v1/payments
External merchants integrate it into their apps.

---
## 2️⃣ Private APIs (Internal APIs)

**Definition:**  
APIs used **only inside an organization**.

**Who can access?**  
Internal teams, internal microservices, internal tools.

**Purpose:**  
To connect internal systems and services.

### Examples

- Auth service calling User service
- Billing service calling Ledger service
- Internal admin dashboards

### Characteristics

- Not publicly documented
    
- Often protected by internal network (VPC)
    
- Can use internal auth (mTLS, internal tokens)
    
- Less strict rate limiting
    

### Example Use Case

Your microservice architecture:

User Service → calls → Token Service

Not exposed outside your infrastructure.

---

## 3️⃣ Partner APIs

**Definition:**  
APIs shared with **specific business partners**, not publicly open.

**Who can access?**  
Selected companies under contract.

**Purpose:**  
B2B integrations.

### Examples

- Amazon Partner API for sellers
    
- PayPal Merchant APIs
    
- Bank APIs shared with fintech partners
    

### Characteristics

- Access controlled via contract
    
- Strong authentication (mTLS, signed requests)
    
- SLA agreements
    
- Often IP whitelisted
    
- Custom onboarding process
    

### Example Use Case

You expose:

POST /api/v1/token/issue

Only approved financial institutions can access it.

---

# 🔎 Quick Comparison

| Feature        | Public API  | Private API            | Partner API       |
| -------------- | ----------- | ---------------------- | ----------------- |
| Audience       | Anyone      | Internal teams         | Selected partners |
| Documentation  | Public      | Internal only          | Restricted        |
| Security Level | High        | Internal network trust | Very High         |
| Example        | Google Maps | Internal Auth Service  | Bank ↔ Fintech    |

---
## Note:

### ✅ What is a **URI**?

**URI (Uniform Resource Identifier)** is a string that identifies a resource on the internet.

It can:

- 🔹 Identify a resource by **location**
    
- 🔹 Identify a resource by **name**
    
- 🔹 Or both
    

👉 Think of **URI as the parent concept**.

Example:

https://example.com/users/1  
mailto:naveen@example.com  
urn:isbn:0451450523

All of these are **URIs**.

---

### ✅ What is a **URL**?

**URL (Uniform Resource Locator)** is a type of URI that tells:

- 📍 **Where** the resource is
    
- 🌐 **How** to access it (protocol)
    

Example:

https://example.com/users/1

Breakdown:

- `https` → Protocol
    
- `example.com` → Domain
    
- `/users/1` → Path
    

A URL always includes a **location**.

---

### ✅ Simple Difference

|URI|URL|
|---|---|
|Identifies a resource|Identifies + Locates a resource|
|Can be name, location, or both|Always includes location|
|Broader concept|Subset of URI|

👉 **All URLs are URIs**  
👉 **Not all URIs are URLs**

---

### 🔥 Quick Analogy (Easy to Remember)

- **URI** = Person’s identity (Aadhaar / Passport / Name)
    
- **URL** = Person’s home address
    

Address tells you _where_ they live.  
Identity may not tell location.
## The 6 API Architecture Styles: REST, RESTful, GraphQL, SOAP, gRPC, WebSockets, and MQTT
- https://dev.to/ivannalon/the-6-api-architecture-styles-rest-restful-graphql-soap-grpc-websockets-and-mqtt-2a9h?utm_source=chatgpt.com

---


## 🔐 What Is API Authentication?

**API Authentication** is the process of verifying _who_ is making a request to your API before allowing access. It ensures only trusted users or systems can call your endpoints. Authentication confirms identity; _authorization_ then determines what that identity is allowed to do. 

---

## 1️⃣ **Basic Authentication**

**How it works:**  
A client sends a **username and password** with each API request in the HTTP `Authorization` header. The credentials are **Base64-encoded** but _not encrypted_. 

Authorization: Basic base64(username:password)

**Pros:**

- Simple and widely supported
    
- Easy to implement
    

**Cons:**

- Credentials are easily decoded unless HTTPS is used
    
- No built-in expiration or rotation
    
- Not suitable for production without TLS/HTTPS 
    

---

## 2️⃣ **Token-Based Authentication**

This is a broad category where the server issues a **token** after login, and the client sends that token with every API request.

**How it works:**

- Client logs in (e.g., with username/password)
    
- Server generates a token (e.g., JWT)
    
- Client includes token (usually in the `Authorization: Bearer` header) on future API calls 
    

**Pros:**

- Tokens can be stateless
    
- Efficient for APIs and mobile apps
    

**Cons:**

- Token storage must be secure
    
- Might need a refresh mechanism 
    

---

## 3️⃣ **JWTs (JSON Web Tokens)**

**What it is:**  
A **JWT** is a self-contained token format that holds _claims_ (like user identity and permissions). It’s digitally signed so the server can trust it without storing session data. 

Structure:

`<HEADER>.<PAYLOAD>.<SIGNATURE>`

**How it works:**

- After successful login, server issues a JWT
    
- Client sends it in request headers (`Authorization: Bearer <token>`)
    
- Server verifies the signature and claims to authenticate the user 
    

**Pros:**  
✔ Stateless (no server session storage)  
✔ Scales well for distributed systems  
✔ Portable between services

**Cons:**  
❌ Token revocation is harder  
❌ Must be carefully stored to avoid leaks 

---

## 4️⃣ **OAuth (OAuth 2.0)**

**What it is:**  
OAuth is a **protocol** for delegated access — _not just authentication_. It allows a user to grant a third-party app access to their resources without sharing passwords. 

**Key idea:**  
Instead of basic credentials, OAuth issues **access tokens** (and optionally **refresh tokens**) through defined flows like _authorization code_, _client credentials_, etc.

**How it works:**

- User grants permission to a client app
    
- App gets an OAuth access token from the authorization server
    
- App uses that token to call APIs on the user’s behalf 
    

OAuth is widely used for:

- Social logins (e.g., “Login with Google/Facebook”)
    
- Third-party API access
    
- Enterprise integrations 
    

**Pros:**  
✔ Secure delegated access  
✔ Supports scopes and fine-grained permissions

**Cons:**  
❌ More complex setup than basic auth or JWT 

---

## 5️⃣ **Session-Based Authentication**

**How it works:**  
This traditional method stores a **session on the server** after login. The server creates a _session ID_ and the client stores it (often in a cookie). The server validates this session ID with every request. 

**Pros:**  
✔ Easy to invalidate (logout)  
✔ Server controls session lifecycle  
✔ Great for web apps with cookies

**Cons:**  
❌ Requires server-side storage  
❌ Can be harder to scale across distributed APIs — requires shared session store 

---

## 🔎 Summary Comparison

|Method|Store State?|Best For|Notes|
|---|---|---|---|
|**Basic Auth**|No|Simple API clients|Unsafe without HTTPS|
|**Token Auth**|Optional|APIs, mobile apps|Flexible with many token types|
|**JWT**|No|Distributed & scalable APIs|Stateless but needs secure storage|
|**OAuth**|No (uses tokens)|Delegated access & third-party APIs|Powerful but complex|
|**Session Auth**|Yes|Traditional web apps|Easy to logout + control|

---
25/02/2026

## 🔹 **1. Pagination**

**What it is:**  
When an API returns a large list of data (like thousands of users), pagination _splits_ the data into smaller “pages” that the client can request one at a time. 

**Why it matters:**

- Reduces large payloads
    
- Improves performance
    
- Makes your API more responsive
    

**How it’s used:**  
Common query parameters: `page`, `limit`, `cursor`, `offset`  
Example:

GET /users?page=2&limit=50



## ✅ **What Is Idempotency?**

refer - https://restfulapi.net/idempotent-rest-apis/?utm_source=chatgpt.com

**Idempotency** is a property of an API operation meaning that **making the _same_ request multiple times has the _same effect_ as making it once** — no matter how many times it’s repeated. This ensures the _state of the system after the first successful request_ does not change on subsequent identical requests. 

> In simple terms: if you send the same API call 1 time or 10 times, the outcome is the same.

## 🔹 Why Do We Need Idempotency?

Imagine:

- You call `POST /create-payment`
    
- Network timeout happens
    
- You don’t know if payment was created
    
- So you retry the request
    

Without idempotency ❌  
→ You might create **two payments**

With idempotency ✅  
→ The server understands it’s the **same request** and returns the **same response**, without creating a duplicate payment.

---

## 🔹 Real-World Example

Think about:

- UPI payment
    
- Online shopping order
    
- Bank transfer
    
- Token minting (like your UNITS transact APIs)
    

If the client retries due to timeout, idempotency ensures:

- Only **one transaction happens**
    
- But client can retry safely
    

---

## 🔹 How It Works (Simple Flow)

1. Client sends request with an **Idempotency-Key**
    

POST /payments  
Idempotency-Key: 12345-abc

2. Server checks:
    
    - If this key was **never used before**  
        → Process request  
        → Store response with this key
        
    - If this key **already exists**  
        → Return stored response  
        → Do NOT process again


## **HTTP** 
## 🧪 **HEAD Method**

**What it is:**  
The **HEAD** method is essentially the same as a `GET` request, **but the server returns only the response _headers_** — _no response body_. It’s like asking, _“What would I get if I did a GET, but don’t actually send the data?”_

### How it works

- You send a request like:
    
    HEAD /api/users/123 HTTP/1.1
    
- Server responds with headers such as:
    
    HTTP/1.1 200 OK  
    Content-Length: 1270  
    Last-Modified: Tue, 10 Feb 2026 08:00:00 GMT
    
    but **no body content**. 
    

### Why it’s useful

- 🔍 **Check resource metadata** before doing a full `GET`  
    — e.g., see its size (`Content-Length`) or last update time (`Last-Modified`). 
    
- 💾 **Save bandwidth** when you only care about the headers, not the data. 
    
- 📌 Often used in caching or conditional requests, where you want to decide if a resource changed without downloading it. 
    

### Properties

- **Idempotent** — calling it many times doesn’t change server state. 
    
- **Safe** — it should not affect the resource. 
    

---

## 🛠️ **OPTIONS Method**

**What it is:**  
The **OPTIONS** method asks the server: _“What HTTP methods are allowed on this resource?”_ It doesn’t return the resource itself — just information about which operations you can perform. 

### How it works

- You send:
    
    OPTIONS /api/users HTTP/1.1
    
- Server responds with headers like:
    
    Allow: GET, POST, PUT, DELETE, OPTIONS
    
    indicating which methods are supported. 
    

### Why it’s useful

- 📋 **Discover API capabilities** — even with no documentation, you can see what is allowed on an endpoint. 
    
- 🌐 **CORS Preflight Responses** — browsers use `OPTIONS` to check permissions (allowed methods/headers) before making cross-origin requests. 
    

### Properties

- **Safe** — it should not alter resource state. 
    
- Often used in middleware or API tools to inspect endpoints.



## 🚀 What Is API Performance?

**API performance** refers to how fast, reliable, and efficient an API responds to client requests — including its ability to handle lots of traffic without slowing down or failing. Good performance leads to better user experience and system reliability. 

---

## 🛠️ Key API Performance Techniques

### ⭐ 1. **Caching**

**What it is:**  
Caching stores frequently requested data in a fast-access layer (memory, CDN, or edge cache) so repeated requests return quickly without re-computing or re-fetching from the backend. 

**Why it matters:**

- Reduces load on servers and databases
    
- Improves response times drastically
    
- Helps scale under heavy traffic
    

**How it’s used:**

- HTTP caching headers (`Cache-Control`, `ETag`)
    
- In-memory caches like Redis or Memcached
    
- CDNs (Content Delivery Networks) for global caching 
    

---

### 🔐 2. **Rate Limiting**

**What it is:**  
Rate limiting controls how many requests a client can make in a given time window. It helps prevent abuse and ensures fair use of API resources. 

**Why it matters:**

- Protects APIs from overload and denial-of-service attacks
    
- Ensures stable performance during traffic spikes
    
- Improves reliability for all users
    

**Examples:**  
Limits per minute/hour per user, IP address, API key, etc. 

---

### 🔄 3. **Load Balancing**

**What it is:**  
Load balancing distributes incoming requests evenly across multiple servers or instances so no single server is overwhelmed. 

**Why it matters:**

- Improves availability and uptime
    
- Increases throughput (requests handled per second)
    
- Avoids single points of failure
    

It’s often used with auto-scaling in cloud environments to add/remove servers based on load. 

---

### 📄 4. **Pagination**

**What it is:**  
Instead of returning large lists of data in one response, pagination breaks results into pages (small chunks). 

**Why it matters:**

- Reduces response size and memory use
    
- Faster data transfer
    
- Better for client & server scalability
    

**Typical parameters:** `page`, `limit`, `cursor`, etc. 

---

### 🗃️ 5. **Database Indexing**

**What it is:**  
Indexing tells the database how to find rows more efficiently — like a table of contents. 

**Why it matters:**

- Speeds up data retrieval for queries used by your API
    
- Reduces database CPU usage
    
- Improves throughput under heavy load
    

Without indexes, the database might scan entire tables, which is slow. 

---

### 📈 6. **Scaling**

**What it is:**  
Scaling adds more computing power or capacity to handle increased load.

**Two approaches:**

- **Vertical scaling:** Bigger server (more CPU/RAM)
    
- **Horizontal scaling:** More servers/instances behind load balancers
    

**Why it matters:**

- Lets your API handle more simultaneous users
    
- Helps maintain performance as demand grows 
    

Cloud platforms (AWS, Azure, GCP) make horizontal scaling easier with auto-scaling groups that add/remove nodes based on traffic.

---

### 🧪 7. **Performance Testing**

**What it is:**  
Performance testing simulates real-world load on your API to measure performance metrics such as latency, throughput, error rates, and scalability. 

**Types of tests:**

- **Load testing:** Normal peak user traffic
    
- **Stress testing:** Extreme traffic beyond expected limits
    
- **Soak testing:** Long-duration heavy usage
    

**Why it matters:**

- Reveals bottlenecks before production issues
    
- Helps plan scaling & resource allocation
    
- Improves reliability and user experience


# 🔹 What is RESTful API?

A **RESTful API** is simply:

👉 An API that **properly follows REST principles**.

So:

- REST = the concept
    
- RESTful = implementation that follows the concept
    

Example:

If you build:

POST /users

to create a user — that's RESTful.

But if you build:

POST /createUser

that is NOT strictly RESTful.

---

# 🔥 Real Example (Production Style)

Imagine you are building your Finternet token service.

Resources could be:

/accounts  
/tokens  
/transactions

Operations:

POST   /accounts  
GET    /accounts/{id}  
POST   /transactions  
GET    /transactions/{id}

That is RESTful design.

---

# 🔹 REST vs RESTful (Simple Difference)

| Term        | Meaning                     |
| ----------- | --------------------------- |
| REST        | Architectural style         |
| RESTful API | API that follows REST rules |

## **Types of Testing:**

- **Smoke Testing** This is done after API development is complete. Simply validate if the APIs are working and nothing breaks.
- **Functional Testing** This creates a test plan based on the functional requirements and compares the results with the expected results.
- **Integration Testing** This test combines several API calls to perform end-to-end tests. The intra-service communications and data transmissions are tested.
- **Regression Testing** This test ensures that bug fixes or new features shouldn’t break the existing behaviors of APIs.
- **Load Testing** This tests applications’ performance by simulating different loads. Then we can calculate the capacity of the application.
- **Stress Testing** We deliberately create high loads to the APIs and test if the APIs are able to function normally.
- **Security Testing** This tests the APIs against all possible external threats.
- **UI Testing** This tests the UI interactions with the APIs to make sure the data can be displayed properly.
- **Fuzz Testing** This injects invalid or unexpected input data into the API and tries to crash the API. In this way, it identifies the API vulnerabilities.



## Web3

**Web3** refers to the next generation of the internet built on **blockchain technology**, aiming to make the web more **decentralized, transparent, and user-owned**.

Here’s a simple breakdown:

### 🔹 What is Web3?

Web3 is an evolution of the internet:

- **Web1 (1990s–early 2000s):** Read-only (static websites)
- **Web2 (current):** Read + write (social media, apps, big tech control)
- **Web3:** Read + write + **own** (users control their data and assets)

---

### 🔹 Key Features

- **Decentralization:** No single company controls the system (uses blockchain)
- **Ownership:** You own your digital assets (crypto, NFTs)
- **Trustless systems:** No need for intermediaries (banks, platforms)
- **Smart contracts:** Code that runs automatically on blockchain

---

### 🔹 Examples of Web3

- Cryptocurrencies (Bitcoin, Ethereum)
- NFTs (digital ownership of art, items)
- DeFi (decentralized finance apps)
- DAOs (community-run organizations)

---

### 🔹 Why it matters

- More privacy and control for users
- Less reliance on big tech companies
- New ways to earn, invest, and build online

---

### 🔹 Downsides / Challenges

- Still complex for beginners
- Security risks (scams, hacks)
- Regulation is unclear
- Scalability issues




Advantages
* Anonymous signon
* self governing
* better search
* Personalized browsing experience (ads)
* uninterrupted services
* User will also receive some tokens when he saw some add (because he invested his time) in brave


cons
* regulation issues





---

# 🧱 1. ACID (Traditional Databases – Strong Consistency)

ACID ensures **reliability and correctness** of transactions.

### 🔹 A – Atomicity

- “All or nothing”
- Example: Bank transfer
    - Debit succeeds but credit fails → rollback everything

### 🔹 C – Consistency

- Database always stays in a **valid state**
- Rules/constraints are never violated

### 🔹 I – Isolation

- Transactions don’t interfere
- Parallel queries behave like they ran one-by-one

### 🔹 D – Durability

- Once committed → **never lost** (even after crash)

### 👉 Example

In a payment system:

Transfer ₹100 from A → B

Either:

- Both debit & credit happen ✅  
    OR
- Nothing happens ❌

👉 Used in:

- PostgreSQL
- MySQL
- Oracle

---

# ⚡ 2. BASE (NoSQL / High Scalability Systems)

BASE is more relaxed than ACID.

### 🔹 BA – Basically Available

- System always responds (even if data is not perfect)

### 🔹 S – Soft State

- Data can change over time (eventual updates)

### 🔹 E – Eventually Consistent

- Data becomes consistent **later**

---

### 👉 Example

Think of:

- Instagram likes 👍
- You like a post → count may not update instantly everywhere

Eventually → all nodes sync ✔️

---

### 👉 Used in:

- Cassandra
- DynamoDB
- MongoDB (in some modes)

---

# ⚖️ ACID vs BASE (Quick Comparison)

|Feature|ACID 🧱|BASE ⚡|
|---|---|---|
|Consistency|Strong|Eventual|
|Availability|Lower|High|
|Performance|Slower|Faster|
|Use case|Banking, finance|Social media, analytics|

---

# 🌍 3. Distributed Databases (Example: Yugabyte)

Now comes the interesting part.

### 👉 What is a Distributed DB?

- Data is stored across **multiple machines (nodes)**
- Still behaves like a **single database**

---

## 🔹 Example: Yugabyte

Yugabyte tries to give:  
👉 **ACID + Scalability (like BASE systems)**

---

### 🧠 How it works

#### 1. Data Sharding

- Data split into pieces (shards)
- Stored across multiple nodes

#### 2. Replication

- Each shard has copies (replicas)
- Ensures fault tolerance

#### 3. Consensus (Raft)

- Nodes agree before committing data
- Ensures **consistency (ACID-like)**

---

### 👉 Example Scenario

You have an app running in:

- India 🇮🇳
- US 🇺🇸
- Europe 🇪🇺

User in India writes data:

👉 Yugabyte:

1. Writes to nearest node
2. Replicates to others
3. Uses consensus → ensures correctness

Even if one node crashes:  
👉 System still works ✅

---

### 🔥 Why Yugabyte is powerful

- Distributed like NoSQL
- SQL support like PostgreSQL
- Strong consistency (ACID)
- High availability

👉 It basically solves:

> “How do we scale without losing correctness?”

---

# 🧠 Final Intuition

- **ACID** → correctness first
- **BASE** → scalability & availability first
- **Distributed DB (Yugabyte)** → tries to balance both

---

# 🚀 Simple Analogy

- **ACID** = Bank system (strict, safe)
- **BASE** = Social media (fast, flexible)
- **Distributed DB** = Global bank (safe + scalable)


## Overrides in Node
```
The simplest possible explanation

  Set the scene

  Your app uses a logging library called HyperDX. HyperDX itself uses a chain of helper libraries:

  HyperDX → OTel-detector → GCP-helper → gaxios → uuid

  The problem: the uuid library at the very end has a security bug. It's fixed in version 14, but your app has version 9.

  The intuition that's wrong

  ▎ "Just install uuid@14 in my app's package.json — npm will use that everywhere, right?"

  No, it won't. Here's why.

  Think of node_modules like nested folders

  When you install npm packages, they sit in folders inside folders, like this:

  your-app/
  ├── node_modules/
  │   ├── hyperdx/
  │   │   └── node_modules/
  │   │       └── otel-detector/
  │   │           └── node_modules/
  │   │               └── gcp-helper/
  │   │                   └── node_modules/
  │   │                       └── gaxios/
  │   │                           └── node_modules/
  │   │                               └── uuid/  ← version 9 (THE BAD ONE)
  │   └── uuid/  ← if you install at top-level, uuid@14 sits HERE

  Now picture a gaxios file that says const uuid = require("uuid"). When Node looks for "uuid", it works like a person
  searching for car keys:

  ▎ "Did I leave them on this table? No.
  ▎ How about the table one room over? No.
  ▎ The kitchen table? Yes — found them!"

  It starts in the closest folder and walks outward until it finds something. The moment it finds a uuid folder, it stops.

  So when gaxios asks for "uuid":

  1. Look in gaxios/node_modules/uuid/ → found! version 9. ✅ Stop.

  It never even looks at the top-level your-app/node_modules/uuid/ where your shiny new uuid@14 is sitting. From gaxios's
  perspective, your version-14 doesn't exist.

  Why doesn't npm just merge them?

  Sometimes npm does merge ("dedupe") and put a single copy at the top. But only when versions are compatible.

  Each library declares what versions of its dependencies are acceptable. gaxios says:

  ▎ "I need uuid version 9-something. Anything 9.0.1 or higher in the 9.x family is fine."

  If you install uuid@9.5 at the top level, npm thinks: "great, 9.5 satisfies gaxios's request, I'll just use one copy." → One
  uuid, dedupe happens, everyone shares.

  If you install uuid@14 at the top level, npm thinks: "14 is not in the 9.x family, gaxios won't accept this." → Two copies,
  gaxios keeps its own version 9.
  uuid, dedupe happens, everyone shares.

  If you install uuid@14 at the top level, npm thinks: "14 is not in the 9.x family, gaxios won't accept this." → Two copies,
  gaxios keeps its own version 9.

  Now extend the same logic up the whole chain

  To get uuid@14 to actually be used by gaxios, you'd need gaxios to say "I'll accept uuid 14." The newer gaxios@7 does say
  that — it dropped uuid entirely. Great, so install gaxios@7 at the top, right?

  Same problem one level up:
  - gcp-helper declares: "I need gaxios in the 6.x family."
  - gaxios@7 isn't in 6.x → npm installs both → gcp-helper still uses gaxios@6 → which uses uuid@9.

  And one level above that:
  - otel-detector declares: "I need gcp-helper in the 6.x family."
  - ...same problem...

  And at the top:
  - HyperDX declares: "I need otel-detector in the 0.56.x family."
  - The fixed otel-detector is 0.74.x → outside 0.56.x → can't propagate.

  Every link in the chain blocks the upgrade. You'd have to fix the topmost one (HyperDX) to widen its acceptance, and then
  each lower one would naturally get newer versions.

  So what can you do?

  Three real options, each with a tradeoff:

  1. overrides (the npm sledgehammer)

  Tell npm: "I don't care what gaxios says it accepts. Force uuid@14 in there anyway."
  It works. The downside is npm ls will say "this is technically invalid" — kind of like getting your car serviced with a part
  the manual didn't bless. Probably fine, but a strict mechanic would frown.

  2. Wait for HyperDX to update

  HyperDX's maintainers ship a new version that accepts the newer detector. Then everything cascades naturally. No frowns from
  npm. But you wait however long they take.

  3. Stop using HyperDX

  Replace the HyperDX library with the underlying OpenTelemetry libraries directly. Then you control the whole chain. But that
  means editing your code (the two files where you require('@hyperdx/...')).

  TL;DR

  You can't fix uuid by adding it to your package.json because Node looks for libraries in the closest folder first, and
  there's already an old uuid sitting in a closer folder than yours will ever be.

  The only ways out are: force it (overrides), wait for upstream to bump, or remove the package that's blocking the chain.
```