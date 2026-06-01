```
═══════════════════════════════════════════════════════════════════
  MCP LEARNING NOTES — Session 1
  Date: 2026-05-26
═══════════════════════════════════════════════════════════════════


1. WHAT IS MCP?
----------------------------------------------------------------------

Problem:
  Every AI app had to write CUSTOM integration code for every tool
  (databases, APIs, files, etc.). N apps × M tools = N×M glue code.

Concept:
  MCP (Model Context Protocol) = a STANDARD "language" between AI
  models and external tools. Think of it as "USB for AI."

How it works:
  ┌──────────┐   speaks MCP   ┌──────────┐   speaks MCP   ┌─────────┐
  │   AI     │ ─────────────► │  Client  │ ─────────────► │  Tool   │
  │ (Claude) │ ◄───────────── │ (bridge) │ ◄───────────── │ Server  │
  └──────────┘                └──────────┘                └─────────┘

Before MCP: N×M custom integrations
After  MCP: N + M (each side only learns one protocol)

----------------------------------------------------------------------


2. THE THREE ROLES IN MCP
----------------------------------------------------------------------

  ┌──────────────────────────────────────────────────────┐
  │                       HOST                           │
  │           (Claude Desktop / Claude Code)             │
  │   ┌──────────────────────────────────────────────┐   │
  │   │  The LLM (the brain)                         │   │
  │   └──────────────────────────────────────────────┘   │
  │   ┌──────────────────────────────────────────────┐   │
  │   │  CLIENT (the waiter — already built-in)      │   │
  │   └──────────────┬───────────────────────────────┘   │
  └──────────────────┼───────────────────────────────────┘
                     │ stdio / http
                     ▼
              ┌────────────┐
              │   SERVER   │  ◄── the kitchen (what you write)
              │  @tool def │
              │  fetch()   │
              │  process() │
              └────────────┘

Restaurant analogy:
  Host   = hungry customer (the AI)
  Client = waiter (carries messages)
  Server = kitchen (does the actual work)

----------------------------------------------------------------------


3. THE SERVER (stdio_server.py)
----------------------------------------------------------------------

  from fastmcp import FastMCP

  mcp = FastMCP()              # create the "kitchen"

  @mcp.tool()                  # ◄── this decorator is the magic
  async def fetch():
      '''docstring → LLM reads this to decide WHEN to call it'''
      return {"data": "Hello, MCP!"}

  mcp.run(transport="stdio")   # talk via stdin/stdout

Key insight:
  • The @mcp.tool() decorator advertises a Python function as a tool.
  • The DOCSTRING is not just a comment — the LLM reads it to decide
    when to call the tool. Good docstrings = better tool selection.

----------------------------------------------------------------------


4. TRANSPORTS — *HOW* MESSAGES TRAVEL
----------------------------------------------------------------------

  ┌─────────────┬──────────────────────────┬──────────────────────┐
  │  Transport  │       How it works       │      When to use     │
  ├─────────────┼──────────────────────────┼──────────────────────┤
  │   stdio     │ Client launches server   │ Local tools on your  │
  │             │ as subprocess, pipes     │ own machine          │
  │             │ via stdin/stdout         │                      │
  ├─────────────┼──────────────────────────┼──────────────────────┤
  │  HTTP/SSE   │ Server runs as a web     │ Remote tools, cloud  │
  │             │ service on a port        │ services             │
  └─────────────┴──────────────────────────┴──────────────────────┘

MCP doesn't care HOW bytes travel — only the message format matters.

----------------------------------------------------------------------


5. THE RAW PYTHON CLIENT (stdio_client.py)
----------------------------------------------------------------------

Flow:
  ┌──────────────┐   spawn server    ┌──────────────┐
  │              │ ────────────────► │              │
  │   CLIENT     │   list_tools      │   SERVER     │
  │              │ ────────────────► │              │
  │              │ ◄── [fetch,proc]  │  @tool fetch │
  │              │   call_tool(fetch)│  @tool proc  │
  │              │ ────────────────► │              │
  │              │ ◄── {"data":...}  │              │
  └──────────────┘                   └──────────────┘
         │                                  │
         └────── stdio pipes ───────────────┘

Key code pattern:
  async with stdio_client(server_params) as (read, write):  # TRANSPORT
      async with ClientSession(read, write) as session:      # PROTOCOL
          await session.initialize()
          tools  = await session.list_tools()
          result = await session.call_tool("fetch")

----------------------------------------------------------------------


6. WHY DOUBLE `async with`?
----------------------------------------------------------------------

Each `async with` manages a DIFFERENT lifecycle:

  ┌────────────────────────────┬───────────────────────────────────┐
  │      Block                 │   What it manages                  │
  ├────────────────────────────┼───────────────────────────────────┤
  │ async with stdio_client()  │  TRANSPORT (pipes + subprocess)    │
  │                            │  Cleanup: kill server, close pipes │
  ├────────────────────────────┼───────────────────────────────────┤
  │ async with ClientSession() │  PROTOCOL session (the conversation)│
  │                            │  Cleanup: goodbye msg, cancel reqs │
  └────────────────────────────┴───────────────────────────────────┘

Phone-call analogy:
  1. Dial number     → connection opens   (stdio_client)
  2. Say "Hello"     → conversation start (ClientSession)
  3. ...talk...
  4. Say "Goodbye"   → conversation ends  (inner block exits first)
  5. Hang up         → connection closes  (outer block exits last)

Why separated? The same ClientSession works with ANY transport:
  async with stdio_client(...) as (r,w):          # stdio
  async with streamablehttp_client(...) as(r,w,_):# http
      async with ClientSession(r, w) as session:  # ← SAME LINE
          ...

----------------------------------------------------------------------


7. THE LANGCHAIN CLIENT (langchain_client.py)
----------------------------------------------------------------------

Concept:
  langchain-mcp-adapters = a BRIDGE that converts MCP tools into
  LangChain Tool objects, so a LangChain agent (LLM) can use them.

Architecture:
  ┌─────────────────────────────────────────────┐
  │            LangChain Agent (LLM)            │
  └────────────────────┬────────────────────────┘
                       │ uses LangChain tools
                       ▼
  ┌─────────────────────────────────────────────┐
  │         MultiServerMCPClient                │
  │  - launches all configured servers          │
  │  - calls list_tools() on each               │
  │  - wraps MCP tools as LangChain Tools       │
  └────┬───────────────┬───────────────┬────────┘
       │               │               │
       ▼               ▼               ▼
   MCP Server A   MCP Server B   MCP Server C

Usage pattern (correct):
  client = MultiServerMCPClient({...})
  tools  = await client.get_tools()
  agent  = create_react_agent(llm, tools)
  await agent.ainvoke({"messages": [...]})

----------------------------------------------------------------------


8. RAW MCP CLIENT vs LANGCHAIN CLIENT
----------------------------------------------------------------------

  ┌──────────────────────┬──────────────────────┬──────────────────────┐
  │       Aspect         │   Raw MCP Client     │  LangChain Adapter   │
  ├──────────────────────┼──────────────────────┼──────────────────────┤
  │ Library              │ mcp SDK              │ langchain-mcp-adapt. │
  │ Multiple servers     │ One per client       │ Many in one config   │
  │ Tool format          │ Raw MCP dicts        │ LangChain Tool objs  │
  │ Connection mgmt      │ Manual (async with)  │ Hidden by adapter    │
  │ Boilerplate          │ More                 │ Less                 │
  │ Best for             │ Learning, testing,   │ Building LLM agents  │
  │                      │ custom hosts, scripts│ on LangChain stack   │
  └──────────────────────┴──────────────────────┴──────────────────────┘

Analogy (validated):
  raw mcp SDK     ≈  psycopg2         (low-level DB driver)
  langchain-mcp   ≈  SQLAlchemy ORM   (framework on top)

Rule of thumb:
  🛠 Tool plumbing / testing  → Raw MCP
  🤖 Building an LLM agent    → LangChain adapter (or your stack's)

But the adapter doesn't REPLACE the raw library — you still drop
down to raw for: building servers, testing, debugging, custom hosts.

----------------------------------------------------------------------


9. USING YOUR SERVER WITH CLAUDE (DESKTOP / CODE)
----------------------------------------------------------------------

Q: Do I need to write a client to use my server with Claude?
A: NO. Claude Desktop / Claude Code / Claude.ai are MCP hosts —
   they have a CLIENT BUILT IN.

You only write:
  ✅ The SERVER
  ✅ A small JSON config that tells the host how to launch it

  ┌─────────────────────────────────────────┐
  │           Claude Desktop                │
  │   ┌─────────────────────────────────┐   │
  │   │   Claude (LLM)                  │   │
  │   └─────────────────────────────────┘   │
  │   ┌─────────────────────────────────┐   │
  │   │   MCP CLIENT (built-in)         │   │  ◄── you DON'T write
  │   └────────────┬────────────────────┘   │
  └────────────────┼────────────────────────┘
                   ▼
            ┌──-──────────┐
            │ YOUR server │   ◄── all you ship
            └────-────────┘

Config file:
  ~/Library/Application Support/Claude/claude_desktop_config.json
  {
    "mcpServers": {
      "practice": {
        "command": "/path/to/.venv/bin/python",
        "args":    ["/path/to/stdio_server.py"]
      }
    }
  }

Same structure as StdioServerParameters — your hand-written client
was basically a mini-Claude-Desktop.

----------------------------------------------------------------------


10. WHEN DO YOU WRITE A CLIENT?
----------------------------------------------------------------------

  ┌─────────────────────────────────────────┬─────────────────────┐
  │              Scenario                   │  Write a client?    │
  ├─────────────────────────────────────────┼─────────────────────┤
  │ Use server in Claude Desktop / Code     │  ❌ No — configure  │
  │ Use server in Cursor / Windsurf / Zed   │  ❌ No — configure  │
  │ Build your own LangChain agent          │  ✅ Yes (adapter)   │
  │ Build a custom non-LangChain AI app     │  ✅ Yes (raw mcp)   │
  │ Test/debug your server in isolation     │  ✅ Yes (raw mcp)   │
  │ Automation script (no LLM)              │  ✅ Yes (raw mcp)   │
  └─────────────────────────────────────────┴─────────────────────┘


═══════════════════════════════════════════════════════════════════
  COMPONENT SUMMARY TABLE
═══════════════════════════════════════════════════════════════════

  ┌─────────────────────────┬──────────────────────────────────────┐
  │ Component               │ What it does                         │
  ├─────────────────────────┼──────────────────────────────────────┤
  │ Host                    │ AI app the user talks to (Claude     │
  │                         │ Desktop, Claude Code, Cursor, etc.)  │
  ├─────────────────────────┼──────────────────────────────────────┤
  │ Client                  │ Speaks MCP, routes between Host & Srv│
  │                         │ Built-in to hosts; can also be raw   │
  │                         │ mcp SDK or langchain-mcp-adapters    │
  ├─────────────────────────┼──────────────────────────────────────┤
  │ Server                  │ Exposes tools/resources via @tool    │
  │                         │ decorator (FastMCP). Stateless code  │
  │                         │ you write once.                      │
  ├─────────────────────────┼──────────────────────────────────────┤
  │ Transport (stdio)       │ Spawns server as subprocess, pipes   │
  │                         │ via stdin/stdout. Local tools.       │
  ├─────────────────────────┼──────────────────────────────────────┤
  │ Transport (HTTP/SSE)    │ Server is a web service on a port.   │
  │                         │ Remote / cloud tools.                │
  ├─────────────────────────┼──────────────────────────────────────┤
  │ stdio_client            │ Helper that launches server + opens  │
  │                         │ stdio pipes. Manages transport life. │
  ├─────────────────────────┼──────────────────────────────────────┤
  │ ClientSession           │ Wraps raw pipes into MCP-protocol-   │
  │                         │ aware object. Provides initialize(), │
  │                         │ list_tools(), call_tool() etc.       │
  ├─────────────────────────┼──────────────────────────────────────┤
  │ StdioServerParameters   │ Recipe describing HOW to launch the  │
  │                         │ server (command + args + env).       │
  ├─────────────────────────┼──────────────────────────────────────┤
  │ FastMCP                 │ The server framework. @tool()        │
  │                         │ decorator + .run(transport=...).     │
  ├─────────────────────────┼──────────────────────────────────────┤
  │ MultiServerMCPClient    │ LangChain adapter. Connects to many  │
  │                         │ MCP servers, exposes tools as        │
  │                         │ LangChain Tool objects.              │
  └─────────────────────────┴──────────────────────────────────────┘


═══════════════════════════════════════════════════════════════════
  KEY TAKEAWAYS
═══════════════════════════════════════════════════════════════════

  • MCP standardizes how AI ↔ tools talk. "USB for AI."
  • Three roles: Host, Client, Server.
  • You write SERVERS. Hosts (Claude) provide the CLIENT.
  • The @mcp.tool() decorator + docstring = how the LLM discovers
    and decides to use your tools.
  • Double `async with` = transport layer + protocol layer,
    cleaned up in reverse order.
  • Raw MCP SDK ≈ psycopg2; LangChain adapter ≈ SQLAlchemy.
  • Use raw MCP for: learning, testing, debugging, custom hosts.
  • Use a framework adapter for: building LLM agents.
  • For Claude Desktop/Code/.ai → no client needed, just config JSON.

═══════════════════════════════════════════════════════════════════

```



---
---




```
═══════════════════════════════════════════════════════════════════
  MCP LEARNING NOTES — Session 2
  Topic: MCP Transports Deep-Dive
  Date: 2026-05-26
═══════════════════════════════════════════════════════════════════


1. WHAT IS A "TRANSPORT" IN MCP?
----------------------------------------------------------------------

Problem:
  MCP defines the LANGUAGE (JSON-RPC messages: initialize,
  list_tools, call_tool, etc.) but the messages still need a
  physical channel to travel through.

Concept:
  Transport = the "road" the MCP messages travel on.
  Protocol  = what they SAY.   Transport = how they MOVE.

Mental picture:
  ┌──────────────────────────────────────────────────────┐
  │              MCP PROTOCOL (JSON-RPC)                 │
  │     initialize, list_tools, call_tool, ...           │
  └──────┬─────────────────┬──────────────────┬──────────┘
         │                 │                  │
         ▼                 ▼                  ▼
    ┌────────┐    ┌──────────────┐    ┌────────────┐
    │ stdio  │    │ streamable-  │    │    sse     │
    │        │    │    http      │    │  (legacy)  │
    └────────┘    └──────────────┘    └────────────┘

----------------------------------------------------------------------


2. STDIO TRANSPORT — "SUBPROCESS + PIPES"
----------------------------------------------------------------------

How it works:
  ┌───────────────────────────────┐
  │           CLIENT              │
  │   1. Spawn server as child    │
  │            │                  │
  │            ▼                  │
  │   ┌───────────────────────┐   │
  │   │ child: python srv.py  │   │
  │   │  stdin  ◄── write ────┼───┼─── msgs IN
  │   │  stdout ──► read  ────┼───┼─── msgs OUT
  │   └───────────────────────┘   │
  └───────────────────────────────┘

Server code:
  mcp.run(transport="stdio")       # no host, no port

Properties:
  • Server runs as CHILD PROCESS of the client.
  • Communicates through stdin/stdout pipes.
  • Server DIES when the client exits.
  • One server instance = one client.
  • No network involved at all.

Best for:
  ✅ Local tools (file system, git, shell)
  ✅ Personal / single-user setups
  ✅ Claude Desktop community servers
  ✅ Tools where no network is desired

⚠️ Gotcha:
  NEVER print() to stdout inside a stdio server —
  stdout IS the protocol channel. Use stderr/logger instead.

----------------------------------------------------------------------


3. STREAMABLE-HTTP TRANSPORT — "WEB SERVICE + STREAMING"
----------------------------------------------------------------------

How it works:
  ┌──────────────┐                       ┌──────────────────┐
  │   CLIENT     │ ─ HTTP POST /mcp ───► │   HTTP SERVER    │
  │              │ ◄── streaming chunks  │  on host:port    │
  │              │     (SSE under hood)  │   (long-running) │
  └──────────────┘                       └──────────────────┘

Server code:
  mcp.run(transport="streamable-http",
          host="0.0.0.0",
          port=8050)

Client import:
  from mcp.client.streamable_http import streamablehttp_client

  async with streamablehttp_client("http://host:8050/mcp") as (r, w, _):
      async with ClientSession(r, w) as session:
          ...

Properties:
  • Server runs INDEPENDENTLY (you start it manually).
  • Listens on a host + port.
  • SINGLE endpoint (/mcp) — handles both sending & receiving.
  • Optional STREAMING responses (server pushes chunks over time).
  • Many clients can connect simultaneously.
  • Server STAYS UP across client connects/disconnects.

Best for:
  ✅ Cloud / remote tools
  ✅ Multi-user / multi-client setups
  ✅ Heavy or stateful servers (caches, DB connections)
  ✅ Production deployments

⚠️ Gotchas:
  • host="0.0.0.0" listens on ALL interfaces (incl. public).
    Use "127.0.0.1" for local-only dev.
  • Production needs AUTH — anyone on the network can hit it.
  • Endpoint usually at /mcp, not root /.

----------------------------------------------------------------------


4. SSE TRANSPORT — "LEGACY HTTP, DUAL-ENDPOINT"
----------------------------------------------------------------------

How it works (the OLD way):
  ┌──────────┐                      ┌─────────────────┐
  │  Client  │                      │     Server      │
  │          │                      │                 │
  │  POST ───┼─── messages ────────►│  /messages      │  (client → server)
  │          │                      │                 │
  │  GET  ───┼─── long-lived ──────►│  /sse           │  (server → client
  │          │ ◄── stream events    │                 │   long stream)
  └──────────┘                      └─────────────────┘

Properties:
  • TWO endpoints (/sse + /messages) instead of one.
  • Always requires a long-lived GET for receiving.
  • Awkward behind load balancers and proxies.
  • Reconnect logic is painful.

Status:
  ⚠️ LEGACY — kept only for backwards compatibility.
  ⚠️ Replaced by streamable-http in modern MCP.
  ⚠️ Only use if you must integrate with an older server.

----------------------------------------------------------------------


5. SIDE-BY-SIDE COMPARISON
----------------------------------------------------------------------

  ┌──────────────────┬──────────────┬──────────────────┬──────────────┐
  │     Aspect       │    stdio     │ streamable-http  │      sse     │
  ├──────────────────┼──────────────┼──────────────────┼──────────────┤
  │ Where server runs│ Child of cli │  Anywhere        │  Anywhere    │
  │ Who starts it    │ The client   │  You manually    │  You manually│
  │ Communication    │ stdin/stdout │  HTTP + SSE      │  HTTP + SSE  │
  │ Endpoints        │  N/A         │  1  (/mcp)       │  2 (/sse +   │
  │                  │              │                  │   /messages) │
  │ Lifetime         │ Dies w/ cli  │  Persistent      │  Persistent  │
  │ # of clients     │ One          │  Many            │  Many        │
  │ Network needed?  │  ❌          │  ✅              │  ✅          │
  │ Auth needed?     │ Local trust  │  Yes in prod     │  Yes in prod │
  │ Status           │ ✅ Current   │  ✅ Current      │  ⚠️ Legacy   │
  │ Best for         │ Local tools  │  Remote/cloud    │  Compat only │
  └──────────────────┴──────────────┴──────────────────┴──────────────┘

----------------------------------------------------------------------


6. CUSTOM TRANSPORTS
----------------------------------------------------------------------

Spec allows other transports (not officially standardized):

  ┌────────────────────────┬─────────────────────────────────────┐
  │  Custom Transport      │  How it works                        │
  ├────────────────────────┼─────────────────────────────────────┤
  │ WebSocket              │ Full-duplex socket, bidirectional   │
  │ Unix domain sockets    │ Like stdio but via socket file      │
  │ TCP raw socket         │ Plain TCP for embedded / IoT        │
  │ Named pipes (Windows)  │ Windows IPC equivalent of UDS       │
  │ In-process / memory    │ Same-process, used for unit tests   │
  └────────────────────────┴─────────────────────────────────────┘

Not in the official spec — you rarely need them.

----------------------------------------------------------------------


7. ONE-LINE MENTAL MODELS
----------------------------------------------------------------------

  stdio            =  "USB cable between two devices in one room"
  streamable-http  =  "Wi-Fi router — many devices, anywhere"
  sse              =  "Old two-cable telephone system"
  custom           =  "Build your own road"

Or in software terms:

  stdio            =  CLI tool (runs, does its thing, exits)
  streamable-http  =  Microservice (runs forever, serves requests)
  sse              =  Legacy long-polling pattern
  custom           =  Bespoke channel

----------------------------------------------------------------------


8. WHAT "STREAMABLE" MEANS
----------------------------------------------------------------------

Normal HTTP:
  Client: "Give me X"
  Server: ... waits ... computes ... waits ...
  Server: "Here's the FULL answer!"   (one shot at end)

Streamable HTTP:
  Client: "Give me X"
  Server: "Working on step 1..."    ← chunk 1
  Server: "Working on step 2..."    ← chunk 2
  Server: "Almost done..."          ← chunk 3
  Server: "Final answer: ..."       ← chunk 4 (done)

Server can push INCREMENTAL updates without finishing the
request. Built on top of HTTP + Server-Sent Events (SSE).

Useful for:
  • Long-running tools (give progress updates)
  • Tools producing output gradually (LLM token streaming)
  • Server-initiated notifications

----------------------------------------------------------------------


9. SAME TOOL CODE, DIFFERENT TRANSPORT
----------------------------------------------------------------------

The whole point of MCP's design — change ONE LINE:

  ┌──────────────────────────────┬──────────────────────────────┐
  │  stdio_server.py             │  http_server.py              │
  ├──────────────────────────────┼──────────────────────────────┤
  │ from fastmcp import FastMCP  │ from fastmcp import FastMCP  │
  │ mcp = FastMCP()              │ mcp = FastMCP()              │
  │                              │                              │
  │ @mcp.tool()                  │ @mcp.tool()                  │
  │ async def fetch():           │ async def fetch_http():      │
  │     ...                      │     ...                      │
  │                              │                              │
  │ mcp.run(                     │ mcp.run(                     │
  │   transport="stdio"          │   transport="streamable-     │
  │ )                            │              http",          │
  │                              │   host="0.0.0.0",            │
  │                              │   port=8050                  │
  │                              │ )                            │
  └──────────────────────────────┴──────────────────────────────┘
        ▲                              ▲
        │                              │
  95% identical                   Only run() differs!

Same applies on the client side — only the import + connection
opener changes; the ClientSession code is identical.

----------------------------------------------------------------------


10. WHICH TRANSPORT TO USE — DECISION GUIDE
----------------------------------------------------------------------

  ┌─────────────────────────────────────┬─────────────────────┐
  │           Your scenario             │   Transport choice  │
  ├─────────────────────────────────────┼─────────────────────┤
  │ Local file/git/shell tool           │  stdio              │
  │ Personal Claude Desktop server      │  stdio              │
  │ Multi-user / shared service         │  streamable-http    │
  │ Cloud-hosted MCP server             │  streamable-http    │
  │ Heavy/stateful server (caches, DBs) │  streamable-http    │
  │ Integrating with old MCP server     │  sse                │
  │ Embedded / IoT / niche              │  custom             │
  └─────────────────────────────────────┴─────────────────────┘

Rule of thumb when building TODAY:
  Local?     → stdio
  Networked? → streamable-http
  sse only if forced by legacy compat.


═══════════════════════════════════════════════════════════════════
  COMPONENT SUMMARY TABLE — TRANSPORTS
═══════════════════════════════════════════════════════════════════

  ┌──────────────────────────┬──────────────────────────────────┐
  │ Component                │ What it does                     │
  ├──────────────────────────┼──────────────────────────────────┤
  │ Transport (concept)      │ Physical channel for MCP msgs;   │
  │                          │ swappable under the protocol.    │
  ├──────────────────────────┼──────────────────────────────────┤
  │ stdio                    │ Pipes via stdin/stdout to a      │
  │                          │ child subprocess. Local only.    │
  ├──────────────────────────┼──────────────────────────────────┤
  │ streamable-http          │ Modern HTTP, single /mcp endpoint│
  │                          │ with optional SSE streaming.     │
  │                          │ Recommended for remote servers.  │
  ├──────────────────────────┼──────────────────────────────────┤
  │ sse                      │ Legacy HTTP w/ 2 endpoints       │
  │                          │ (/sse GET + /messages POST).     │
  │                          │ Deprecated; backwards compat.    │
  ├──────────────────────────┼──────────────────────────────────┤
  │ Custom (WS, UDS, TCP…)   │ Allowed but not standardized;    │
  │                          │ niche / embedded use cases.      │
  ├──────────────────────────┼──────────────────────────────────┤
  │ streamablehttp_client    │ Python SDK helper to open an     │
  │                          │ HTTP transport for the client.   │
  ├──────────────────────────┼──────────────────────────────────┤
  │ sse_client               │ SDK helper for the legacy SSE    │
  │                          │ transport on the client side.    │
  ├──────────────────────────┼──────────────────────────────────┤
  │ mcp.run(transport=...)   │ FastMCP server entry point.      │
  │                          │ Changing this string switches    │
  │                          │ between stdio / http / sse.      │
  └──────────────────────────┴──────────────────────────────────┘


═══════════════════════════════════════════════════════════════════
  KEY TAKEAWAYS — SESSION 2
═══════════════════════════════════════════════════════════════════

  • MCP has 3 official transports: stdio, streamable-http, sse.
  • Custom transports (WebSocket, UDS, TCP) are possible but rare.
  • stdio  = local subprocess + pipes; dies with client.
  • streamable-http = modern HTTP, single /mcp endpoint, multi-client.
  • sse = legacy dual-endpoint HTTP; only for backwards compat.
  • "Streamable" means the server can push incremental chunks
    instead of one final answer.
  • The MCP DESIGN PRINCIPLE:
       protocol = WHAT to say
       transport = HOW to say it
  • Same tool code works across all transports — only the
    mcp.run(transport=...) line changes.
  • Rule of thumb today:
       local tool       → stdio
       networked tool   → streamable-http
       legacy server    → sse
       exotic / IoT     → custom

═══════════════════════════════════════════════════════════════════

```



---
---

