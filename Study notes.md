
1. Yugabyte
2. ORM - GORM


---
## ORM
 
**ORM** stands for **Object–Relational Mapping**.

It’s a technique that lets you work with a **database using objects and classes** in your programming language instead of writing raw SQL queries.

- **Database table** → becomes a **class**
    
- **Table row** → becomes an **object**
    
- **Table column** → becomes an **object field**
    

You interact with objects, and the ORM takes care of SQL behind the scenes.
### Example (without ORM vs with ORM)

#### ❌ Without ORM (raw SQL)

`SELECT id, name, email FROM users WHERE id = 1;`

#### ✅ With ORM (conceptually)

`user := db.First(&User{}, 1)`

You don’t write SQL—the ORM generates it for you.

### Why ORMs are used

- ✅ Less SQL to write
    
- ✅ Cleaner, readable code
    
- ✅ Easier CRUD operations (Create, Read, Update, Delete)
    
- ✅ Database-agnostic (can switch DBs with minimal changes)
    
- ✅ Built-in validation & relationships
  
### One real-world analogy

Think of ORM like a **translator**:

- You speak **Go / Python / Java**
    
- Database speaks **SQL**
    
- ORM translates between them automatically

---
## MPC Protocol

**MPC (Multi-Party Computation) protocols** are cryptographic techniques that let **multiple parties jointly generate, use, and rotate cryptographic keys** **without ever reconstructing the full private key in one place**. Each party holds only a _share_ of the key, and sensitive operations are done collaboratively.

## 1️⃣ MPC for **Key Generation**

**Problem:** Creating a private key on one machine is risky (single point of compromise).

**MPC solution:**

- Multiple parties (e.g., devices, servers, HSMs) **jointly generate** a key.
    
- Each party ends up with a **key share**.
    
- The **full private key never exists**, not even temporarily.


**Result:**

- No key export
    
- No “master key” to steal
    
- Safe even if one participant is compromised
    

---

## 2️⃣ MPC for **Signing**

**Problem:** Traditional signing requires access to the full private key.

**MPC solution:**

- Each party computes a **partial signature** using its key share.
    
- Partial signatures are **mathematically combined**.
    
- The final signature is **identical** to a normal signature (ECDSA/EdDSA/RSA).
    

**Important:**

- No party learns anything about the other shares
    
- The private key is never reconstructed
    

**To the blockchain or verifier:**  
👉 It looks like a normal signature — MPC is invisible externally.

---

## 3️⃣ MPC for **Key Rotation**

**Problem:** Rotating keys usually means generating and moving new private keys.

**MPC solution:**

- Parties **reshare** or **refresh** key shares.
    
- Cryptographic material changes, but:
    
    - Public key stays the same (optional)
        
    - Or a new public key is derived safely
        
- Old shares become useless.
    

**Benefits:**

- Zero downtime
    
- No key exposure
    
- Ideal for long-lived systems
    

---

## Why MPC is powerful

|Traditional keys|MPC keys|
|---|---|
|Single private key|Split into shares|
|Stored in one place|Distributed|
|High blast radius|Limited compromise|
|Hard rotation|Safe & seamless rotation|

---

## Common MPC models

- **t-of-n threshold**  
    Example: 2 of 3 parties must participate to sign
    
- **Device + server**
    
- **Multi-region servers**
    
- **Human + machine approvals**
    

---

## Where MPC is used today

- Crypto wallets & custody
    
- Enterprise key management
    
- Secure authentication systems
    
- Blockchain validators
    
- Financial infrastructure (HSM alternative)
    

---

## MPC vs Encryption (important distinction)

- **Encryption:** protects stored data
    
- **MPC:** protects _control_ of cryptographic power  
    👉 Even insiders cannot misuse keys alone
    

---

## One-sentence summary

**MPC protocols allow keys to be generated, used, and rotated collaboratively — without ever creating or exposing the private key — eliminating single points of failure.**

---

## ACID Transactions

## 🔹 What is a Transaction?

A **transaction** is a group of database operations treated as **one single unit of work**.

Example:

`1. Deduct ₹500 from Account A 2. Add ₹500 to Account B`

Both must succeed together — or none should.

![[Pasted image 20260106144823.png]]

![[Pasted image 20260106150950.png]]
## 🔹 BASE (High Availability Model)

**BASE = Basically Available, Soft state, Eventually consistent**

### What it guarantees

- System stays **available even during failures**
    
- Data may be **temporarily inconsistent**
    
- Eventually, all replicas **become consistent**
    

📌 Example:

- Like counts on posts
    
- Comments
    
- Product recommendations
    

💡 Trade-off:

- Faster
    
- Scales very well
    
- Temporary inconsistency is acceptable
    

---

## 🧠 BASE Explained Simply

### 1️⃣ **Basically Available**

- System responds **even if some nodes are down**

### 2️⃣ **Soft State**

- Data can change **without new input**
    
- Replicas sync in background
    

### 3️⃣ **Eventual Consistency**

- If no new updates occur, all replicas **eventually match**

## Example We’ll Use

**Instagram-like app**

- You like a post
    
- The post is stored in **multiple servers (replicas)** around the world

---

## 1️⃣ Basically Available — “You Always Get a Response”

❌ **Not BASE**:

> “Server is down, try again later”

✅ **BASE**:

> “Your request is accepted, here’s a response”

📌 In our example:

- You tap ❤️ Like
    
- Even if some servers are down → app still works
    

💡 Meaning:

- System prefers **availability over perfect accuracy**
    
- It never blocks users
    

---

## 2️⃣ Soft State — “Data Is Not Fixed Immediately”

This is the **most confusing part**, so read carefully 👇

### What “Soft State” means

> The data **can change later**, even if no new request comes in.

📌 Example:

- Server A: Like count = **101**
    
- Server B: Like count = **102**
    

Both are **temporarily okay**.

After some time:

- Background sync runs
    
- Both servers update to **102**
    

💡 Why “soft”?

- State is **not final**
    
- It’s allowed to change automatically
    

---

## 3️⃣ Eventually Consistent — “All Will Agree Later”

This is the **promise BASE makes**.

> If no new updates happen, **all copies will become the same**

📌 Example:

- Right now:
    
    - You see **101 likes**
        
    - Your friend sees **102 likes**
        
- After a few seconds:
    
    - Everyone sees **102 likes**
        

✔️ No permanent wrong data  
✔️ Just delayed agreement

---
## Distributed SQL Database

### Simple definition

A **Distributed SQL database** is a database that:

- Uses **SQL**
    
- Looks like **one database**
    
- But runs across **multiple machines (nodes)**
    

👉 You get **SQL + ACID transactions + horizontal scalability**.

## Break it down

### 1️⃣ **Distributed**

- Data is spread across many nodes
    
- Each node stores **part of the data**
    
- Nodes replicate data for fault tolerance
    

### 2️⃣ **SQL**

- Supports standard SQL (tables, joins, indexes)
    
- Compatible with relational schemas
    
- Often PostgreSQL-compatible
    

### 3️⃣ **Database**

- Strong consistency
    
- ACID transactions
    
- Automatic failover
## Traditional DB vs Distributed SQL

| Feature           | Traditional SQL (Postgres) | Distributed SQL         |
| ----------------- | -------------------------- | ----------------------- |
| Runs on           | Single machine             | Many machines           |
| Scaling           | Vertical (bigger server)   | Horizontal (more nodes) |
| Failure handling  | Manual failover            | Automatic               |
| SQL support       | Full                       | Full                    |
| ACID transactions | ✅                          | ✅                       |

1. **Data is sharded**
    
    - Tables are split into chunks
        
2. **Replicated**
    
    - Each shard exists on multiple nodes
        
3. **Consensus (Raft)**
    
    - Majority agrees before committing data
        
4. **SQL layer**
    
    - You query like normal SQL
        
    - DB figures out where the data lives

---

## Why Distributed SQL was created

Old choice:

- **SQL** → strong consistency, poor scaling
    
- **NoSQL** → great scaling, weak consistency
    

Distributed SQL gives **both**:

> **SQL correctness + NoSQL scalability**

---
## What is NoSQL?

**NoSQL** (often read as _“Not Only SQL”_) refers to a class of databases designed to store and process **large volumes of data** with **flexible schemas**, **high scalability**, and **high availability**.  
Unlike traditional relational (SQL) databases, NoSQL databases do **not rely on fixed tables, rows, and columns**.
## Why NoSQL exists (the problem it solves)

Traditional SQL databases are great for:

- Structured data
    
- Strong consistency
    
- Complex joins and transactions
    

But they struggle when:

- Data structure changes frequently
    
- Data volume grows very large
    
- You need to scale horizontally (across many servers)
    
- You need very high read/write throughput
    

**NoSQL was created to solve these problems.**


## Key characteristics of NoSQL

### 1. Schema-less or flexible schema

You don’t need to define a rigid structure upfront.

`{   "id": 1,   "name": "Naveen",   "skills": ["Go", "React"] }`

Another record can look different:

`{   "id": 2,   "name": "Rahul",   "experience": 5 }`

✔ Both are valid.

---

### 2. Horizontal scalability

NoSQL databases are built to **scale out**, not just up.

- Add more machines → more capacity
    
- Uses **sharding** and **replication**
    

---

### 3. High performance

Optimized for:

- Fast reads/writes
    
- Massive concurrent users
    
- Distributed systems
    

---

### 4. BASE instead of ACID (often)

Many NoSQL systems follow **BASE**:

- **Basically Available**
    
- **Soft State**
    
- **Eventually Consistent**
    

(Though modern NoSQL DBs can also support strong consistency when needed.)

---
**Types of NoSQL databases**

### 1. Key–Value Stores

Data is stored as a simple **key → value** pair.

Example:

`"user:101" → "{name: Naveen, age: 25}"`

**Use cases**

- Caching
    
- Session storage
    
- Real-time data
    

**Examples**

- Redis
    
- Amazon DynamoDB
    

---

### 2. Document Databases

Data stored as **JSON/BSON documents**.

**Use cases**

- User profiles
    
- Content management
    
- APIs & microservices
    

**Example**

- MongoDB
    

---

### 3. Column-Family Stores

Data stored in **columns instead of rows**.

**Use cases**

- Analytics
    
- Time-series data
    
- Large-scale writes
    

**Examples**

- Apache Cassandra
    
- HBase
    

---

### 4. Graph Databases

Data stored as **nodes and relationships**.

**Use cases**

- Social networks
    
- Recommendation engines
    
- Fraud detection
    

**Example**

- Neo4j
    

---

## NoSQL vs SQL (quick comparison)

|Feature|SQL|NoSQL|
|---|---|---|
|Schema|Fixed|Flexible|
|Scaling|Vertical|Horizontal|
|Transactions|Strong ACID|BASE / tunable|
|Joins|Supported|Limited / none|
|Best for|Financial systems|Big data, real-time apps|

---

## When should you use NoSQL?

Use NoSQL when:

- Data structure changes frequently
    
- You expect **huge traffic**
    
- You need **low latency**
    
- You are building **distributed systems**
    
- You want easy horizontal scaling
    

---

## When NOT to use NoSQL?

Avoid NoSQL when:

- You need complex joins
    
- Strong transactional guarantees are mandatory
    
- Data relationships are highly structured
    
- Financial or banking-grade consistency is required
    

---

## Real-world examples

- **WhatsApp** → Messages storage
    
- **Netflix** → Recommendations & metadata
    
- **Uber** → Real-time tracking
    
- **Amazon** → Product catalogs & sessions

**NoSQL allows records to have different fields, which makes it flexible and scalable, but structure is still enforced by application logic, not the database.**

