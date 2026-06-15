
```bash
cd charts
helm template proof-service proof-service -n proof-service
```

```bash
export TERRAHELP_ENCRYPTION_KEY="<TERRAHELP_ENCRYPTION_KEY>"
```

```bash
gcloud auth login
gcloud auth application-default login --account=naveen.hiremath@finternetlab.io
```

```bash
cd /scripts
./provision.sh install --provider gcp --env dev --resource gke --clean --dry-run
./provision.sh install --provider gcp --env dev --resource gke --clean
```

```bash
cd /scripts
./install-services.sh -e dev -c gcp --dry-run
./install-services.sh -e dev -c gcp
```



```bash
gcloud compute scp --zone "asia-southeast1-c" --project "finternet-sandbox" \
  fed-vm-all.tar.gz "federation-dev":~/federation/
  
tar -xzf ~/fed-vm-all.tar.gz
```


```
set -a; . ./.env; set +a

./down.sh -v
```


```bash
gcloud compute ssh --zone "asia-southeast1-c" "federation-dev" --project "finternet-sandbox" -- -N -L 3000:localhost:3000 -L 3001:localhost:3001 -L 3002:localhost:3002 -L 3100:localhost:3100 -L 3101:localhost:3101 -L 4000:localhost:4000 -L 5432:localhost:5432 -L 5433:localhost:5433 -L 5434:localhost:5434 -L 6000:localhost:6000 -L 6090:localhost:6090 -L 8060:localhost:8060 -L 8061:localhost:8061 -L 8082:localhost:8082 -L 8083:localhost:8083 -L 8090:localhost:8090 -L 8091:localhost:8091 -L 8200:localhost:8200 -L 8201:localhost:8201 -L 9000:localhost:9000 -L 9001:localhost:9001 -L 9010:localhost:9010 -L 9011:localhost:9011 -L 9070:localhost:9070 -L 9071:localhost:9071 -L 9080:localhost:9080 -L 9081:localhost:9081 -L 9092:localhost:9092 -L 9093:localhost:9093


gcloud compute ssh --zone "asia-southeast1-c" "federation-dev" --project "finternet-sandbox" -- \
  -L 3000:localhost:3000 -L 3001:localhost:3001 \
  -L 3100:localhost:3100 -L 3101:localhost:3101 \
  -L 5432:localhost:5432  -L 5433:localhost:5433  -L 5434:localhost:5434 \
  -L 4000:localhost:4000 \
  -L 8200:localhost:8200  -L 8201:localhost:8201 \
  -L 8082:localhost:8082  -L 8083:localhost:8083 \
  -L 8090:localhost:8090  -L 8091:localhost:8091 \
  -L 9092:localhost:9092  -L 9093:localhost:9093 \
  -L 9000:localhost:9000  -L 9010:localhost:9010 \
  -L 9001:localhost:9001  -L 9011:localhost:9011
```

without units
```bash
gcloud compute ssh --zone "asia-southeast1-c" "federation-dev" --project "finternet-sandbox" -- -N -f \
  -L 3100:localhost:3100 -L 3101:localhost:3101 \
  -L 5432:localhost:5432  -L 5433:localhost:5433  -L 5434:localhost:5434 \
  -L 4000:localhost:4000 \
  -L 8200:localhost:8200  -L 8201:localhost:8201 \
  -L 8082:localhost:8082  -L 8083:localhost:8083 \
  -L 8090:localhost:8090  -L 8091:localhost:8091 \
  -L 9092:localhost:9092  -L 9093:localhost:9093 \
  -L 9000:localhost:9000  -L 9010:localhost:9010 \
  -L 9001:localhost:9001  -L 9011:localhost:9011
```




```
 docker volume prune -af
```




```bash
cd ~/federation/federation-vm/compose
cat > /tmp/imgs.env <<'EOF'
UNITS_API_IMAGE=ghcr.io/finternet-io/units-api/api:2026.06.10_fed
WORKFLOWS_IMAGE=ghcr.io/finternet-io/units-workflows/workflow-orchestrator:2026.06.10_fed
REGISTRY_IMAGE=ghcr.io/finternet-io/units-services/units-registry:2026.06.11_fed
TOKEN_ENGINE_IMAGE=ghcr.io/finternet-io/units-token-runtime/token-engine:2026.06.10_fed
EOF
```

to check images
```bash
for c in units-api-1 workflows-1 token-engine-1 registry; do
  echo -n "$c: "; img=$(docker inspect "$c" --format '{{.Config.Image}}')
  docker image inspect "$img" --format "{{.RepoTags}} built={{.Created}}"
done
```



```bash
for p in 3000 3001 3100 3101 5432 5433 5434 4000 4001 8200 8201 8082 8083 8090 8091 9092 9093 9000 9010 9001 9011; do
  pids=$(lsof -ti :"$p")
  if [ -n "$pids" ]; then
    echo "killing port $p -> $pids"
    kill -9 $pids
  else
    echo "port $p free"
  fi
done
```

```bash
# fixed in 
# Resume this session with:
claude --resume 940d01fa-6e24-4dee-a4ec-67aca5b91eec
```


```bash
Resume this session with:
claude --resume c9130fca-61d5-47c4-bba4-191243e71825
```