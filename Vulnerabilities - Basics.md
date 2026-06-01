# Vulnerabilities — Basics

## The one-sentence version

> **A vulnerability is a known mistake in someone else's code that a hacker can use to attack you.**

That's it. The rest is detail.

---

## A real-world analogy

Imagine you live in an apartment building. You didn't build it. The builder did.

Years later, someone discovers: *"The lock on apartment #305's door can be opened with a paperclip."*

That's a **vulnerability**.

- It's a **weakness** — the lock is broken
- It's **publicly known** — written up somewhere
- It affects **you**, even though you didn't build the lock
- The **fix** is for the builder to ship a new lock

You have three choices:
1. Wait for the builder to send a new lock and install it (**upgrade**)
2. Add your own deadbolt over the broken lock (**override / patch**)
3. Decide it doesn't matter because you never use that door (**accept the risk**)

That's literally what software vulnerability management is.

---

## How this maps to code

| Apartment world | Code world |
|---|---|
| Your apartment | Your app |
| The building | The packages your app uses (Express, React, etc.) |
| The broken lock | A bug in one of those packages |
| The newspaper warning | An advisory (CVE / GHSA / RUSTSEC) |
| Installing a new lock | Upgrading the package version |
| Putting on a deadbolt | Adding an override |
| "I never use that door" | "That code never runs in my app" |

---

## A concrete example

We found this on `finternet-app`:

> *"The `qs` package, versions 6.11.1 to 6.15.1, has a bug. If someone sends a specially-shaped HTTP request, it crashes the server."*

Translated:
- **Where:** The `qs` library — used by Express to parse URL parameters
- **What an attacker can do:** Crash your server with one weird request
- **Who's affected:** Every app using those versions
- **The fix:** Use version 6.15.2 or newer
- **What we did:** Forced our app to use a newer version via an override

Result: the door now has a working lock.

---

## Why you didn't write the bug but it's still your problem

Your `package.json` has maybe 20 libraries you chose. Those 20 libraries each use 30 more. Each of those uses another 30. By the time you `npm install`, you've actually got **thousands** of small packages in your app, all written by strangers.

Any one of them can have a bug.

You're responsible for the whole pile, because they're all running inside your app.

---

## Severity — how worried should you be?

| Word | What it really means |
|---|---|
| **Critical** | "A stranger on the internet can take over your server right now." Fix today. |
| **High** | "A stranger on the internet can crash your server or steal data." Fix this week. |
| **Medium / Moderate** | "Something bad could happen, but the attacker needs specific conditions." Fix when you can. |
| **Low** | "Theoretically a problem, in practice rarely matters." Fix eventually. |

---

## The 3 reactions to a vulnerability

When a scanner says "you have a vulnerability":

### Fix it — most common
Update the package. Done. (Examples: `turbo`, `qs`, `ws`, `uuid`, `openssl`, `protobufjs`, `golang.org/x/net`.)

### Patch it locally — when there's no upstream fix
Apply your own custom fix to the broken code. (Example: `bigint-buffer` via `pnpm.patchedDependencies`.)

### Accept the risk — when the bug exists but can't actually be triggered
Document why the broken code can't be reached. Move on. (Example: OpenTelemetry packages — the bug needs a Prometheus HTTP server running, and HyperDX uses OTLP instead, so the vulnerable code never runs.)

---

## Why everyone tracks vulnerabilities

Three reasons, in plain English:

1. **Hackers really do use these.** Most successful hacks today aren't clever zero-days — they're "you forgot to update a library for 8 months and we walked in."
2. **Auditors will ask.** SOC 2, ISO 27001, customer security reviews — they all want to see "you know about these and you have a process."
3. **Insurance demands it.** Cyber insurance policies require you to patch known vulnerabilities within a deadline.

---

## Common categories of vulnerabilities

| Category | What it does | Example seen this session |
|---|---|---|
| **Denial of Service (DoS)** | Crash the app | `qs.stringify` crash on malformed input |
| **Buffer overflow** | Corrupt memory, sometimes leads to code execution | `bigint-buffer` `toBigIntLE()` |
| **Information disclosure** | Leak data the attacker shouldn't see | `ws` uninitialized memory disclosure |
| **Authentication bypass / session fixation** | Log in as someone else | `turbo` login callback CSRF |
| **Remote code execution (RCE)** | Run arbitrary commands on your server | `turbo` Yarn Berry detection |
| **Timing side-channel** | Leak secrets by measuring how long crypto takes | `rsa` Marvin Attack |

---

## Tools that find vulnerabilities

| Tool | What it scans | What it reads |
|---|---|---|
| **Dependabot** (GitHub) | Your dependency tree | GitHub Advisory Database (GHSA-xxx) |
| **npm audit** | Node packages | npm registry advisories |
| **govulncheck** (Go) | Go packages, with **call-graph awareness** — only flags vulns whose code is actually called | Go vulnerability database |
| **cargo audit** (Rust) | Rust crates | RustSec advisory DB (RUSTSEC-yyyy-nnnn) |
| **Vanta** | Aggregates from GitHub + others into compliance dashboards | Multiple sources |

`govulncheck` is the smartest of these — it doesn't just check version numbers, it traces which functions you actually call. So a vulnerable module sitting in your dependency tree but never called by your code is reported as **latent** (not blocking).

---

## Vulnerability IDs — what the codes mean

| Format | Source | Example |
|---|---|---|
| `CVE-YYYY-NNNNN` | MITRE / NVD — the universal industry ID | CVE-2025-3194 |
| `GHSA-xxxx-xxxx-xxxx` | GitHub Advisory — used by Dependabot, npm audit | GHSA-3gc7-fjrx-p6mg |
| `RUSTSEC-YYYY-NNNN` | RustSec database — used by `cargo audit` | RUSTSEC-2023-0071 |
| `GO-YYYY-NNNN` | Go vulnerability database — used by `govulncheck` | GO-2026-5026 |

The same vulnerability often has multiple IDs (one CVE + one GHSA, etc.). They cross-reference each other.

---

## TL;DR — the whole concept in 4 bullets

- A vulnerability = **a known bug that an attacker can abuse**
- It lives in **other people's code** that your app depends on
- It has an **ID** (so everyone refers to the same thing), a **severity** (so you know how scared to be), and usually a **fix** (a newer version)
- Your job: **find them → fix them → keep records**

Everything else — the CVEs, GHSAs, CVSS scores, severity levels, Dependabot, npm audit, Vanta — is just **plumbing** to do those four things at scale.

---

## Worked examples from the Finternet codebase

### Fixed by version bump
| Vulnerability | Where | Fix |
|---|---|---|
| GO-2026-5026, GO-2026-4918 (`golang.org/x/net`) | units-api | `go get golang.org/x/net@v0.55.0` |
| `turbo` ≤2.9.13 (CSRF + Yarn RCE) | finternet-app | bump turbo to ^2.9.14 |
| `openssl` 0.10.79 → 0.10.80 | units-services/proofService | `cargo update -p openssl --precise 0.10.80` |

### Fixed by pnpm/npm override (transitive deps)
| Package | Vulnerability | Fix |
|---|---|---|
| `qs` 6.11.1–6.15.1 | DoS in stringify | override `qs: ">=6.15.2"` |
| `ws` <8.20.1 | Uninitialized memory | override `ws: ">=8.20.1"` |
| `uuid` <11.1.1 | Bounds check | override `uuid: ">=11.1.1"` |
| `protobufjs` ≤7.5.7 | DoS via recursive JSON | override `protobufjs: "^7.6.0"` |

### Patched locally (no upstream fix)
| Package | Vulnerability | Mitigation |
|---|---|---|
| `bigint-buffer` ≤1.1.5 | Buffer overflow in `toBigIntLE()` | `pnpm.patchedDependencies` → `patches/bigint-buffer@1.1.5.patch` |

### Risk-accepted (vulnerable code path never runs)
| Cluster | Vulnerability | Justification |
|---|---|---|
| OpenTelemetry trio (sdk-node, auto-instrumentations-node, exporter-prometheus) + @hyperdx/node-opentelemetry | GHSA-q7rr-3cgh-j5r3 — Prometheus exporter HTTP server crash | HyperDX uses OTLP transport, never starts the Prometheus HTTP server. CodeQL confirms zero calls to vulnerable functions. Dismissed in Dependabot with documented reasoning. |
| `rsa` crate via `sqlx` | RUSTSEC-2023-0071 (Marvin Attack timing sidechannel) | TLS terminates upstream of proofService; no untrusted RSA-decrypt path in our code. Allow-listed in cargo-audit by exact ID. |

---

## Quick reference: how to find vulnerabilities locally

```bash
# Node (in any pnpm project)
pnpm audit

# Node (in any npm project)
npm audit

# Go
go install golang.org/x/vuln/cmd/govulncheck@latest
govulncheck ./...

# Rust
cargo install cargo-audit
cargo audit
```

## Quick reference: how to fix common cases

```jsonc
// package.json — force a transitive dep to a patched version (npm + pnpm)
{
  "overrides": {                  // npm
    "qs": ">=6.15.2"
  },
  "pnpm": {                       // pnpm (root package.json)
    "overrides": {
      "qs": ">=6.15.2",
      "ws": ">=8.20.1"
    },
    "patchedDependencies": {
      "bigint-buffer@1.1.5": "patches/bigint-buffer@1.1.5.patch"
    }
  }
}
```

```bash
# Go — bump a single module
go get golang.org/x/net@v0.55.0
go mod tidy

# Rust — bump a single crate without changing Cargo.toml
cargo update -p openssl --precise 0.10.80
```

---

## What do hackers actually *get* by exploiting a vulnerability?

Crashing the server is just one tactic. The real prizes, roughly in order of damage:

| Attack outcome | What the attacker walks away with |
|---|---|
| **Take you offline (DoS)** | Extortion ("pay or we keep crashing you"), competitor sabotage, distraction cover for a quieter attack |
| **Steal data** | User passwords, API keys, financial info, internal secrets — sold on dark-web markets |
| **Hijack accounts** | Log in as someone else (or an admin) → drain money, read messages, plant backdoors |
| **Run their own code (RCE)** | The holy grail — full server takeover. Ransomware, cryptojacking, pivot into your network |
| **Steal crypto keys via timing** | Silently forge signatures and decrypt traffic forever, without tripping any alarms |

### How they cash in
- Stolen credit cards: **$5–$100 each** on dark web
- Medical records: **$250–$1000 each**
- Ransomware demands: **$50K–$10M**
- Cryptojacking: **~$100–$1000/month per server**
- Stolen credentials → reused on banks, email, AWS

### What it costs you (the victim)
- **Direct:** incident response, legal, ransom payments
- **Regulatory:** GDPR fines up to **4% of global revenue**, SOC 2 audit failures
- **Reputation:** customers leave; B2B deals stall
- **Time:** weeks/months of security team cleanup

---

## Why do new versions *also* keep getting new vulnerabilities?

**The list never empties.** Five reasons:

1. **The bug was always there — someone just discovered it now.** Researchers find years-old flaws daily. Code "safe yesterday" can be flagged tomorrow because the weakness was never noticed before.
2. **New features = new bugs.** Every release adds code. New code can have new bugs. Sometimes the fix for one bug introduces another.
3. **Attackers get smarter.** Crypto safe in 1995 is broken today (MD5, SHA-1, RSA-1024). Same code, new threat. The `rsa` Marvin Attack is exactly this — theoretical since the 90s, practical in 2023.
4. **Your dependencies have dependencies.** A library you trust can pull in a vulnerable sub-library next month, without changing its own API at all.
5. **Edge cases nobody thought of.** `qs` had been parsing query strings fine for years until someone tried `?a[]=,&a[]=` and crashed it. The bug had been there since the comma-format feature was added in 2018.

### What this means in practice

**Vulnerability management is gardening, not construction.** You don't "finish." You run scanners continuously, patch within SLA, document the unfixable, repeat forever.

| Severity | Typical patch SLA |
|---|---|
| Critical | Hours to a day |
| High | This week |
| Medium | This sprint / month |
| Low | When you next touch that area |

The goal is **not** "zero vulnerabilities." It's:
- **Detect** new ones fast (daily scans)
- **Triage** honestly (is this reachable in our app?)
- **Fix** the real ones in time
- **Document** the rest (so auditors and your future self know why you didn't panic)

---

## TL;DR of the TL;DR

- Hackers don't just crash servers — they steal data, hijack accounts, take over machines, and **convert all of it to money** (dark-web sales, ransomware, fraud).
- New vulns keep appearing forever because they're *discovered*, not *created* — and attacker techniques keep improving too.
- Security work is continuous gardening. The dashboard is never empty for long; the goal is to keep it triaged and current.
