



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

---
## ⚡ Automatic Instrumentation: Where ClickStack Really Shines

Here’s where ClickStack starts to feel different from most observability stacks.

Yes, it speaks OpenTelemetry natively.  
Yes, it accepts traces, metrics, and logs via standard OTLP.

But the real magic is **how little effort it takes to get meaningful visibility**.

To understand why that matters, let’s look at a simple Go service built with Fiber.

---

## Life Without Automatic Instrumentation

Without automatic instrumentation, observability slowly creeps into your application code.

You don’t just write business logic anymore —  
you write spans, manage context, and manually connect logs to traces.

Even a simple Fiber route starts to look like this:

`app.Get("/hello", func(c *fiber.Ctx) error { 	tracer := otel.Tracer("my-service")  	ctx, span := tracer.Start(context.Background(), "GET /hello") 	defer span.End()  	log.Info("processing hello request")  	time.Sleep(100 * time.Millisecond)  	span.SetAttributes( 		attribute.String("http.method", "GET"), 		attribute.String("http.route", "/hello"), 	)  	return c.SendString("hello world") })`

At first glance, this feels manageable.

But look closely at what’s happening:

- You manually create a span
    
- You manage the span lifecycle
    
- You must ensure logs run _inside_ the span context
    
- You rely on every developer following the same pattern
    

Miss any of these steps, and:

- Logs lose trace context
    
- Requests become harder to debug
    
- Observability becomes inconsistent
    

Nothing here is _wrong_ — but none of it is business logic either.

Multiply this across dozens of routes, and observability starts to feel like a tax.

---

## Enter Automatic Instrumentation

This is where ClickStack’s design really pays off.

Instead of instrumenting _your code_, you instrument _the framework_.

In practice, this means using OpenTelemetry’s auto-instrumentation libraries together with ClickStack’s OpenTelemetry-native ingestion.

For Go services, this typically includes:

`go get -u go.opentelemetry.io/otel go get -u github.com/hyperdxio/otel-config-go go get -u github.com/hyperdxio/opentelemetry-go go get -u github.com/hyperdxio/opentelemetry-logs-go`

And framework-level instrumentation, for example:

`go get -u go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp`

These libraries automatically:

- Create HTTP spans
    
- Propagate request context
    
- Export traces, logs, and metrics
    

---

## One Middleware, Clean Handlers

With instrumentation in place, Fiber itself becomes observable.

There is **one important nuance** to be aware of:

> Auto-instrumentation creates and propagates context,  
> but logs are only linked to traces if the logger reads trace metadata from that context.

This does **not** require manual spans or extra instrumentation — just a small helper.

---

### Attach Trace Metadata to Logs (Defined Once)

`func WithTraceMetadata(ctx context.Context, logger *zap.Logger) *zap.Logger { 	spanContext := trace.SpanContextFromContext(ctx) 	if !spanContext.IsValid() { 		return logger 	}  	return logger.With( 		zap.String("trace_id", spanContext.TraceID().String()), 		zap.String("span_id", spanContext.SpanID().String()), 	) }`

---

### Auto-Instrumented Fiber Handler (Final Form)

`app := fiber.New() app.Use(otelfiber.Middleware())  app.Get("/hello", func(c *fiber.Ctx) error { 	logger := WithTraceMetadata(c.Context(), zap.L())  	logger.Info("processing hello request")  	time.Sleep(100 * time.Millisecond) 	return c.SendString("hello world") })  app.Listen(":8080")`

That’s it.

No spans.  
No tracer lifecycle.  
No attribute management.

HTTP traces are created automatically, and logs written inside the request are **linked to the same trace**.

---

## Why This Log Line Is the Key Difference

In both approaches, you wrote the same log:

`log.Info("processing hello request")`

But the outcome is very different.

### Without auto-instrumentation

- You must ensure the log runs inside an active span
    
- You rely on discipline and conventions
    
- One missed context → broken correlation
    

### With auto-instrumentation + context-aware logging

- The log automatically inherits the request context
    
- It carries the same trace and span IDs
    
- It’s linked to the HTTP request in ClickStack
    
- No manual trace ID handling required
    

You didn’t:

- Pass trace IDs explicitly
    
- Modify your business logic
    
- Add tracing code to your handlers

The context was already there — you just used it.

---

## What You Get — By Default

With a single middleware and a context-aware logger, ClickStack automatically receives:

- HTTP request traces
    
- Route names and HTTP methods
    
- Latency and error information
    
- Logs linked to the same trace
    
- Consistent context propagation
    

All of this happens **by default**, not by convention.

---

## Shipping Telemetry Is Still Standard OpenTelemetry

Nothing proprietary is happening here.

Telemetry flows into ClickStack using standard OpenTelemetry environment variables:

`export OTEL_EXPORTER_OTLP_ENDPOINT=https://localhost:4318 \ OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf \ OTEL_SERVICE_NAME='<NAME_OF_YOUR_APP_OR_SERVICE>' \ OTEL_EXPORTER_OTLP_HEADERS='authorization=<YOUR_INGESTION_API_KEY>'`

Your API key lives in the HyperDX UI under **Team Settings → API Keys**.

From that point on, traces, logs, and metrics start flowing automatically.

---

## Why This Changes the Observability Experience

Automatic instrumentation quietly changes everything:

- Observability is no longer something you “remember to add”
    
- Logs and traces are correlated automatically
    
- New services are observable from day one
    
- Your code stays focused on what matters
    

That’s the real win.

ClickStack doesn’t just collect telemetry —  
it removes the friction that usually prevents teams from doing observability well.

And once observability stops feeling like work, it becomes indispensable

---
## 🧠 Real-World Use Cases

Where does ClickStack shine? Everywhere telemetry matters.

📍 **Fast Root Cause Analysis** — Correlate logs with traces and metrics instantly.  
📍 **High-Cardinality Observability** — Handle massive amounts of telemetry without performance woes.  
📍 **Real-Time Alerting** — Spot anomalies and visualize trends as they happen.  
📍 **Session + Server Correlation** — See a user session _and_ the backend traces it generated — all connected.







Elaborate shipping and add images
