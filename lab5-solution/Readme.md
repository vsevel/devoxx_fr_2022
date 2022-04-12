To allow vault to authenticate the application against Kubernetes, we need to give cluster role `system:auth-delegator` to all vault service accounts (`default` in our case).
Only cluster admins can create this cluster role binding, so it was pre-provisioned on this cluster.
For simplicity, all `system:authenticated` users (inlcuding all service accounts) have been granted this cluster role.

For reference you can check file `allow-users-auth-delegator.yaml` (but do not attempt to apply it).

We are going now to configure the Kubernetes authentication provider in our Vault.

In this first step, enter one of the unsealed vault pods:
```
kubectl exec -it $(kubectl get pods -l app=vault -o jsonpath='{.items[0].metadata.name}') -- sh
```

When inside the pod, configure the authentication provider:
```
export VAULT_TOKEN=...
export VAULT_ADDR=http://127.0.0.1:8200
vault auth enable kubernetes
vault write auth/kubernetes/config kubernetes_host=https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT
vault read auth/kubernetes/config
```

Alternatively, you can configure this authentication from the UI.
In that case, you need to replace `$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT` by `172.30.0.1:443` in the `kubernetes_host` url.

While still in the pod, create a policy `mypolicy` for our application:
```
cat <<EOF | vault policy write mypolicy -
path "secret/foo" {
  capabilities = ["read", "list"]
}
EOF

vault policy read mypolicy
```

Finally, create the `myapprole` vault role that will allow our application in the application namespace to authenticate against vault using its Kubernetes JWT token.
Please note that you have to adjust the application namespace below in property `bound_service_account_namespaces`:
```
vault write auth/kubernetes/role/myapprole \
  bound_service_account_names='*' \
  bound_service_account_namespaces='vault-<userId>-app' \
  policies=mypolicy ttl=2h
exit
```

Now we are going to check that the authentication is working.

First create the application namespace:
```
oc new-project vault-<userId>-app
```

We need a JWT token from a pod in this namespace. The simplest way, is to launch a fake application, and grab its JWT token.

Launch a busybox:
```
kubectl apply -f busybox.yaml
```

Once running, export its JWT token and create a json document that we are going to use as the body of an authentication request on vault:
```
export K8S_JWT=$(oc exec busybox -- cat /var/run/secrets/kubernetes.io/serviceaccount/token)
echo -e "{\"jwt\": \"$K8S_JWT\", \"role\": \"myapprole\"}" > login.json
```

Check that the document is correct:
```
cat login.json
```

You should see:
```
{"jwt": "ey...", "role": "myapprole"}
```

Get a vault token for this JWT token and application vaut role `myapprole`:
```
export VAULT_CLIENT_TOKEN=$(curl -s --request POST --data @login.json http://$VAULT_HOST/v1/auth/kubernetes/login | jq -r '.auth.client_token')
```

Check that you got a value for the vault token:
```
echo $VAULT_CLIENT_TOKEN
```

You should see something like this:
```
s.xwWHBheiAbDmNYOMrWSCVID9
```

And finally display the application secret using this token:
```
curl -s --header "X-Vault-Token: $VAULT_CLIENT_TOKEN" http://$VAULT_HOST/v1/secret/foo | jq '.data'
```

You should see:
```
{
  "password": "bar"
}
```
