# Create Ingress

To expose the vault API and UI outside the cluster we need to setup an Ingress. Complete the `vault-ingress.yaml` file with the service name and port as well as the correct host (with your userid). You need to use the same domain as the cluster console (starts with apps.***.com).

Reference: https://kubernetes.io/docs/concepts/services-networking/ingress/#the-ingress-resource.

Once completed apply the file:
```
oc apply -f vault-ingress.yaml
```

You can now access the vault api via the host you setup:
```
export VAULT_HOST=$(oc get ingress myvault -o jsonpath='{.spec.rules[0].host}')
curl -s http://$VAULT_HOST/v1/sys/health | jq
```

# Open Vault UI

In addition to the API you have a UI available:
```
# for MacOS
open http://$VAULT_HOST/ui
# for Windows
start http://$VAULT_HOST/ui
```

# Test Vault

By providing the vault token in a HTTP Header you can make authenticated calls to the API and query our secret:
```
export VAULT_TOKEN=...
curl -s --header "X-Vault-Token: $VAULT_TOKEN" http://$VAULT_HOST/v1/secret/foo | jq
```