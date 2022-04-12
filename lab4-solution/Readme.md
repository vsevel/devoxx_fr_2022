# Check readiness
```
oc delete pod $(oc get pods -l app=vault -o jsonpath='{.items[0].metadata.name}')
export VAULT_TOKEN=...
curl -s --header "X-Vault-Token: $VAULT_TOKEN" http://$VAULT_HOST/v1/secret/foo | jq
curl -s http://$VAULT_HOST/v1/sys/health | jq
```

# Experiment with Vault health options
## http code always 200
```
export VAULT_OPTS='standbyok=true&standbycode=200&sealedcode=200&uninitcode=200'
curl -s -v http://$VAULT_HOST/v1/sys/health?$VAULT_OPTS | jq
```
## http code 503 on sealed instance
```
export VAULT_OPTS='standbyok=true&standbycode=200&uninitcode=200'
curl -s -v http://$VAULT_HOST/v1/sys/health?$VAULT_OPTS | jq
# => http code 503 on sealed instance
```

# Update Vault deployment
```
oc apply -f vault-deployment.yaml
```

# Unseal first Vault
```
oc exec -it $(oc get pods -l app=vault -o jsonpath='{.items[0].metadata.name}') -- sh
    export VAULT_KEY=...
    export VAULT_ADDR=http://127.0.0.1:8200
    vault operator unseal $VAULT_KEY
exit
```

# Check readiness
```
$ oc get pods
NAME                     READY   STATUS    RESTARTS   AGE
consul-consul-server-0   1/1     Running   0          18m
consul-consul-server-1   1/1     Running   0          18m
consul-consul-server-2   1/1     Running   0          18m
vault-69f84ccf6f-88lhw   1/1     Running   0          2m54s
vault-69f84ccf6f-tcc8v   0/1     Running   0          2m54s
```

# Test Vault
```
export VAULT_TOKEN=...
curl -s --header "X-Vault-Token: $VAULT_TOKEN" http://$VAULT_HOST/v1/secret/foo | jq
```

# Unseal second Vault
```
oc exec -it $(oc get pods -l app=vault -o jsonpath='{.items[1].metadata.name}') -- sh
    export VAULT_KEY=...
    export VAULT_ADDR=http://127.0.0.1:8200
    vault operator unseal $VAULT_KEY
exit
```