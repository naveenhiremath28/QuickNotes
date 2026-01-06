
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

## Distributed SQL Database

