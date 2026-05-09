
```
================================================================
   DEVOPS NOTES: HELM + KUBERNETES (Beginner Session)
================================================================

----------------------------------------------------------------
1. HELM RELEASE NAME vs NAMESPACE
----------------------------------------------------------------
These are TWO different things that beginners often confuse.

  Release Name → A label Helm puts on every resource it creates.
                 Set with: helm install <RELEASE_NAME> ./chart
                 Becomes a prefix: test-postgresql-0

  Namespace    → A "folder" in the cluster where resources live.
                 Resources in different namespaces are isolated.
                 Defaults to whatever your kubectl context uses.

EXAMPLE:
  helm install test ./manifests/crud-helm
       │
       └── release name = "test"
       └── namespace    = "default" (unless overridden)

USEFUL COMMANDS:
  helm list --all-namespaces          (see all releases everywhere)
  kubectl get pods -A                 (see all pods, all namespaces)
  kubectl get pods -n <namespace>     (pods in one namespace)

----------------------------------------------------------------
2. SUB-CHART DEPENDENCIES HAVE HIDDEN DEFAULTS
----------------------------------------------------------------
When a chart depends on another chart (like Keycloak depending
on PostgreSQL), the sub-chart deploys with DEFAULT VALUES unless
you explicitly configure it.

PROBLEM WE HIT:
  - Top-level postgresql:  → went to crud-golang namespace ✅
  - Top-level keycloak:    → went to crud-golang namespace ✅
  - Keycloak's BUNDLED postgres → went to default namespace ❌
                                  (we never configured it)

WHY: Bitnami Keycloak chart bundles its own postgres sub-chart.
     If you don't override keycloak.postgresql.*, defaults win.

FIX OPTIONS:
  Option A — Disable bundled postgres, use shared one:
    keycloak:
      postgresql:
        enabled: false
      externalDatabase:
        host: "postgres"
        port: 5432
        user: "bn_keycloak"
        password: "keycloak"
        database: "bitnami_keycloak"

  Option B — Let it have its own, but in the right place:
    keycloak:
      postgresql:
        fullnameOverride: "keycloak-postgres"
        namespaceOverride: "crud-golang"

PREVIEW BEFORE INSTALL:
  helm dependency list ./chart
  helm template ./chart | grep "kind:" | sort | uniq -c

----------------------------------------------------------------
3. CHART VERSION vs IMAGE TAG (Two Different Versions!)
----------------------------------------------------------------
Beginners constantly mix these up.

  CHART VERSION       IMAGE TAG
  ─────────────       ──────────
  Version of the      Version of the actual app inside
  Helm packaging      the Docker container
  (templates)
  
  Found in:           Found in:
  Chart.yaml          values.yaml (image.tag)
  Chart.lock          
  
  Example:            Example:
  postgresql:18.5.16  bitnami/postgresql:17.6.0-debian-12-r0

LESSON: Updating Chart.lock won't fix image errors.
        Updating image.tag won't change chart behavior.

----------------------------------------------------------------
4. CHART.LOCK — AUTO-GENERATED, DON'T EDIT
----------------------------------------------------------------
Chart.lock is like package-lock.json (Node) or go.sum (Go).
It records EXACT versions of dependencies for reproducibility.

NEVER edit it by hand. Only update it via:
  helm dependency update ./chart

When does it need updating?
  - You added/removed a dependency in Chart.yaml
  - You bumped a dependency version in Chart.yaml
  - You cloned the repo fresh and charts/ folder is missing

----------------------------------------------------------------
5. BITNAMI IMAGE MIGRATION (August 2025)
----------------------------------------------------------------
Broadcom restructured Bitnami's free Docker Hub catalog.

WHAT CHANGED:
  - Many bitnami/* images were moved to bitnamilegacy/*
  - The "latest" tag was removed from many images
  - Some images now require a paid subscription

ERROR YOU'LL SEE:
  Failed to pull image "docker.io/bitnami/keycloak:latest":
  ... not found

FIX:
  Change repository from "bitnami/X" → "bitnamilegacy/X"
  AND pin a specific version (never use "latest")

  postgresql:
    image:
      repository: bitnamilegacy/postgresql
      tag: "17.6.0-debian-12-r0"
  
  keycloak:
    image:
      repository: bitnamilegacy/keycloak
      tag: "26.0.7-debian-12-r0"

BROWSE TAGS:
  https://hub.docker.com/r/bitnamilegacy/keycloak/tags
  https://hub.docker.com/r/bitnamilegacy/postgresql/tags

----------------------------------------------------------------
6. NEVER USE "latest" — ALWAYS PIN VERSIONS
----------------------------------------------------------------
"latest" is a moving target. Today it's v26, tomorrow v27 with
breaking changes.

GOLDEN RULE:
  Same chart + same values = same result, every time.
  Treat "latest" as a code smell.

----------------------------------------------------------------
7. JOB IMMUTABILITY ("field is immutable" error)
----------------------------------------------------------------
Kubernetes Jobs are SEALED ENVELOPES.
Once created, you cannot edit spec.template.

Why? A Job runs a one-time task (seed DB, backup, etc.).
     If you could change its image mid-run, what should
     Kubernetes do? Stop? Restart? Re-run? It's ambiguous.
     So Kubernetes refuses edits.

ANALOGY:
  Deployment / StatefulSet = restaurant (open daily, menu changes)
  Job                      = catering order (one event, can't edit)

WHEN IT BITES YOU:
  You change values.yaml (e.g. image tag).
  Helm tries to patch the existing Job. Kubernetes says NO.

----------------------------------------------------------------
8. ONE VALUE, MANY PLACES (Helm Trap)
----------------------------------------------------------------
A single value in values.yaml can flow into multiple resources.

EXAMPLE:
  postgresql.image.tag is used in:
    1. The Postgres pod itself (StatefulSet) — accepts changes
    2. The seed Job's wait-for-postgres init container — REJECTS
    3. The seed Job's seed container — REJECTS

ONE EDIT → THREE PLACES UPDATED → ONE FAILS (the Job)

LESSON: When changing a value, think about every template that
        consumes it.

----------------------------------------------------------------
9. SOLUTIONS FOR JOB IMMUTABILITY
----------------------------------------------------------------

  Option 1 — Manual delete (quick fix):
  ─────────────────────────────────────
    kubectl delete job <job-name> -n <namespace>
    helm upgrade <release> ./chart
    
    + Works immediately
    - Have to remember every time

  Option 2 — Helm hook (PROPER FIX, recommended):
  ───────────────────────────────────────────────
    Add to Job's metadata:
    
      annotations:
        "helm.sh/hook": post-install,post-upgrade
        "helm.sh/hook-delete-policy": before-hook-creation
    
    + Set once, forget forever
    + Industry standard
    - Job runs on every upgrade (usually fine)

  Option 3 — Random suffix on Job name:
  ─────────────────────────────────────
    Make Job name change every install (e.g. add random hash).
    + Kept history of every run
    - Old Jobs pile up unless cleaned

  Option 4 — Use init container in app pod instead:
  ─────────────────────────────────────────────────
    Skip the Job entirely. App pod runs seed before starting.
    + No immutability issue
    - Runs on every pod restart (must be idempotent)

----------------------------------------------------------------
10. COMMON IMMUTABLE FIELDS IN KUBERNETES
----------------------------------------------------------------
When you see "field is immutable", these are usual suspects:

  Resource          Immutable Field(s)
  ──────────        ───────────────────────────────────
  Job               spec.template (entire pod template)
  Deployment        spec.selector
  StatefulSet       spec.selector, spec.serviceName
  Service           spec.clusterIP
  PersistentVolume  spec.resources.requests.storage
  Claim             (can only GROW, never shrink)

FIX: Almost always — kubectl delete the resource, then re-apply.

----------------------------------------------------------------
11. KEYCLOAK NEEDS A DATABASE (Always)
----------------------------------------------------------------
Keycloak is STATELESS. All realms, users, roles, clients,
sessions, tokens live in a Postgres database. Not in Keycloak.

YOUR TWO OPTIONS:
  - Bundled DB (chart spins one up, isolated, simple)
  - External DB (point to existing one, shared, less overhead)

EXTERNAL DB SETUP REQUIRES:
  Before Keycloak starts, the database and user must exist:
  
    CREATE USER bn_keycloak WITH PASSWORD 'keycloak';
    CREATE DATABASE bitnami_keycloak OWNER bn_keycloak;
    GRANT ALL PRIVILEGES ON DATABASE bitnami_keycloak
      TO bn_keycloak;
  
  Run this from your seed Job (so it executes before Keycloak).

----------------------------------------------------------------
QUICK COMMAND CHEAT SHEET
----------------------------------------------------------------
  helm install <release> ./chart        Install fresh
  helm upgrade <release> ./chart        Update existing
  helm uninstall <release>              Remove release
  helm list --all-namespaces            See all releases
  helm dependency list ./chart          See chart deps
  helm dependency update ./chart        Refresh deps + lock
  helm template ./chart                 Render YAML locally
                                        (no install — preview)

  kubectl get pods -A                   All pods everywhere
  kubectl get pods -n <ns> -w           Watch pods live
  kubectl describe pod <name> -n <ns>   Detailed pod info
  kubectl logs <pod> -n <ns>            Container logs
  kubectl delete job <name> -n <ns>     Delete a Job
  kubectl get pvc -A                    Check leftover storage

----------------------------------------------------------------
DEBUGGING MENTALITY (THE DEVOPS WAY)
----------------------------------------------------------------
  - Fix ONE error at a time. Don't chase multiple in parallel.
  - Read the LAST line of long error walls — that's the real msg.
  - "Events:" section in kubectl describe is your best friend.
  - When in doubt: helm uninstall, clean PVCs, start fresh.
  - Pin everything. Avoid "latest". Reproducibility > convenience.

================================================================

```


## KUBERNETES SERVICE FQDNs

```
================================================================
   DEVOPS NOTES: KUBERNETES SERVICE FQDNs
================================================================

----------------------------------------------------------------
1. WHAT IS AN FQDN?
----------------------------------------------------------------
FQDN = Fully Qualified Domain Name.
The COMPLETE DNS address of a Service inside the cluster.

FORMAT:
  <service-name>.<namespace>.svc.<cluster-domain>

EXAMPLE:
  postgres.crud-data.svc.cluster.local
     │         │      │        │
     │         │      │        └── cluster domain (default)
     │         │      └── tells DNS "this is a Service"
     │         └── namespace where the Service lives
     └── Service name

----------------------------------------------------------------
2. FOUR WAYS TO ADDRESS A SERVICE
----------------------------------------------------------------

  FORM                          EXAMPLE                        WORKS FROM
  ──────────────────────────    ──────────────────────────     ─────────────
  Short name                    postgres                       Same ns only
  With namespace                postgres.crud-data             Any namespace
  With .svc                     postgres.crud-data.svc         Any namespace
  Full FQDN                     postgres.crud-data.svc.        Any namespace
                                cluster.local                  (safest)

----------------------------------------------------------------
3. WHY THE SHORT NAME "JUST WORKS" (Same Namespace)
----------------------------------------------------------------
Every pod has /etc/resolv.conf with a DNS "search list".

TYPICAL CONTENTS (pod in crud-app namespace):
  search crud-app.svc.cluster.local svc.cluster.local cluster.local
  nameserver 10.96.0.10

When you type "postgres", the resolver tries EACH suffix:
  1. postgres.crud-app.svc.cluster.local   ← tries own ns first
  2. postgres.svc.cluster.local
  3. postgres.cluster.local

That's why shortnames work — BUT ONLY for Services in the
pod's OWN namespace. Cross-namespace = the search list won't
find it without the namespace part.

----------------------------------------------------------------
4. WHEN TO USE FQDNs
----------------------------------------------------------------
  - Cross-namespace calls  → shortname fails, FQDN required
  - Production configs     → explicit > implicit; no surprises
  - Sidecars / meshes      → some bypass the search list
  - init containers        → DNS may be limited at startup

GOLDEN RULE:
  Same namespace      → shortname is fine
  Different namespace → use at LEAST <svc>.<ns>, prefer full FQDN

----------------------------------------------------------------
5. CLUSTER DOMAIN GOTCHA
----------------------------------------------------------------
"cluster.local" is the DEFAULT cluster domain — but not always.
Some clusters use custom domains (e.g. "cluster.prod").

IF YOU MIGRATE CLUSTERS:
  - Hardcoded "postgres.crud-data.svc.cluster.local" → breaks
  - Safer middle ground: "postgres.crud-data.svc"
  - Even safer: read from env / config, not hardcoded

----------------------------------------------------------------
6. REAL EXAMPLE FROM THIS PROJECT (3-Namespace Split)
----------------------------------------------------------------

  COMPONENT        LIVES IN         OTHERS REACH IT VIA
  ──────────       ─────────────    ─────────────────────────
  Go app           crud-app         app.crud-app.svc.cluster.local
  PostgreSQL       crud-data        postgres.crud-data.svc.cluster.local
  Keycloak         crud-auth        keycloak.crud-auth.svc.cluster.local

CONFIG CHANGES:
  # App ConfigMap
  DB_HOST: "postgres.crud-data.svc.cluster.local"
  KC_HOST: "keycloak.crud-auth.svc.cluster.local"

  # Keycloak values (its DB is in another namespace now!)
  externalDatabase:
    host: "postgres.crud-data.svc.cluster.local"

----------------------------------------------------------------
7. DEBUGGING DNS INSIDE A POD
----------------------------------------------------------------
When "service not found" errors hit, get inside a pod and test:

  kubectl exec -it <pod> -n <ns> -- sh

  # Check what search domains this pod has
  cat /etc/resolv.conf

  # Test resolution at each level
  nslookup postgres
  nslookup postgres.crud-data
  nslookup postgres.crud-data.svc.cluster.local

  # If nslookup isn't installed, use getent:
  getent hosts postgres.crud-data.svc.cluster.local

WHAT GOOD OUTPUT LOOKS LIKE:
  Name:   postgres.crud-data.svc.cluster.local
  Address: 10.96.45.123     ← ClusterIP of the Service

IF YOU GET "NXDOMAIN":
  - Service doesn't exist in that namespace, OR
  - Typo in the name, OR
  - CoreDNS is down (check: kubectl get pods -n kube-system)

================================================================

```
