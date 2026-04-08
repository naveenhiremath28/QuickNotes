
## 1. Repository

```bash
# Add a repository
helm repo add <name> <url>
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add stable https://charts.helm.sh/stable

# List repositories
helm repo list

# Update all repositories
helm repo update

# Remove a repository
helm repo remove <name>

# Search charts in repos
helm search repo <keyword>
helm search repo nginx
helm search repo nginx --versions       # show all versions
```

---

## 2. Install

```bash
# Basic install
helm install <release-name> <chart>
helm install myapp bitnami/nginx

# Install from local chart directory
helm install myapp ./mychart

# Install with inline value override
helm install myapp ./mychart --set image.tag=v2.0.0

# Install with multiple --set flags
helm install myapp ./mychart --set image.tag=v2 --set replicas=3

# Install with a values file
helm install myapp ./mychart -f prod-values.yaml

# Install combining file + inline override
helm install myapp ./mychart -f prod.yaml --set image.tag=v2

# Install into a specific namespace
helm install myapp ./mychart -n production

# Install and create namespace if missing
helm install myapp ./mychart -n production --create-namespace

# Install a specific chart version
helm install myapp bitnami/nginx --version 13.2.0

# Dry run (simulate, don't apply)
helm install myapp ./mychart --dry-run

# Generate a release name automatically
helm install ./mychart --generate-name
```

---

## 3. Upgrade

```bash
# Basic upgrade
helm upgrade <release-name> <chart>
helm upgrade myapp bitnami/nginx

# Upgrade with value override
helm upgrade myapp ./mychart --set image.tag=v3.0.0

# Upgrade with a values file
helm upgrade myapp ./mychart -f prod-values.yaml

# Install if not exists, upgrade if exists (upsert)
helm upgrade --install myapp ./mychart

# Auto rollback on failure
helm upgrade myapp ./mychart --atomic

# Upgrade with timeout
helm upgrade myapp ./mychart --atomic --timeout 5m

# Cleanup new resources if upgrade fails
helm upgrade myapp ./mychart --cleanup-on-fail

# Dry run upgrade
helm upgrade myapp ./mychart --dry-run

# Reuse previous values + apply new overrides
helm upgrade myapp ./mychart --reuse-values --set image.tag=v3

# Reset all values to chart defaults before upgrading
helm upgrade myapp ./mychart --reset-values -f new.yaml

# Recommended CI/CD upgrade command
helm upgrade --install myapp ./mychart \
  --set image.tag=$IMAGE_TAG \
  -f prod-values.yaml \
  --atomic \
  --timeout 3m \
  --cleanup-on-fail \
  -n production
```

---

## 4. Rollback

```bash
# View release revision history
helm history <release-name>
helm history myapp

# Roll back to a specific revision
helm rollback <release-name> <revision>
helm rollback myapp 2

# Roll back to the previous revision
helm rollback myapp

# Roll back in a specific namespace
helm rollback myapp 2 -n production

# Dry run rollback
helm rollback myapp 2 --dry-run
```

---

## 5. List & Status

```bash
# List all releases in current namespace
helm list

# List releases in a specific namespace
helm list -n production

# List releases in all namespaces
helm list -A

# List only failed releases
helm list --failed

# List only deployed releases
helm list --deployed

# Show status of a release
helm status myapp
helm status myapp -n production
```

---

## 6. Get / Inspect

```bash
# Get values used in the current release
helm get values myapp

# Get values at a specific revision
helm get values myapp --revision 2

# Get rendered K8s manifests currently deployed
helm get manifest myapp

# Get hooks of a release
helm get hooks myapp

# Get everything (values + manifest + hooks + notes)
helm get all myapp

# Show a chart's default values (before installing)
helm show values bitnami/nginx

# Show chart metadata
helm show chart bitnami/nginx

# Show chart README
helm show readme bitnami/nginx
```

---

## 7. Uninstall

```bash
# Uninstall a release
helm uninstall <release-name>
helm uninstall myapp

# Uninstall from a specific namespace
helm uninstall myapp -n production

# Uninstall but keep release history
helm uninstall myapp --keep-history

# Dry run uninstall
helm uninstall myapp --dry-run
```

---

## 8. Chart Development

```bash
# Create a new chart scaffold
helm create <chart-name>
helm create mychart

# Lint a chart (validate syntax and structure)
helm lint ./mychart

# Render templates locally without installing
helm template <release-name> <chart>
helm template myapp ./mychart

# Render with a values file
helm template myapp ./mychart -f prod-values.yaml

# Render and show computed values
helm template myapp ./mychart --debug

# Package chart into a .tgz archive
helm package ./mychart

# Package with a specific version
helm package ./mychart --version 2.0.0

# Download chart dependencies
helm dependency update ./mychart

# List chart dependencies
helm dependency list ./mychart

# Pull a chart to local disk
helm pull bitnami/nginx

# Pull and extract
helm pull bitnami/nginx --untar

# Pull a specific version
helm pull bitnami/nginx --version 13.2.0
```

---

## 9. Testing & Plugins

```bash
# Run helm tests for a release
helm test myapp

# List installed plugins
helm plugin list

# Install a plugin
helm plugin install <url>

# Update a plugin
helm plugin update <plugin-name>

# Remove a plugin
helm plugin uninstall <plugin-name>
```

---

## 10. Misc

```bash
# Show Helm version
helm version

# Show Helm environment info
helm env
```

---

---

# COMBINED CHEAT SHEET

|Action|kubectl|helm|
|---|---|---|
|List running apps|`kubectl get deployments -A`|`helm list -A`|
|See app details|`kubectl describe deploy <n>`|`helm status <n>`|
|See logs|`kubectl logs <pod>`|—|
|Apply config|`kubectl apply -f file.yaml`|`helm upgrade --install`|
|Delete app|`kubectl delete -f file.yaml`|`helm uninstall <n>`|
|Scale app|`kubectl scale deploy <n> --replicas=3`|`helm upgrade <n> --set replicas=3`|
|Roll back|`kubectl rollout undo deploy/<n>`|`helm rollback <n> <rev>`|
|See history|`kubectl rollout history deploy/<n>`|`helm history <n>`|
|Debug pod|`kubectl exec -it <pod> -- bash`|—|
|Check events|`kubectl get events -n <ns>`|—|
|See raw YAML|`kubectl get deploy <n> -o yaml`|`helm get manifest <n>`|
|See values|—|`helm get values <n>`|