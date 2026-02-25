
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

|Feature|Public API|Private API|Partner API|
|---|---|---|---|
|Audience|Anyone|Internal teams|Selected partners|
|Documentation|Public|Internal only|Restricted|
|Security Level|High|Internal network trust|Very High|
|Example|Google Maps|Internal Auth Service|Bank ↔ Fintech|

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
