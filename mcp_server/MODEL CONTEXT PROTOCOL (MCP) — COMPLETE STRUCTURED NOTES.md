```
23:31
```
mcp -> anthropic

Before mcp

service will have apis eg, create update delete


```
================================================================
   MODEL CONTEXT PROTOCOL (MCP) — COMPLETE STRUCTURED NOTES
================================================================

MCP — UNDERSTANDING (BEFORE vs AFTER)
----------------------------------------------------------------
1. THE PROBLEM — LIFE BEFORE MCP (DEEP DIVE)
----------------------------------------------------------------

THE BASIC CHALLENGE:
  AI models (like Claude, ChatGPT) are POWERFUL but LIMITED.
  They live inside a closed box — they can only see and use:
    - Their training data (frozen at some date)
    - The text you type into them

  They CANNOT (by themselves):
    - Read your files
    - Query your database
    - Send emails or Slack messages
    - Call APIs (Oracle, GitHub, Stripe, etc.)
    - Access real-time data (weather, stocks, news)

  To make AI useful in the real world, we NEED to connect it
  to external systems. But how?


THE OLD WAY — DIRECT INTEGRATION:

  Before MCP, every developer wrote CUSTOM CODE to connect
  the AI (or their app) to each external service.

  Imagine you're building an AI assistant. You want it to:
    - Query Oracle database
    - Read GitHub issues
    - Send Slack messages
    - Check Google Calendar

  You would write 4 separate integration layers — one for
  each service — with custom auth, error handling, retries,
  parsing logic, and prompt formatting.


VISUAL — BEFORE MCP (DIRECT API CONNECTIONS):

  ┌─────────────────┐
  │  Developer / AI │
  │   (Python app)  │
  └─────────────────┘
       │
       ├──── Own Logic ────► ┌──────────────────────┐
       │  (custom code 1)    │   Oracle API         │
       │                     │  oracle/v2/create    │
       │                     │  oracle/v1/query     │
       │                     │  oracle/v1/update    │
       │                     └──────────────────────┘
       │
       └──── Own Logic ────► ┌──────────────────────┐
          (custom code 2)    │   Another API        │
                             │  oracle/v2/create    │
                             │  oracle/v1/query     │
                             │  oracle/v1/update    │
                             └──────────────────────┘

  Each "Own Logic" block is your custom wrapper:
    - Handles authentication (API keys, OAuth tokens)
    - Builds HTTP requests with correct headers
    - Parses responses
    - Manages errors and retries
    - Translates API data into something the AI understands


THE 3 BIG PROBLEMS (from the diagram):

  1. API URL CHANGE
     Example:
        oracle/v1/create  →  oracle/v2/create
     The API team updates the endpoint. Your code now breaks.
     You must hunt through your codebase to fix every call.

     If you have 10 apps using this API → 10 places to fix.

  2. API LOGIC CHANGE
     The API changes:
        - Request format (JSON shape changes)
        - Response format (new fields, removed fields)
        - Authentication method (Basic auth → OAuth)
     Your wrapper logic must be rewritten.

  3. DIFFERENT CONNECTION CODE
     Oracle uses REST + Basic Auth
     GitHub uses REST + OAuth Bearer tokens
     Slack uses webhooks + signing secrets
     Postgres uses TCP + SQL protocol

     Every service is DIFFERENT, so every integration is
     custom — there's no shared pattern you can reuse.


OTHER HIDDEN PROBLEMS:

  - NO STANDARDIZATION
    Two developers integrating Oracle in two apps will write
    completely different wrappers. No shared standard.

  - HARD FOR AI MODELS
    LLMs cannot RELIABLY call APIs directly. They don't know
    the exact endpoint, payload format, or error semantics.
    They need a STRUCTURED, DESCRIBED interface to use tools
    safely.

  - TIGHT COUPLING
    Your app's code is locked to a specific API version.
    If you want to switch from Oracle → Postgres, you rewrite
    everything.

  - SECURITY SCATTERED
    API keys, tokens, rate limits, and access rules are
    spread across every app and every integration. Hard to
    audit, hard to rotate, hard to secure.

  - NO DISCOVERY
    Your AI has no way to ASK "what tools are available?"
    You must hardcode every tool's existence in your prompt.


REAL-WORLD ANALOGY:

  Imagine every electronic device needed its OWN power outlet
  shape. Phones, laptops, TVs, fridges — all different plugs,
  different voltages, different cables.

  You'd carry 20 chargers everywhere. Chaos.

  That's what AI ↔ tool integration looked like before MCP.
  Every integration was a custom plug.

----------------------------------------------------------------
2. WHAT IS MCP — DEEP EXPLANATION
----------------------------------------------------------------

DEFINITION:
  MCP (Model Context Protocol) is an OPEN STANDARD released
  by Anthropic in late 2024 that defines a SINGLE, UNIFORM
  way for AI applications to connect to external tools, data,
  and services.

  Think of MCP as the "USB-C for AI integrations."
  One plug. Works everywhere.


THE CORE IDEA:

  Instead of writing custom code for every API, you wrap each
  API inside an "MCP SERVER". The AI app (called an MCP HOST)
  talks to that server through one STANDARD PROTOCOL.

  The protocol defines:
    - How to discover what a server can do
    - How to call its functions (tools)
    - How to read its data (resources)
    - How to handle errors
    - How to authenticate

  Once you build an MCP server for Oracle, ANY AI app
  (Claude, Cursor, ChatGPT clients, etc.) can use it WITHOUT
  writing custom integration code.


VISUAL — WITH MCP (STANDARDIZED CONNECTIONS):

  ┌─────────────────┐
  │  Developer / AI │
  │   (Python app)  │
  └─────────────────┘
       │
       │ Standard MCP protocol (JSON-RPC)
       │
       ├──────► ┌──────────────────┐      ┌──────────────────┐
       │        │   MCP Server     │      │   Oracle API     │
       │        │  ┌────────────┐  │      │ oracle/v2/create │
       │        │  │  func      │──┼─────►│ oracle/v1/query  │
       │        │  │  func      │──┼─────►│ oracle/v1/update │
       │        │  │  func      │──┼─────►└──────────────────┘
       │        │  │  func      │  │
       │        │  └────────────┘  │
       │        └──────────────────┘
       │
       └──────► ┌──────────────────┐      ┌──────────────────┐
                │   MCP Server     │      │   Another API    │
                │  ┌────────────┐  │      │ oracle/v2/create │
                │  │  func      │──┼─────►│ oracle/v1/query  │
                │  │  func      │──┼─────►│ oracle/v1/update │
                │  │  func      │──┼─────►└──────────────────┘
                │  │  func      │  │
                │  └────────────┘  │
                └──────────────────┘

func -> wrapper functions which internally calls the apis

HOW THE PICTURE CHANGED:

  Notice the difference vs the BEFORE diagram:

  Before MCP:
    AI ──(custom code per API)──► API

  With MCP:
    AI ──(standard protocol)──► MCP Server ──► API

  The MCP Server is a TRANSLATOR:
    - On one side, it speaks the standard MCP protocol
    - On the other side, it speaks the API's native language
    - It HIDES the API's complexity behind clean "functions"


WHAT'S INSIDE AN MCP SERVER?

  An MCP server exposes 3 things to the AI:

  1. TOOLS — actions the AI can take
     Examples:
       - create_record(name, email)
       - send_message(channel, text)
       - get_weather(city)

  2. RESOURCES — data the AI can read
     Examples:
       - file://config.yaml
       - db://users/table
       - api://github/repos/myrepo

  3. PROMPTS — pre-built templates for common tasks
     Examples:
       - "summarize_pull_request"
       - "generate_bug_report"

  Each tool has:
     - A NAME (what to call it)
     - A DESCRIPTION (what it does, in plain English)
     - An INPUT SCHEMA (what arguments it needs)

  This makes it easy for the AI to UNDERSTAND and USE.


HOW THE AI DISCOVERS TOOLS:

  When the AI host (Claude) connects to an MCP server:

    Step 1: Handshake
       Client → Server: "Hi, I'm an MCP client v1.0"
       Server → Client: "Hi, I'm an MCP server v1.0"

    Step 2: Capability exchange
       Client → Server: "What tools / resources do you have?"
       Server → Client: "Here is my list:
                          - tool: create_record(name, email)
                          - tool: query_record(id)
                          - resource: db://users
                          ..."

    Step 3: Tool use
       AI decides: "I need to create a user."
       Client → Server: call create_record(name="John")
       Server hits real API → returns result
       Server → Client: { success: true, id: 42 }
       AI: "Done! User created with ID 42."

  This is HUGE because the AI doesn't need to be PRE-TRAINED
  on every API. It just asks the server what's available
  and uses it.


REAL-WORLD ANALOGY (CONTINUED):

  Think of MCP servers like "apps in an app store":

    - The MCP Host (Claude) = your phone
    - MCP Servers = installable apps
    - Each MCP server adds new abilities to the AI
    - You can mix and match servers for any use case

  Example setup for a developer using Claude:
    + GitHub MCP server   → AI can manage repos
    + Filesystem MCP      → AI can read project files
    + Postgres MCP        → AI can query the DB
    + Slack MCP           → AI can post messages

  The AI suddenly has SUPERPOWERS without changing its code.


WHY IT'S CALLED "MODEL CONTEXT PROTOCOL":

  - MODEL    = the AI model (Claude, GPT, etc.)
  - CONTEXT  = the data, tools, and capabilities given to it
  - PROTOCOL = the standard rules for delivering that context

  So MCP literally means:
  "A standard way to deliver context (tools + data) to a model."

================================================================
```

---

```
================================================================
   MCP COMPONENTS — THE 3 CORE PARTS
================================================================

----------------------------------------------------------------
1. OVERVIEW — MCP HAS 3 MAIN COMPONENTS
----------------------------------------------------------------
Every MCP setup involves THREE roles working together.

  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
  │   MCP HOST   │   │  MCP CLIENT  │   │  MCP SERVER  │
  │              │   │              │   │              │
  │ Environment  │   │ Connector    │   │ Tool         │
  │ where AI     │   │ inside host  │   │ provider     │
  │ runs         │   │              │   │              │
  └──────────────┘   └──────────────┘   └──────────────┘

  Think of it like:
    Host    = the BUILDING
    Client  = the PHONE inside the building
    Server  = the SERVICE you're calling

----------------------------------------------------------------
2. MCP HOST
----------------------------------------------------------------

DEFINITION:
  The HOST is the ENVIRONMENT where we build connections —
  the application or runtime that the user actually interacts
  with, and where the AI / code lives.

EXAMPLES OF HOSTS:
  - Claude Desktop App
  - Cursor IDE
  - VS Code
  - A custom Python script
  - Any application that wants to use MCP servers

VISUAL:

  ┌──────────────────────────────────┐
  │         MCP HOST                 │
  │  (Claude / Cursor / VS Code /    │
  │     Python Code, etc.)           │
  │                                  │
  │   ┌────────────────────────┐     │
  │   │   The AI / LLM Brain   │     │ ← decision maker
  │   ├────────────────────────┤     │
  │   │   MCP Client (inside)  │     │ ← talks to servers
  │   └────────────────────────┘     │
  └──────────────────────────────────┘

KEY POINTS:
  - Host is the USER-FACING app
  - Host EMBEDS the MCP client inside it
  - Host OWNS the LLM (or runs the code that uses MCP)
  - Host decides WHICH MCP servers to connect to

----------------------------------------------------------------
3. MCP CLIENT
----------------------------------------------------------------

DEFINITION:
  The CLIENT is the part INSIDE the host that knows how to
  speak the MCP protocol and connect to MCP servers.

ROLE:
  - Establishes the connection to MCP servers
  - Sends JSON-RPC requests to the server
  - Receives responses and forwards them to the host
  - Manages multiple server connections at once

VISUAL:

  ┌──────────────────────────────────┐
  │            HOST                  │
  │                                  │
  │   ┌────────────────────────┐     │
  │   │     MCP CLIENT         │     │
  │   │  - Connects to servers │     │
  │   │  - Sends requests      │     │
  │   │  - Receives responses  │     │
  │   └────────────────────────┘     │
  └────────────────┬─────────────────┘
                   │
                   │ MCP protocol
                   ▼
            (to MCP servers)

KEY POINTS:
  - Client lives INSIDE the host (not standalone)
  - Client speaks the STANDARD MCP protocol
  - One host can have MULTIPLE clients (one per server)
  - You usually don't build the client yourself — Anthropic
    or library authors provide it

----------------------------------------------------------------
4. MCP SERVER
----------------------------------------------------------------

DEFINITION:
  The SERVER is a program that WRAPS an external service or
  data source and exposes it as MCP tools/resources/prompts.

ROLE:
  - Wraps an external API, database, file system, etc.
  - Exposes TOOLS (functions the AI can call)
  - Exposes RESOURCES (data the AI can read)
  - Exposes PROMPTS (templates for AI to use)
  - Handles authentication with the underlying service

VISUAL:

           From MCP Client
                  │
                  │ MCP protocol
                  ▼
  ┌──────────────────────────────────┐
  │         MCP SERVER               │
  │                                  │
  │   ┌────────────────────────┐     │
  │   │  Tools / Resources /   │     │
  │   │  Prompts (exposed)     │     │
  │   └────────────────────────┘     │
  │              │                   │
  │              ▼                   │
  │   ┌────────────────────────┐     │
  │   │  External Service      │     │
  │   │  (API / DB / files)    │     │
  │   └────────────────────────┘     │
  └──────────────────────────────────┘

EXAMPLES:
  - GitHub MCP server     → wraps GitHub API
  - Postgres MCP server   → wraps a Postgres database
  - Filesystem MCP server → wraps your local files
  - Slack MCP server      → wraps Slack API

KEY POINTS:
  - Server is INDEPENDENT of the host
  - One server can be used by MANY hosts
  - You CAN build your own MCP server (most common dev task)
  - Server can run locally (stdio) or remotely (HTTP)

----------------------------------------------------------------
5. HOW THEY WORK TOGETHER
----------------------------------------------------------------

FULL PICTURE — ALL 3 COMPONENTS:

  ┌──────────────────────────────────┐
  │           MCP HOST               │
  │  (Claude / Cursor / VS Code)     │
  │                                  │
  │   ┌────────────────────────┐     │
  │   │   LLM / AI Brain       │     │
  │   ├────────────────────────┤     │
  │   │   MCP CLIENT           │     │
  │   └────────────────────────┘     │
  └────────────────┬─────────────────┘
                   │
                   │ MCP protocol (JSON-RPC)
                   │ over stdio or HTTP
                   │
                   ├──────► ┌──────────────────────┐
                   │        │  MCP SERVER: GitHub  │ ──► GitHub API
                   │        └──────────────────────┘
                   │
                   ├──────► ┌──────────────────────┐
                   │        │  MCP SERVER: Postgres│ ──► Postgres DB
                   │        └──────────────────────┘
                   │
                   └──────► ┌──────────────────────┐
                            │  MCP SERVER: Files   │ ──► Local FS
                            └──────────────────────┘

FLOW:
  1. User talks to the HOST (e.g. types in Claude)
  2. HOST's LLM decides it needs a tool
  3. HOST's CLIENT sends a request to the right MCP SERVER
  4. SERVER calls the external service and returns the result
  5. CLIENT delivers the result back to the HOST
  6. HOST shows the result to the user

----------------------------------------------------------------
6. SIDE-BY-SIDE COMPARISON
----------------------------------------------------------------

  ┌────────────┬──────────────────────────────────────────────┐
  │ Component  │ Description                                  │
  ├────────────┼──────────────────────────────────────────────┤
  │ HOST       │ Environment where AI runs.                   │
  │            │ Examples: Claude, Cursor, VS Code, Python    │
  │            │ It contains the LLM and the MCP client.      │
  ├────────────┼──────────────────────────────────────────────┤
  │ CLIENT     │ Lives inside the host.                       │
  │            │ Talks the MCP protocol.                      │
  │            │ Sends requests, receives responses.          │
  ├────────────┼──────────────────────────────────────────────┤
  │ SERVER     │ Wraps an external service (API / DB / FS).   │
  │            │ Exposes tools, resources, prompts.           │
  │            │ Can be local (stdio) or remote (HTTP).       │
  └────────────┴──────────────────────────────────────────────┘

----------------------------------------------------------------
7. KEY INTUITION SUMMARY
----------------------------------------------------------------

  HOST    = "where the AI lives" (Claude / Cursor / VS Code)
  CLIENT  = "the messenger inside the host"
  SERVER  = "the tool provider outside the host"

  Relationship:
    HOST  contains  CLIENT
    CLIENT  talks to  SERVER (via MCP protocol)
    SERVER  wraps  external service

  ONE-LINER:
    "The HOST uses its CLIENT to talk to MCP SERVERS,
     which expose tools and data to the AI."

================================================================
```



---

```
================================================================
   MCP — DATA LAYER & TRANSPORT LAYER
================================================================

----------------------------------------------------------------
1. THE TWO LAYERS OF MCP
----------------------------------------------------------------
MCP is built on TWO layers that work together.

  ┌─────────────────────────────────────────┐
  │            MCP ARCHITECTURE             │
  │                                         │
  │   ┌───────────────────────────────┐     │
  │   │        DATA LAYER             │     │ ← WHAT messages
  │   │  (Message format + semantics) │     │   look like
  │   └───────────────────────────────┘     │
  │                  │                      │
  │                  ▼                      │
  │   ┌───────────────────────────────┐     │
  │   │      TRANSPORT LAYER          │     │ ← HOW messages
  │   │  (Communication channel)      │     │   travel
  │   └───────────────────────────────┘     │
  │                                         │
  └─────────────────────────────────────────┘

THINK OF IT LIKE THIS:

  - Data Layer       = the LANGUAGE (what to say)
  - Transport Layer  = the PHONE LINE (how to send it)

  You can keep the same language (JSON-RPC) but change the
  phone line (stdio vs HTTP) without changing anything else.

----------------------------------------------------------------
2. DATA LAYER
----------------------------------------------------------------

DEFINITION:
  The data layer implements a JSON-RPC 2.0 based exchange
  protocol that defines the MESSAGE STRUCTURE and SEMANTICS.

  In plain English:
    "It's the rulebook for what every message looks like
     and what every message means."

WHY JSON-RPC 2.0?
  - Lightweight, human-readable JSON format
  - Industry-standard, well-supported in every language
  - Supports requests, responses, and notifications
  - Easy to debug (just JSON text)

EXAMPLE JSON-RPC MESSAGE:

  ┌─────────────────────────────────────────┐
  │ {                                       │
  │   "jsonrpc": "2.0",                     │
  │   "id": 1,                              │
  │   "method": "tools/call",               │
  │   "params": {                           │
  │     "name": "create_record",            │
  │     "arguments": { "name": "John" }     │
  │   }                                     │
  │ }                                       │
  └─────────────────────────────────────────┘

WHAT THE DATA LAYER INCLUDES:

  1. LIFECYCLE MANAGEMENT
     Handles:
       - Connection INITIALIZATION
       - CAPABILITY negotiation
       - Connection TERMINATION
     Between clients and servers.

     Example flow:
       Client → Server : "Hello, I'm v1.0, I support X"
       Server → Client : "Hello, I'm v1.0, I support Y"
       Both agree on common features → session begins

  2. SERVER FEATURES
     What servers expose to clients:
       - TOOLS      → for AI actions (functions to call)
       - RESOURCES  → for context data (read-only data)
       - PROMPTS    → for interaction templates

  3. CLIENT FEATURES
     What clients allow servers to ask for:
       - SAMPLING   → server asks client's LLM to generate
       - ELICITATION→ server asks user for input
       - LOGGING    → server sends log messages to client

  4. UTILITY FEATURES
     Extra capabilities:
       - NOTIFICATIONS    → real-time updates
       - PROGRESS TRACKING→ for long-running operations

VISUAL — DATA LAYER OVERVIEW:

  ┌─────────────────────────────────────────────────┐
  │              DATA LAYER (JSON-RPC 2.0)          │
  ├─────────────────────────────────────────────────┤
  │                                                 │
  │   Lifecycle      │ init / negotiate / terminate │
  │   Server feat.   │ tools, resources, prompts    │
  │   Client feat.   │ sampling, elicit, logging    │
  │   Utility feat.  │ notifications, progress      │
  │                                                 │
  └─────────────────────────────────────────────────┘

----------------------------------------------------------------
3. TRANSPORT LAYER
----------------------------------------------------------------

DEFINITION:
  The transport layer manages COMMUNICATION CHANNELS and
  AUTHENTICATION between clients and servers.

  In plain English:
    "It's the pipe through which the JSON messages travel,
     and the security around that pipe."

WHAT THE TRANSPORT LAYER HANDLES:
  - Connection establishment
  - Message framing (where one message ends and another starts)
  - Secure communication between MCP participants
  - Authentication (tokens, API keys, OAuth)

MCP SUPPORTS 2 TRANSPORT MECHANISMS:

  ┌──────────────────────┐    ┌──────────────────────┐
  │   STDIO TRANSPORT    │    │  STREAMABLE HTTP     │
  │                      │    │      TRANSPORT       │
  │  Local processes,    │    │  Remote servers,     │
  │  same machine        │    │  network calls       │
  └──────────────────────┘    └──────────────────────┘

----------------------------------------------------------------
4. STDIO TRANSPORT
----------------------------------------------------------------

PURPOSE:
  Communication between processes on the SAME machine.

HOW IT WORKS:
  Uses standard input (stdin) and output (stdout) streams
  to exchange messages.

VISUAL:

  ┌──────────────────────┐         ┌──────────────────────┐
  │   MCP Client (Host)  │         │     MCP Server       │
  │   (e.g. Claude app)  │         │  (local process)     │
  │                      │         │                      │
  │   stdout  ────────►  │ ───────►│  stdin               │
  │                      │         │                      │
  │   stdin   ◄────────  │ ◄───────│  stdout              │
  └──────────────────────┘         └──────────────────────┘
            (same machine, direct pipe)

KEY POINTS:
  - No network calls — direct process-to-process pipe
  - Extremely fast (no network overhead)
  - Used for LOCAL MCP servers (filesystem, local DB, etc.)
  - Server is launched by the client as a child process

WHEN TO USE STDIO:
  - Local file system access
  - Local database tools
  - Developer tools on your own machine
  - Any case where the server runs LOCALLY

----------------------------------------------------------------
5. STREAMABLE HTTP TRANSPORT
----------------------------------------------------------------

PURPOSE:
  Communication with REMOTE servers over the internet.

HOW IT WORKS:
  - Client sends messages via HTTP POST
  - Server can stream responses using Server-Sent Events (SSE)
  - Supports standard HTTP authentication

VISUAL:

  ┌──────────────────────┐         ┌──────────────────────┐
  │   MCP Client (Host)  │         │   Remote MCP Server  │
  │   (e.g. Claude app)  │         │   (cloud-hosted)     │
  │                      │         │                      │
  │           HTTP POST  │ ───────►│                      │
  │                      │         │                      │
  │     SSE (streaming)  │ ◄───────│                      │
  └──────────────────────┘         └──────────────────────┘
                (across the network)

AUTHENTICATION:
  Supports standard HTTP auth methods:
    - Bearer tokens
    - API keys
    - Custom headers
    - OAuth (recommended by MCP)

KEY POINTS:
  - Works over the internet (remote servers)
  - Uses HTTP POST for client-to-server messages
  - Uses Server-Sent Events (SSE) for streaming responses
  - Network overhead, but allows global reach
  - Server can be hosted on any cloud / domain

WHEN TO USE STREAMABLE HTTP:
  - Connecting to SaaS APIs (Slack, GitHub, Notion, etc.)
  - Multi-user / shared MCP servers
  - Servers hosted on the cloud
  - Anywhere the server is NOT on the same machine

----------------------------------------------------------------
6. STDIO vs STREAMABLE HTTP
----------------------------------------------------------------

  ┌──────────────────────┬──────────────────────────────────┐
  │ STDIO                │ STREAMABLE HTTP                  │
  ├──────────────────────┼──────────────────────────────────┤
  │ Same machine         │ Across network                   │
  │ stdin/stdout pipes   │ HTTP POST + SSE                  │
  │ No network overhead  │ Has network latency              │
  │ Local servers        │ Remote / cloud servers           │
  │ Launched as a child  │ Independent service              │
  │  process by client   │  hosted anywhere                 │
  │ Faster               │ Slower, but global               │
  │ No auth needed       │ Tokens, API keys, OAuth          │
  │ Example: filesystem  │ Example: GitHub, Slack, Notion   │
  └──────────────────────┴──────────────────────────────────┘

----------------------------------------------------------------
7. WHY THE TRANSPORT LAYER IS SEPARATE
----------------------------------------------------------------

KEY DESIGN PRINCIPLE:
  The transport layer ABSTRACTS communication details
  from the protocol layer.

  This means:
    - Same JSON-RPC 2.0 message format works for BOTH
      stdio and HTTP transports.
    - You can SWITCH transports without changing message logic.
    - The data layer doesn't care HOW messages move — only
      that they move.

VISUAL:

  ┌───────────────────────────────────────┐
  │     JSON-RPC 2.0 Message (same)       │
  └───────────────────────────────────────┘
                │
                ▼
       ┌────────┴─────────┐
       │                  │
       ▼                  ▼
  ┌─────────┐        ┌──────────┐
  │  STDIO  │   OR   │   HTTP   │
  └─────────┘        └──────────┘

  → Plug-and-play transport choice.

----------------------------------------------------------------
8. FULL PICTURE — LAYERS TOGETHER
----------------------------------------------------------------

  ┌─────────────────────────────────────────────┐
  │              MCP CLIENT (Host)              │
  │                                             │
  │  ┌────────────────────────────────────┐     │
  │  │   Data Layer (JSON-RPC 2.0)        │     │
  │  │   - lifecycle, tools, resources,   │     │
  │  │     prompts, notifications         │     │
  │  └────────────────────────────────────┘     │
  │                  │                          │
  │  ┌────────────────────────────────────┐     │
  │  │   Transport Layer                  │     │
  │  │   - stdio  OR  streamable HTTP     │     │
  │  └────────────────────────────────────┘     │
  └─────────────────────────────────────────────┘
                   │
                   ▼
  ┌─────────────────────────────────────────────┐
  │              MCP SERVER                     │
  │                                             │
  │  ┌────────────────────────────────────┐     │
  │  │   Transport Layer                  │     │
  │  │   - stdio  OR  streamable HTTP     │     │
  │  └────────────────────────────────────┘     │
  │                  │                          │
  │  ┌────────────────────────────────────┐     │
  │  │   Data Layer (JSON-RPC 2.0)        │     │
  │  │   - exposes tools, resources,      │     │
  │  │     prompts                        │     │
  │  └────────────────────────────────────┘     │
  └─────────────────────────────────────────────┘

----------------------------------------------------------------
9. KEY INTUITION SUMMARY
----------------------------------------------------------------

  MCP = Data Layer + Transport Layer

  DATA LAYER:
    - JSON-RPC 2.0 format
    - Defines message structure + meaning
    - Handles lifecycle, tools, resources, prompts

  TRANSPORT LAYER:
    - Moves messages between client and server
    - Two flavors:
        * stdio       → local, fast, same machine
        * HTTP+SSE    → remote, networked, auth required

  KEY INSIGHT:
    The two layers are INDEPENDENT. The same JSON-RPC
    message can travel over stdio OR HTTP without
    changing its content.

================================================================
```





used langchain to create a tool