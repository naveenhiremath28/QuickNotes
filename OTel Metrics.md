
## What is OTel Metrics?

OpenTelemetry Metrics is the observability signal for numeric measurements of your system over time. It is vendor-neutral — instrument once, export anywhere (Prometheus, Datadog, Grafana Cloud, etc.).

**Pipeline:** `Your code → SDK / MeterProvider → MetricReader → Exporter → Backend`

OTel has three signals: Metrics, Traces, Logs. Metrics = time-series numbers.

## Counter

A Counter measures **how many times something happened**. It only ever goes up. When your process restarts, it resets to zero — backends detect this as a "reset" and handle it gracefully.

**The question it answers:**

> "How many times did X happen — total, and per second?"

**What it stores internally:**

- A single cumulative number that increases monotonically

**What you can ask at query time:**

- Total count since start → `http_requests_total`
- Rate over a window → `rate(http_requests_total[5m])` — requests per second averaged over 5 min
- Spike detection → `irate(http_requests_total[1m])` — instantaneous rate

**What it cannot answer:**

- How long did each request take?
- Were most requests fast or slow?
- What was the worst case?

**Real examples:**

- Total HTTP requests served
- Number of errors thrown
- Bytes sent over the network
- Cache hits and misses
- Retries attempted

**Mental model:** Think of a car's odometer. It only goes up. You can calculate speed (rate of change) but you cannot know if the last mile was highway or traffic.

---

## UpDownCounter

An UpDownCounter measures **the current size of something that fluctuates**. Unlike a Counter it can go negative. You don't set an absolute value — you record changes (deltas), and the sum gives you the current state.

**The question it answers:**

> "How much of X do I have right now, given it grows and shrinks?"

**What it stores internally:**

- A running sum of all deltas — positive and negative

**What you can ask at query time:**

- Current value → `queue_depth` (the sum of all `add()` calls)
- Trend over time → is the queue growing or draining?

**What it cannot answer:**

- Rate (meaningless — it goes up and down)
- Percentiles or distribution
- Peak value (you'd need a Gauge for that)

**Real examples:**

- Active HTTP connections (`+1` on connect, `-1` on disconnect)
- Jobs in a queue (`+1` enqueue, `-1` dequeue)
- Items currently in a cache
- In-flight database transactions
- Goroutines / threads alive

**Mental model:** Think of a bank account balance. Every deposit is `+n`, every withdrawal is `-n`. The balance is the running sum. You'd never call `rate()` on a bank balance — that's not meaningful.

**Gauge vs UpDownCounter — the subtle difference:**

||UpDownCounter|Gauge|
|---|---|---|
|You record|Deltas — the change|Absolute value — the current reading|
|Example|`add(+1)` when job enqueues|`observe(42)` after polling `/proc/meminfo`|
|Use when|You control the changes|You're reading an external value|

If you own the increment/decrement logic → UpDownCounter. If you're polling something external → Gauge.

---

## Gauge

A Gauge captures **what the value is right now**. There is no accumulation. Every new observation simply overwrites the previous one. It's a snapshot, not a running total.

**The question it answers:**

> "What is the current value of X at this moment?"

**What it stores internally:**

- Only the most recent observed value

**What you can ask at query time:**

- Current reading → `system_cpu_utilization`
- Is it above a threshold? → alerting rules
- How has it trended? → backends graph the sequence of snapshots over time

**What it cannot answer:**

- How many times X happened
- Rate of change (you can approximate it but it's not meaningful semantically)
- Distribution or percentiles

**Real examples:**

- CPU utilization (%)
- Memory used (bytes)
- Disk space remaining
- Temperature of a sensor
- Number of active users (if read from a DB, not tracked by your code)
- JVM heap size

**Mental model:** Think of a thermometer. It tells you the temperature right now. The previous reading is gone. You can plot readings over time to see a trend, but the instrument itself only knows the current value.

**Observable Gauge pattern** — gauges are almost always implemented as callbacks because the value is polled, not pushed:

js

```js
meter.createObservableGauge('system.memory.used', { unit: 'bytes' })
  .addCallback(result => {
    // called each collection cycle
    result.observe(process.memoryUsage().heapUsed, { type: 'heap' });
  });
```

---

## Histogram

A Histogram measures **the distribution of a value across many events**. Instead of storing every individual measurement, it sorts values into predefined buckets and tracks how many fell into each. It also tracks the total count and sum so you can compute averages, and from the buckets you can estimate percentiles.

**The question it answers:**

> "What does the spread of X look like — what's typical, what's the worst case, and where are the outliers?"

**What it stores internally:**

- `count` — total number of observations
- `sum` — total of all observed values
- `min` / `max` — lowest and highest seen
- Bucket counts — how many observations fell into each range

**What you can ask at query time:**

- p50 (median) → half of requests were faster than this
- p95 → 95% of requests were faster than this
- p99 → the worst 1% of requests were slower than this
- Average → `sum / count`
- Throughput → `rate(histogram_count[5m])`

**What it cannot answer:**

- The exact value of any individual observation (it's bucketed, not raw)
- Percentiles more precise than your bucket boundaries allow

**Real examples:**

- HTTP request duration (ms)
- Database query time (ms)
- Message queue wait time (ms)
- Request payload size (bytes)
- Response body size (bytes)
- Time to first byte

**Mental model:** Imagine you ran a 100m race for 1000 people and instead of recording each person's exact time, you put a tally mark on a board divided into lanes: "under 10s", "10–12s", "12–15s", "over 15s". You lose the exact times but you can immediately see whether most people were fast or slow, and where the stragglers are.

**Why averages alone are dangerous:**

```
1000 requests:
  999 completed in 10ms
    1 completed in 10,000ms (10 seconds)

Average = (999×10 + 1×10000) / 1000 = 19.99ms  ← looks fine
p99     = 10,000ms                               ← one user in a hundred is suffering
```

The average said everything was fine. The histogram revealed the problem.

**Bucket boundaries matter** — choose them to match your SLO thresholds:

js

```js
// For an API with a 200ms SLO
meter.createHistogram('http.request.duration', {
  unit: 'ms',
  boundaries: [5, 10, 25, 50, 100, 200, 500, 1000, 2000],
  //                               ^^^  ← your SLO boundary
});
```

Put a boundary exactly at your SLO threshold so you can directly query what percentage of requests met it.

---

## Summary — What Each Instrument Answers

|Instrument|Core question|Aggregation|Rate query?|Percentiles?|
|---|---|---|---|---|
|Counter|How many times did X happen?|Sum (↑ only)|Yes|No|
|UpDownCounter|How much of X is there right now?|Sum (↑↓)|No|No|
|Gauge|What is X at this exact moment?|LastValue|No|No|
|Histogram|What does the spread of X look like?|Buckets + sum + count|Yes (on count)|Yes|

---

## The RED Method — putting it together

For any service, these three metrics tell you almost everything:

```
Rate     → Counter    → how many requests/sec is this service handling?
Errors   → Counter    → how many are failing?
Duration → Histogram  → how long are they taking, and what's the p99?
```

Counter vs Histogram:

**A Counter counts occurrences of an event.** **A Histogram measures a value _attached to_ each occurrence.**

---

Think of it this way. Every time a request comes in, two things exist:

1. The fact that a request happened — **that's a Counter**
2. How long that request took — **that's a Histogram**

```
Request arrives
    │
    ├── counter.add(1)              ← "one more request happened"
    │
    └── histogram.record(42)        ← "this one took 42ms"
```

The Counter only knows _that_ something happened. It has no idea about the 42ms. The Histogram only knows _how long_each thing took. It doesn't care about the count on its own.

---

**The deeper difference — what the numbers mean:**

||Counter|Histogram|
|---|---|---|
|What you pass in|Always `1` (or a fixed increment)|The actual measured value — `42`, `130`, `7`|
|What accumulates|The total count|A distribution of values across buckets|
|Useful query|"How many requests per second?"|"What was the p99 latency?"|

With a Counter you call `add(1)` — you're just ticking a tally. With a Histogram you call `record(42)` — you're submitting a measurement.

---

**Where people get confused** is that a Histogram _also_ internally maintains a count. So you get request rate _for free_ from a histogram too — `rate(histogram_count[5m])`. But a Counter cannot give you percentiles, ever. So in practice:

- If you only care about _how many_ → Counter, it's cheaper
- If you care about _how long / how big_ → Histogram, it gives you everything

---

**Simplest way to remember it:**

> Counter = did it happen? How often?
> 
> Histogram = when it happened, what was the measurement?