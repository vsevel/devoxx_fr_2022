# Create namespace

```
oc new-project vault-<userId>
```

# Create Vault and Service

```
oc apply -f vault-deployment.yaml
oc apply -f vault-service.yaml
```

# Initialize and Unseal Vault

```
export VAULT_POD=$(oc get pods --output=jsonpath={.items..metadata.name})
oc exec -it $VAULT_POD -- sh
    export VAULT_ADDR=http://127.0.0.1:8200
    vault status
    vault operator init -key-shares=1 -key-threshold=1
        => Unseal Key 1: ...
        => Initial Root Token: ...
    export VAULT_KEY=...
    export VAULT_TOKEN=...
    vault operator unseal $VAULT_KEY
    vault secrets enable -path=secret kv
    vault write secret/foo password=bar
    vault read secret/foo

    wget -qO - http://vault:8200/v1/sys/health
exit
```