
## 🔐 What **OAuth** Is

**OAuth** stands for **“Open Authorization.”** It’s an **open standard protocol** designed for **authorization and access delegation** — letting one application access a user’s data from another service **without the user sharing their password**with that application. 

### Key Idea

Imagine you want a blogging platform to access your photos stored on another service — but you don’t want to give that app your credentials. OAuth lets you grant _limited access_ (like “read photos”) without sharing your username/password. 

### What OAuth Actually Does

- It **delegates access** to user resources: the user (resource owner) grants permission to a third-party app (client) to access data held by another server (resource server). 
    
- It issues **access tokens**, which clients use to request protected resources instead of credentials. 
    
- It’s about **authorization**, not authentication (it doesn’t verify identity by itself). 
    

---

## ⚙️ What **OAuth 2.0** Is

**OAuth 2.0** is the **second and most widely used version** of OAuth. It’s defined as an **authorization framework** that doesn’t dictate a single “one-size-fits-all” implementation — instead, it offers a set of **flows (grant types)** for different use cases (web apps, mobile apps, APIs, etc.). 

### Main Features of OAuth 2.0

✔ **Token-based access**  
Clients get **access tokens** (and optionally **refresh tokens**) to interact with protected resources. 

✔ **Multiple authorization flows**  
Different approaches (like _authorization code_ or _client credentials_) are used depending on the type of client. 

✔ **Clear separation of roles**  
Modern OAuth 2.0 clearly separates:

- **Resource Owner** — user who owns the data
    
- **Client** — app requesting access
    
- **Authorization Server** — issues tokens
    
- **Resource Server** — holds data and validates tokens 
    

✔ **Security via tokens and HTTPS**  
OAuth 2.0 avoids requiring clients to sign every request (unlike older versions) and instead relies on HTTPS security and short-lived tokens. 

---

## 📌 OAuth 1.0 vs OAuth 2.0 – What’s Different?

OAuth 2.0 isn’t just a small update — it’s a **major redesign** of the original OAuth:

### 🔹 Protocol Complexity

- **OAuth 1.0** used complex cryptographic signatures to secure every request.
    
- **OAuth 2.0** simplifies this by relying on HTTPS and tokens, making it easier to implement. 
    

### 🔹 Token Management

- OAuth 2.0 introduces **short-lived access tokens** and optional **refresh tokens** for better security and flexibility. 
    

### 🔹 Roles and Structure

- OAuth 1 didn’t clearly separate the roles of resource server and authorization server.
    
- OAuth 2.0 formalizes these roles for clarity and modular implementations. 
    

### 🔹 Use Cases

- OAuth 2.0 supports a wider range of application types (mobile, web, IoT) with dedicated flows. 
    

---

## ❗ Important Clarification: **Authorization ≠ Authentication**

Even though many services use OAuth 2.0 during login screens (“Sign in with Google/Facebook”), OAuth itself is **not an authentication protocol** — it only gives access to resources. Verification of identity comes from other layers like **OpenID Connect (OIDC)**, which is built on top of OAuth 2.0. 

---

## 🧠 Summary

| Concept            | What It Is                                                                                           |
| ------------------ | ---------------------------------------------------------------------------------------------------- |
| **OAuth**          | Open standard for **authorizing** third-party access without passwords.                              |
| **OAuth 2.0**      | Modern, widely adopted **authorization framework** with flexible grant flows and token-based access. |
| **Authentication** | Not handled by OAuth — use OIDC or similar for identity verification.                                |
## 🧠 Core Difference Overview

|Aspect|**OAuth (OAuth 1.0)**|**OAuth 2.0**|
|---|---|---|
|**Version**|Original version of OAuth|Redesigned, not backward compatible with 1.0|
|**Security Mechanism**|Requires cryptographic signatures on each request|Uses HTTPS/TLS instead of signatures for security|
|**Complexity**|More complex to implement due to signatures|Simpler and more developer-friendly|
|**Token Lifespan**|Tokens often long-lived|Access tokens are short-lived and often paired with refresh tokens|
|**Flexibility**|Limited authorization flows|Multiple flows (grant types) for different app types|
|**Roles Defined**|Fewer explicit roles; no separation of auth and resource servers|Clear separation: resource owner, client, auth server, resource server|
|**Use Case Suitability**|Harder for mobile/non-browser clients|Designed for web, mobile, API, desktop apps|
|**Backward Compatibility**|N/A|**Not backward compatible with OAuth 1.0**|

---

## 📌 What Changed Between OAuth 1.0 and OAuth 2.0

### 🔐 **Security and Implementation**

- **OAuth 1.0** relies on signing every API request using complex cryptographic operations. 
    
- **OAuth 2.0** simplifies this by using standard **HTTPS/TLS**, eliminating the need for request signatures. 
    

### ⚙️ **Token Handling**

- OAuth 1.0 typically issued longer-lived tokens. 
    
- OAuth 2.0 uses **short-lived access tokens** and **refresh tokens** to improve security and reduce risk if a token is leaked. 
    

### 🧩 **Flexibility & Flows**

- OAuth 1.0 had a **single standard flow**, mainly for web apps. 
    
- OAuth 2.0 supports multiple **grant types/flows** (e.g., authorization code, implicit, client credentials, device code), making it adaptable to many environments. 
    

### 🏛️ **Roles and Architecture**

- OAuth 1.0 did not explicitly separate **resource server** and **authorization server** roles. 
    
- OAuth 2.0 defines clear roles that help scale services across distributed systems. 
    

---

## 🧾 **Summary**

- **OAuth 1.0** is the _older, more complex protocol_ with cryptographic signatures and limited flexibility. 
    
- **OAuth 2.0** is a _modern, flexible authorization framework_ designed to be easier for developers, tailored for many client types, and more scalable — but it relies on HTTPS for security and does not maintain backward compatibility with OAuth 1.0.

## 🔹 **OAuth 1.0 / OAuth 1.0a (Legacy Example)**

### 📌 Example: **Twitter API with OAuth 1.0a**

Many older or _legacy_ APIs used OAuth 1.0a (a more secure revision of 1.0) to let applications access a user’s account without seeing their password. Twitter is the classic example. 

#### **Typical flow (3-legged OAuth):**

1. **Get a Request Token**  
    Your app calls Twitter’s `POST oauth/request_token` endpoint to get a temporary credential. 
    
2. **Redirect User to Authorize**  
    You redirect the user to Twitter’s `oauth/authorize` page with the request token. The user logs in to Twitter and approves your app. 
    
3. **Exchange for Access Token**  
    After the user approves, you exchange the authorized request token for an **access token + secret** using `POST oauth/access_token`. 
    
4. **Make Signed API Calls**  
    With the access token _and secret_, you can sign each API request (e.g., to post a Tweet) using a signature algorithm like `HMAC-SHA1`. 
    

👉 This flow involves cryptographic signing of requests and secrets, which is one reason OAuth 1.0 was considered _complex_ and largely replaced. 

**Usage in Practice:**

- **Twitter “Sign in with Twitter”** used OAuth 1.0a. 
    
- Other older services historically supported OAuth 1.0a too (e.g., some Jira or legacy APIs). 
    

---

## 🔸 **OAuth 2.0 (Modern Real-World Examples)**

### 📌 Example 1: **“Sign in with Google” / Google APIs**

OAuth 2.0 is widely used by Google to allow apps to get limited access to user resources (e.g., Google Drive files) or to sign users in without storing their password. 

#### Typical OAuth 2.0 Authorization Code Flow:

1. **Redirect to Google Consent**  
    Your app sends the user to Google’s authorization endpoint with its _client ID_, requested _scopes_, and _redirect URI_. 
    
2. **User Logs In & Grants Access**  
    The user authenticates with Google and consents to the scopes (like reading their email). 
    
3. **Receive Authorization Code**  
    Google redirects back to your app with an authorization code. 
    
4. **Exchange Code for Tokens**  
    Your server exchanges that code for an **access token** (and often a **refresh token**) from Google’s token endpoint. 
    
5. **Use Access Token**  
    Include the token (e.g., `Authorization: Bearer <token>`) in requests to Google APIs like Gmail or Drive. 
    

✔ **Scopes** let you request limited permissions (e.g., only email, only profile).   
✔ **Refresh tokens** allow your app to get new access tokens without further user interaction. 

**Real uses:**

- “Sign in with Google” on websites and mobile apps. 
    
- Accessing Google APIs (Gmail, Drive, Calendar) from third-party clients.


## Using OAuth 2.0 to Access Google APIs

To begin, obtain OAuth 2.0 client credentials from the [Google API Console](https://console.developers.google.com/). Then your client application requests an access token from the Google Authorization Server, extracts a token from the response, and sends the token to the Google API that you want to access.
## Basic steps

All applications follow a basic pattern when accessing a Google API using OAuth 2.0. At a high level, you follow five steps:

### 1. Obtain OAuth 2.0 credentials from the Google API Console.

Visit the [Google API Console](https://console.developers.google.com/) to obtain OAuth 2.0 credentials such as a client ID and client secret that are known to both Google and your application. The set of values varies based on what type of application you are building. For example, a JavaScript application does not require a secret, but a web server application does.

**You must create an OAuth client appropriate for the platform on which your app will run,**

### 2. Obtain an access token from the Google Authorization Server.

Before your application can access private data using a Google API, it must obtain an access token that grants access to that API. A single access token can grant varying degrees of access to multiple APIs. A variable parameter called `scope` controls the set of resources and operations that an access token permits. During the access-token request, your application sends one or more values in the `scope` parameter.

Some requests require an authentication step where the user logs in with their Google account. After logging in, the user is asked whether they are willing to grant one or more permissions that your application is requesting. This process is called user **consent**.

### 3. Examine scopes of access granted by the user.

Compare the scopes included in the access token response to the scopes required to access features and functionality of your application dependent upon access to a related Google API. Disable any features of your app unable to function without access to the related API.

### 4. Send the access token to an API.

After an application obtains an access token, it sends the token to a Google API in an [HTTP Authorization request header](https://developer.mozilla.org/docs/Web/HTTP/Headers/Authorization).
Access tokens are valid only for the set of operations and resources described in the `scope`of the token request. For example, if an access token is issued for the Google Calendar API, it does not grant access to the Google Contacts API. You can, however, send that access token to the Google Calendar API multiple times for similar operations.

### 5. Refresh the access token, if necessary.

Access tokens have limited lifetimes. If your application needs access to a Google API beyond the lifetime of a single access token, it can obtain a refresh token. A refresh token allows your application to obtain new access tokens.

![[Pasted image 20260131153415.png]]
### Client-side (JavaScript) applications

The Google OAuth 2.0 endpoint supports JavaScript applications that run in a browser.

The authorization sequence begins when your application redirects a browser to a Google URL; the URL includes query parameters that indicate the type of access being requested. Google handles the user authentication, session selection, and user consent.

The result is an access token, which the client should validate before including it in a Google API request. When the token expires, the application repeats the process.

![[Pasted image 20260131153859.png]]
### Service accounts

Google APIs such as the Prediction API and Google Cloud Storage can act on behalf of your application without accessing user information. In these situations your application needs to prove its own identity to the API, but no user consent is necessary. Similarly, in enterprise scenarios, your application can request delegated access to some resources.

For these types of server-to-server interactions you need a **service account**, which is an account that belongs to your application instead of to an individual end-user. Your application calls Google APIs on behalf of the service account, and user consent is not required. (In non-service-account scenarios, your application calls Google APIs on behalf of end-users, and user consent is sometimes required.)

A service account's credentials, which you obtain from the Google API Console, include a generated email address that is unique, a client ID, and at least one public/private key pair. You use the client ID and one private key to create a signed JWT and construct an access-token request in the appropriate format. Your application then sends the token request to the Google OAuth 2.0 Authorization Server, which returns an access token. The application uses the token to access a Google API. When the token expires, the application repeats the process.


# Using OAuth 2.0 for Web Server Applications


This OAuth 2.0 flow is specifically for user authorization. It is designed for applications that can store confidential information and maintain state. A properly authorized web server application can access an API while the user interacts with the application or after the user has left the application.

Web server applications frequently also use [service accounts](https://developers.google.com/identity/protocols/oauth2/service-account) to authorize API requests, particularly when calling Cloud APIs to access project-based data rather than user-specific data. Web server applications can use service accounts in conjunction with user authorization.