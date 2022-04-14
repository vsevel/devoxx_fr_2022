# Create namespace

```
oc new-project vault-<userId>
```

# Vault configuration

Vault local configuration can be injected via the environment variable VAULT_LOCAL_CONFIG

reference: https://hub.docker.com/_/vault

Add following environment variable to the deployment (file vault-deployment.yaml) :

```
        - name: VAULT_LOCAL_CONFIG
          value: |
            {"backend":
              {"file":
                {"path": "/vault/file"}
              },
              "default_lease_ttl": "168h",
              "max_lease_ttl": "720h" ,
              "disable_mlock": true,
              "listener":
              { "tcp" :
                { "address" : "0.0.0.0:8200",
                "tls_disable": 1}
              }
            }
```

# Vault storage

Add an empty-dir volume named vault-file-backend to the vault Deployment (file vault-deployment.yaml)

```
      - name: vault-file-backend
        emptyDir: {}
```

and a respective volumemount in following path: /vault/file

```
        - name: vault-file-backend
          mountPath: /vault/file
```

Reference: https://kubernetes.io/fr/docs/tasks/configure-pod-container/configure-volume-storage/

And apply in kubernetes:

```
oc apply -f vault-deployment.yaml
```

# Service

A service is needed to expose the pod. Complete vault-service.yml file by adding:

- the port to expose (8200)

```
  - name: vault
    port: 8200
```

- the label selector (to find the correct pod(s))

```
    app: vault
```

Reference: https://kubernetes.io/docs/concepts/services-networking/service/

And apply in kubernetes:

```
oc apply -f vault-service.yaml
```

# Initialize and Unseal Vault

When just started with a fresh storage, Vault must be initialized and unsealed.

Reference: https://learn.hashicorp.com/tutorials/vault/getting-started-deploy#initializing-the-vault

Find the correct vault pod name:

```
export VAULT_POD=$(oc get pods --output=jsonpath={.items..metadata.name})
```

Enter into the pod

```
oc exec -it $VAULT_POD -- sh
```

and execute the approriate commands:

```
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
