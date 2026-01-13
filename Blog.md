

# **CLICKSTACK**


1. clickstack explanation
2. Components it has
3. Features of clickstack - https://clickhouse.com/docs/use-cases/observability/clickstack/architecture
4. Ingesting with OpenTelemetry
5. automatic instrumentation - https://clickhouse.com/docs/use-cases/observability/clickstack/sdks/golang
```export OTEL_EXPORTER_OTLP_ENDPOINT=https://localhost:4318 \  
OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf \  
OTEL_SERVICE_NAME='<NAME_OF_YOUR_APP_OR_SERVICE>' \  
OTEL_EXPORTER_OTLP_HEADERS='authorization=<YOUR_INGESTION_API_KEY>'
```




# 🚀 _Journey into ClickStack: The Modern Observability Story You Actually Want to Read_

You know that moment when your app breaks in production?  
Logs scattered across systems. Traces tucked away in another silo. Metrics buried in some other dashboard you forgot how to log into. Your team is ping-ponging between tools trying to connect the dots… and your SLA is screaming. 🎯

That was me, not too long ago—lost in a sea of observability tools that _barely talked to each other_. Until I found something refreshingly simple yet powerful: **ClickStack**.

Let me take you on a story from confusion to clarity.

---
## 🧠 What is ClickStack — A Unified Observability Universe

In the simplest terms:

**ClickStack is a production-grade observability platform that unifies logs, traces, metrics, and session data in one place — powered by ClickHouse and OpenTelemetry.**

Instead of treating observability signals as separate silos, ClickStack brings them together into a **single, coherent story**.

You’re no longer asking:

- “Which dashboard should I open?”
    
- “Where do I correlate this trace with logs?”
    
- “Why does nothing line up during incidents?”
    

With ClickStack, everything already speaks the same language.

---
## 🧩 ClickStack’s Architecture — The Cast of Characters

Behind the scenes, ClickStack is composed of a few elegant building blocks. Each is essential to the story:

### 🟡 **ClickHouse** — The Heart of the Stack

This column-oriented analytical database sits at the core. It’s _fast_, _efficient_, and built for observability workloads. Think sub-second querying even on massive datasets. 

### 🟢 **OpenTelemetry Collector** — The Gateway In

This collector is the entry point for all telemetry — logs, metrics, traces, and session data. It uses the **OpenTelemetry (OTel) standard** to receive data from your apps and push it into ClickHouse. 

### 🔵 **HyperDX UI** — The Dashboard & Explorer

HyperDX is the purpose-built frontend where your observability world comes to life. Search, filter, visualize, and correlate — all without context switching. 

Together, these pieces create one seamless stack — easy to deploy, open-source, and built for scale.

## ✨ What Makes ClickStack Special

Here’s where ClickStack shines — the features that make engineers _actually enjoy_ observing their systems:

✅ Unified logs, metrics, traces, and session replay in a **single place** — _no silos_.   
✅ Sub-second search and analytics even over **petabytes of telemetry**.   
✅ **Natural language or SQL querying**, whichever fits your brain.   
✅ Built-in dashboards, alerts, and correlation — minimal setup needed.   
✅ Fast, efficient storage with **high compression and low cost**. 

In short: _ClickStack doesn’t just store data — it makes it _useful_._ It was built from the ground up to help _you find answers fast_ — not just pile data into a black hole.

## 🔌 Ingesting Telemetry with OpenTelemetry

One of the best design decisions in ClickStack is that it speaks _OpenTelemetry natively_. That means you can instrument your apps with standard OpenTelemetry SDKs — and send everything into ClickStack with **zero vendor lock-in**. 

The typical ingestion setup looks like this:

`export OTEL_EXPORTER_OTLP_ENDPOINT=https://localhost:4318 \ OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf \ OTEL_SERVICE_NAME='<NAME_OF_YOUR_APP_OR_SERVICE>' \ OTEL_EXPORTER_OTLP_HEADERS='authorization=<YOUR_INGESTION_API_KEY>'`

This tells your application how to send telemetry into ClickStack through the OpenTelemetry pipeline — simple, standardized, reliable.

---

## 🚀 Automatic Instrumentation — Your Time Saver

And now, _the part every developer loves_.

ClickStack doesn’t just accept telemetry — it works with **SDKs that provide automatic instrumentation**. That’s a game changer.

With automatic instrumentation, you don’t have to manually insert tracing and metrics calls everywhere. Instead:

✨ Your Go, Node, Python, or other apps get instrumented with minimal code changes.  
✨ The SDK captures spans, HTTP requests, database calls, and metrics for you.  
✨ You start seeing traces, logs, and metrics flowing into ClickStack _without writing boilerplate_.

Why does that matter?

👉 You spend **less time instrumenting** and more time **understanding what’s happening in production**.  
👉 You avoid the dreaded “it works on my machine” debugging dance.  
👉 You get richer telemetry _out of the box_ without dancing between docs and examples.

Automatic instrumentation doesn’t just save time — it elevates your whole observability experience.

---

## 🧠 Real-World Use Cases

Where does ClickStack shine? Everywhere telemetry matters.

📍 **Fast Root Cause Analysis** — Correlate logs with traces and metrics instantly.  
📍 **High-Cardinality Observability** — Handle massive amounts of telemetry without performance woes.  
📍 **Real-Time Alerting** — Spot anomalies and visualize trends as they happen.  
📍 **Session + Server Correlation** — See a user session _and_ the backend traces it generated — all connected.