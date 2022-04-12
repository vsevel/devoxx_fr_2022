# Deploy Consul

Add the hashicorp repository

```
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
```

Search for consul template and install it

```
helm search repo hashicorp/consul
helm upgrade -i consul hashicorp/consul -f values.yaml
```

Chart default file for reference: https://github.com/hashicorp/consul-k8s/blob/main/charts/consul/values.yaml

# Test Consul

Identify members

```
oc get pods -w
oc exec consul-consul-server-0 -- consul members
```

Access locally to consul from inside the container

```
oc exec consul-consul-server-0 -- wget -qO - http://127.0.0.1:8500/v1/status/leader
```

Access through the service

```
oc exec consul-consul-server-0 -- wget -qO - http://consul-consul-server:8500/v1/status/leader
```

# Deploy Vault

```
oc apply -f vault-deployment.yaml
```

# Initialize and Unseal Vault

```
oc exec -it $(oc get pods -l app=vault -o jsonpath='{.items[0].metadata.name}') -- sh
    export VAULT_ADDR=http://127.0.0.1:8200
    vault status
    vault operator init -key-shares=1 -key-threshold=1
    export VAULT_KEY=...
    export VAULT_TOKEN=...
    vault operator unseal $VAULT_KEY
    vault secrets enable -path=secret kv
    vault write secret/foo password=bar
    vault read secret/foo
exit
```

# Open Consul UI

```
oc port-forward consul-consul-server-0 8500:8500
# On MacOS
open http://localhost:8500
# On Windows
start http://localhost:8500
```

# Scale up Vault to 2 replicas

```
oc scale --replicas=2 deployment vault
oc get pods -w
```

# Unseal new Vault

```
oc exec -it $(oc get pods -l app=vault -o jsonpath='{.items[1].metadata.name}') -- sh
# unseal new standby vault pod and read value
    export VAULT_KEY=...
    export VAULT_TOKEN=...
    export VAULT_ADDR=http://127.0.0.1:8200
    vault operator unseal $VAULT_KEY
    vault read secret/foo
exit
```

# Deep dive into helm

You can compare the generated statefulSet yaml

```
oc get sts consul-consul-server -o yaml 
```

with the chart template : https://github.com/hashicorp/consul-k8s/blob/main/charts/consul/templates/server-statefulset.yaml
