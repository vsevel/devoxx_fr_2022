# Create vault-auth-delegator ClusterRolebinding
```
oc apply -f cluster-role-auth-delegator.yaml
```

# Configure Kubernetes Auth + Vault policy and role
```
oc exec -it $(oc get pods -l app=vault -o jsonpath='{.items[0].metadata.name}') -- sh
  export VAULT_TOKEN=...
  export VAULT_ADDR=http://127.0.0.1:8200

  vault auth enable kubernetes

  vault write auth/kubernetes/config kubernetes_host=https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT

  vault read auth/kubernetes/config

cat <<EOF | vault policy write mypolicy -
path "secret/foo" {
  capabilities = ["read", "list"]
}
EOF

  vault policy read mypolicy

  vault write auth/kubernetes/role/myapprole \
      bound_service_account_names='*' \
      bound_service_account_namespaces='vault-user1-app' \
      policies=mypolicy ttl=2h

exit
```

# Create demo namespace
```
oc new-project vault-user1-app
```

# Get token from app pod
```
oc apply -f busybox.yaml

export K8S_JWT=$(oc exec busybox -- cat /var/run/secrets/kubernetes.io/serviceaccount/token)
echo -e "{\"jwt\": \"$K8S_JWT\", \"role\": \"myapprole\"}" > login.json
export VAULT_CLIENT_TOKEN=$(curl -s --request POST --data @login.json http://$VAULT_HOST/v1/auth/kubernetes/login | jq -r '.auth.client_token')
echo $VAULT_CLIENT_TOKEN
```

# Display secret
```
$ curl -s --header "X-Vault-Token: $VAULT_CLIENT_TOKEN" http://$VAULT_HOST/v1/secret/foo | jq '.data'
{
  "password": "bar"
}
```

Reference: https://www.vaultproject.io/docs/auth/kubernetes

