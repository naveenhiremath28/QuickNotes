

Doubt: why jwt validation cant be done int api gateway
```
ZP@b7axAgCAlbX
```

```bash
export OTEL_SERVICE_NAME=finternet-units-workflow
export SERVICE_VERSION=1.0.0
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
export OTEL_EXPORTER_OTLP_HEADERS=authorization=246da7cc-626d-48ba-b7a4-eed53c9ee0ed
```

```
claude --dangerously-skip-permissions
```

```
naveen.hiremath@finternetlab.io
```

```
lakshmi@example.co.in
```


```
0xa923B13270F8622B5d5960634200Dc4302b7611E
```


```
0x95ff257ea9842077C23CFeec9B40C55631F4300d
```


```
0x2CfF890f0378a11913B6129B2E97417a2c302680
```


```
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
```



```
kubectl exec -n units-api units-api-7c78c5d64c-487jl -- env | grep -i "CRYPTO\|VAULT\|ROLE\|SECRET"
```

```
curl -s -X POST http://localhost:8200/v1/auth/approle/login \ -d '{"role_id":"<actual-role-id>","secret_id":"<actual-secret-id>"}'
```

```
 curl -s -X POST http://localhost:8200/v1/auth/approle/login \
  -d '{"role_id":"finternet-user","secret_id":"paS8fpZHvIuHbCsRjeRiwrS4Vz99cXfj"}'
```






```
CRYPTO_CONFIG={"address":"http://vault.vault.svc.cluster.local:8200","authMethod":"kubernetes","kubernetesRole":"units-api","kubernetesAuthPath":"kubernetes","transitMount":"transit","encryptionKeyName":"units-encryption-master","jwtAuthPath":"jwt","jwtRole":"finternet-user"}
```


### Get the service account token (it's also a JWT)
```bash
kubectl exec -n units-api units-api-7c78c5d64c-487jl -- \
  cat /var/run/secrets/kubernetes.io/serviceaccount/token
```




```
curl -s -X POST http://localhost:8200/v1/auth/jwt/login \
  -d '{
    "role": "finternet-user",
    "jwt": "eyJhbGciOiJSUzI1NiIsImtpZCI6IklIejhheFlEX2JpRXQ4WXpleUk5Q3MwdHRNV1BRaDdsMWltb1dKWlpMTE0ifQ.eyJhdWQiOlsiaHR0cHM6Ly9jb250YWluZXIuZ29vZ2xlYXBpcy5jb20vdjEvcHJvamVjdHMvZmludGVybmV0LXNhbmRib3gvbG9jYXRpb25zL2FzaWEtc291dGhlYXN0MS1hL2NsdXN0ZXJzL2ZpbnRlcm5ldC1kZXYtY2x1c3RlciJdLCJleHAiOjE4MDY3MzA2NjgsImlhdCI6MTc3NTE5NDY2OCwiaXNzIjoiaHR0cHM6Ly9jb250YWluZXIuZ29vZ2xlYXBpcy5jb20vdjEvcHJvamVjdHMvZmludGVybmV0LXNhbmRib3gvbG9jYXRpb25zL2FzaWEtc291dGhlYXN0MS1hL2NsdXN0ZXJzL2ZpbnRlcm5ldC1kZXYtY2x1c3RlciIsImp0aSI6IjcyZWQ2ZmYwLTc0MjktNDkxNS1iNGE3LTMxNTdhNjZmZDAyMiIsImt1YmVybmV0ZXMuaW8iOnsibmFtZXNwYWNlIjoidW5pdHMtYXBpIiwibm9kZSI6eyJuYW1lIjoiZ2tlLWZpbnRlcm5ldC1kZXYtY2wtZmludGVybmV0LWRldi1wby05ZWFiYzc0Yy04ZWo2IiwidWlkIjoiNmViNzhiODktYTdlMy00MzM1LWI2YTUtNWJjMTE5MTk0NmU4In0sInBvZCI6eyJuYW1lIjoidW5pdHMtYXBpLTdjNzhjNWQ2NGMtNDg3amwiLCJ1aWQiOiI0NjE3OWMwZS1lNDYzLTRiY2UtYjRiNC0yYTZkMDI0NGUzYTYifSwic2VydmljZWFjY291bnQiOnsibmFtZSI6InVuaXRzLWFwaS1zYSIsInVpZCI6ImQxMzZiMGI3LTRhZjktNDcxYi05M2MyLTdiMDUzMTRjN2MyMyJ9LCJ3YXJuYWZ0ZXIiOjE3NzUxOTgyNzV9LCJuYmYiOjE3NzUxOTQ2NjgsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDp1bml0cy1hcGk6dW5pdHMtYXBpLXNhIn0.uMhxi9pAquD4lOd8HhrC7DOfH3QI1wr1zp7dKkFbqa351ykPKoVgBfJjVI2JnfooUXJHPgLzcXSeZ_R_vEO9f9Xuz6PPD93liS3tMi84G470CnxmB0-CZn77sWNcFpjAMCAWE8T2OUXklshld1iOtZCIcrp6ukhJwCM-bVM3UEjCg1EoON_iSWjfNSEIHDYykqlS7v-NCrex2KTzK_0bnT_xT6EYsv90V67J8GhopjdRedfyANCSu6VqZ2hgNEXq-CqdEWf2qpfQioGT3dm78Qd04UvC9hpiFTEuzrpmKencS3x3v125Rbstpe_vzI-f8cSYROfsV4JusDN3-Z96eA"
  }'
```











---

have better knowledge on - system design

http://interviewready.io/course-page/system-design-course?srsltid=AfmBOopZyZwoO-bebCmK2cfDobkI9A00ObFHy2sHXF6_3ECDHKy-UUST

aws https://www.udemy.com/course/aws-certified-developer-associate-dva-c01/?utm_campaign=Search_Keyword_Alpha_Prof_la.ES_cc.ROW-Spanish&utm_source=google&utm_medium=paid-search&portfolio=ROW-Spanish&utm_audience=mx&utm_tactic=nb&utm_term=capacitaci%C3%B3n%20aws&utm_content=g&funnel=&test=&gad_source=1&gad_campaignid=21487757262&gbraid=0AAAAADROdO0-d-vpbTkOOYkF4ttp3kPn7&gclid=Cj0KCQiAtfXMBhDzARIsAJ0jp3BZg1AqmjopWhZsK4IpIDswB_T-ahtJxZNslcjFupUpvvOt7Vl2PN8aAu-7EALw_wcB

exam - https://aws.amazon.com/certification/certified-developer-associate/

Bytebyte code
https://bytebytego.com/
https://bytebytego.com/guides/api-web-development/

try to build micro services projects, ai projects (like book my show)

