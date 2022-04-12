These instructions allow to deploy a complete single replica vault with consul storage, including a service, ingress with a role and policy to be used in lab6.

```
export USER_ID=<username>

oc delete ns vault-$USER_ID
oc delete ns vault-$USER_ID-app

export VAULT_HOST=vault-$USER_ID.apps.cluster-4zmbp.4zmbp.sandbox688.opentlc.com

oc new-project vault-$USER_ID

cd devoxx/lab1-solution
oc apply -f vault-service.yaml

cd ../lab2-solution
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm search repo hashicorp/consul
helm upgrade -i consul hashicorp/consul -f values.yaml

cd ../lab3-solution
# replace <username> in ingress definition
oc apply -f vault-ingress.yaml 

cd ../lab4-solution
oc apply -f vault-deployment.yaml 
oc scale --replicas=1 deployment vault

cd ../lab5-solution

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
	vault auth enable kubernetes
	vault write auth/kubernetes/config kubernetes_host=https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT
	vault read auth/kubernetes/config
	
cat <<EOF | vault policy write mypolicy -
path "secret/foo" {
  capabilities = ["read", "list"]
}
EOF

	vault write auth/kubernetes/role/myapprole \
	  bound_service_account_names='*' \
	  bound_service_account_namespaces='vault-<username>-app' \
	  policies=mypolicy ttl=2h
exit


oc new-project vault-$USER_ID-app
oc apply -f busybox.yaml
# warning: following command fails in VSCode/Windows
export K8S_JWT=$(oc exec busybox -- cat /var/run/secrets/kubernetes.io/serviceaccount/token)
echo -e "{\"jwt\": \"$K8S_JWT\", \"role\": \"myapprole\"}" > login.json
export VAULT_CLIENT_TOKEN=$(curl -s --request POST --data @login.json http://$VAULT_HOST/v1/auth/kubernetes/login | jq -r '.auth.client_token')
echo $VAULT_CLIENT_TOKEN
curl -s --header "X-Vault-Token: $VAULT_CLIENT_TOKEN" http://$VAULT_HOST/v1/secret/foo | jq '.data'
```