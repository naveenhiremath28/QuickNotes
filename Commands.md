


docker run -d \
  --name keycloak \
  -p 8080:8080 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  quay.io/keycloak/keycloak:26.4.1 \
  start-dev



docker run --name vault-dev \
  -d \
  -p 8201:8200 \
  -e VAULT_DEV_ROOT_TOKEN_ID=dev-token \
  -e VAULT_DEV_LISTEN_ADDRESS=0.0.0.0:8200 \
  hashicorp/vault:latest



docker run -d \
  --name clickstack \
  -p 8080:8080 \
  -p 4317:4317 \
  -p 4318:4318 \
  docker.hyperdx.io/hyperdx/hyperdx-all-in-one:latest



> openssl genrsa -out jwt_private.pem 2048
> awk '{printf "%s\\n", $0}' jwt_private.pem


