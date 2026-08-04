
```bash
export TERRAHELP_ENCRYPTION_KEY="<TERRAHELP_ENCRYPTION_KEY>"
```

```bash
gcloud auth login
gcloud auth application-default login --account=naveen.hiremath@nfh.globals
```

```bash
cd /scripts
./provision.sh install --provider gcp --env staging --instance units-in --resource gke --clean --dry-run


./provision.sh install --provider gcp --env staging --instance units-sg --resource gke --clean --dry-run

./provision.sh install --provider gcp --env staging --instance registry --resource gke --clean --dry-run



./provision.sh install --provider gcp --env staging --instance units-in --resource gke --clean


./provision.sh install --provider gcp --env staging --instance units-sg --resource gke --clean

./provision.sh install --provider gcp --env staging --instance registry --resource gke --clean
```


```bash
kubie ctx
```


```bash
cd /scripts
./install-services.sh -e dev -c gcp --dry-run
./install-services.sh -e dev -c gcp

# --dry-run

./install-services.sh -e staging -c gcp -i registry

./install-services.sh -e staging -c gcp -i units-in

./install-services.sh -e staging -c gcp -i units-sg
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
  -L 9001:localhost:9001  -L 9011:localhost:9011 \
  -L 3500:localhost:3500
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
  -L 9001:localhost:9001  -L 9011:localhost:9011 \
  -L 3500:localhost:3500
```

only units and registry

```bash
gcloud compute ssh --zone "asia-southeast1-c" "federation-dev" --project "finternet-sandbox" -- \
  -L 3000:localhost:3000 -L 3001:localhost:3001 \
  -L 3100:localhost:3100 -L 3101:localhost:3101 \
  -L 5432:localhost:5432  -L 5433:localhost:5433  -L 5434:localhost:5434
```


```
 docker volume prune -af
```

gcloud compute ssh --zone "asia-southeast1-c" "federation-dev" --project "finternet-sandbox" -- \
  -L 8091:localhost:8090

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
echo "127.0.0.1 units-api-1" | sudo tee -a /etc/hosts
```


```bash
git tag -a 2026.06.25_RC token-program-refactor -m "2026.06.25_RC"
git push origin 2026.06.25_RC
```





```bash
registry:  gcloud container fleet memberships get-credentials registry-sanctum --project=finternet-sandbox

units-sg:  gcloud container fleet memberships get-credentials units-sg-sanctum --project=finternet-sandbox 

units-in:  gcloud container fleet memberships get-credentials units-in-sanctum --project=finternet-sandbox



registry: sanctum.finternetlab.io (app -> / , registry -> /ulip, /v1) 
units-sg: sanctum-sg.finternetlab.io, units.sanctum-sg.finternetlab.io 
units-in: sanctum-in.finternetlab.io, units.sanctum-in.finternetlab.io
```





```bash
FED_STACK_MODE=images npm start
```


```json
{
  "context": {
    "id": "api.account.keys.register",
    "version": "v1",
    "ts": "{{$isoTimestamp}}",
    "msgId": "{{$guid}}",
    "developerToken": "{{developer_token}}",
    "authorization": "{{auth_token}}"
  },
  "payload": {
    "publicKey": "0x04f364799593ca3f87941ba35cf265e781fbae9af8a7fe0bfdf90b276b16cde0f23b0d0cad71776c4a6752dc495a80a4db2b7ef081ff2be7651ee97cb3a014d771",
    "type": "secp256k1",
    "name": "proxy-test-pubkey-0x-coinbase"
  }
}
```

```json
"payload": {
    "publicKey": "040e9d94b5a1c3941241825ec45b1c16de5906f697e20498bdccdaaa42bc6c8ddcae3afa8c8f7ddd24936d4d34452af38dd01783fe975b68e3a4aa3ddf649be1f4",
    "type": "secp256k1",
    "name": "proxy-test-pubkey-hex-robinhood"
  }
```

```json

Proxy scenario A — Binance hot wallet (top holder of USDT/USDC and dozens more):
{
  "context": {
    "id": "api.account.keys.register",
    "version": "v1",
    "ts": "{{$isoTimestamp}}",
    "msgId": "{{$guid}}",
    "developerToken": "{{developer_token}}",
    "authorization": "{{auth_token}}"
  },
  "payload": {
    "address": "0xF977814e90dA44bFA03b6295A0616a897441aceC",
    "type": "secp256k1",
    "name": "proxy-test-binance-8"
  }
}
Expected: 200/201 with "address": "0xf977814e90da44bfa03b6295a0616a897441acec" (lowercased), "publicKeyHex": null, "isDefault": false — and holdings discovery fans out to find its enormous ERC-20 balances for proxy import.

Proxy scenario B — vitalik.eth (diverse long-tail token holdings, good for testing many small/odd tokens):
  "payload": {
    "address": "0xd8dA6BF26964aF9D7eEd9e03E93601FD31E883f7",
    "type": "secp256k1",
    "name": "proxy-test-vitalik"
  }

Proxy scenario C — Kraken hot wallet (another distinct high-volume holder):
  "payload": {
    "address": "0x2910543Af39abA0Cd09dBb2D50200b3E800A63D2",
    "type": "secp256k1",
    "name": "proxy-test-kraken"
  }

```


public key for 
```
/Users/naveenvhiremath/Downloads/test-key.pem
```

```json
The key at test-key.pem is an Ed25519 private key. Its 32-byte public key, in the two encodings /keys/register auto-detects (RegisterKeyRequest.PublicKey accepts hex or base58):

- Hex: 7dc05939d3670209aa3b5e32297c650318d22f0637b224a918691c78ea51ec29
- Base58: 9Tt5RUt9fKhnXxhhztDjQQWRH7Jbo1dRVrVQfFXZymzG

Example request payload:

{
  "payload": {
    "publicKey": "7dc05939d3670209aa3b5e32297c650318d22f0637b224a918691c78ea51ec29",
    "type": "ed25519",
    "name": "test-key",
    "isPrimary": false
  }
}

```



```
 /Users/naveenvhiremath/Downloads/test-key-2.pem
```

```json

Public key:
- Hex: 0c965d8b132ce6a73c9f5521f87f9b0f143e59e2ca8031e33b9422b1366ebaf7
- Base58: r8spsHHQsiNA7p3Hh6mMPzaueuGBrtr4ew74dyGAcUN

```
