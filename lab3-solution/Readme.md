# Create Ingress
```
kubectl apply -f vault-ingress.yaml

export VAULT_HOST=$(oc get ingress myvault -o jsonpath='{.spec.rules[0].host}')
curl -s http://$VAULT_HOST/v1/sys/health | jq
```

# Open Vault UI
```
# for MacOS
open http://$VAULT_HOST/ui
# for Windows
start http://$VAULT_HOST/ui
```

# Test Vault
```
export VAULT_TOKEN=...
curl -s --header "X-Vault-Token: $VAULT_TOKEN" http://$VAULT_HOST/v1/secret/foo | jq
```