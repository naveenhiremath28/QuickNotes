

```
 docker system prune -a --volumes -f
```

```
docker run -d \
  --name keycloak \
  -p 8080:8080 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=admin \
  quay.io/keycloak/keycloak:26.4.1 \
  start-dev
```



```
docker run --name vault-dev \
  -d \
  -p 8201:8200 \
  -e VAULT_DEV_ROOT_TOKEN_ID=dev-token \
  -e VAULT_DEV_LISTEN_ADDRESS=0.0.0.0:8200 \
  hashicorp/vault:latest
```


```
docker run -d \
  --name clickstack \
  -p 8080:8080 \
  -p 4317:4317 \
  -p 4318:4318 \
  docker.hyperdx.io/hyperdx/hyperdx-all-in-one:latest
```



> openssl genrsa -out jwt_private.pem 2048
> awk '{printf "%s\\n", $0}' jwt_private.pem



```
docker exec -it units-kafka sh
```


```
cd /opt/kafka
```

```
./bin/kafka-topics.sh --bootstrap-server localhost:9092 --list

./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic units.token.operations --from-beginning




./bin/kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic units.token.operations


./bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic units.token.operations --partitions 4 --replication-factor 1 --config min.insync.replicas=1




./bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic units.token.operations


./bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic units.token.operations


./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list

./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --group token-engine
```

