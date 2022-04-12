To demonstrate the use of readiness probes, start by killing one of the vault pods:
```
oc delete pod $(oc get pods -l app=vault -o jsonpath='{.items[0].metadata.name}')
```

Attempt to get the `foo` secret by repeating this request multiple times (to make sure each pod will be accessed):
```
export VAULT_TOKEN=...
curl -s --header "X-Vault-Token: $VAULT_TOKEN" http://$VAULT_HOST/v1/secret/foo | jq
```

You should see some successful requests, and some failed ones.

Similarly, get the health of the pods through the service by executing the followinf request multiple times:
```
curl -s http://$VAULT_HOST/v1/sys/health | jq
```

Why is the request failing around 50% of the time?

Some requests will return will indicate `"sealed": true` (the restarted instance), and some others `"sealed": false`. The sealed instance is returning the errors.

The issue is that requests are sent half of the time to the sealed instance. How can we discard this instance to receive requests untils it gets unsealed?

Check the current readiness probe, and experiment with `health` api options. Execute multiple times the following request:
```
export VAULT_OPTS='standbyok=true&standbycode=200&sealedcode=200&uninitcode=200'
curl -s -v http://$VAULT_HOST/v1/sys/health?$VAULT_OPTS | jq
```

What is the http code on the sealed instance? How do we change that to return an error code that makes the readiness probe fail?

Reference: https://www.vaultproject.io/api-docs/system/health

Now remove the `sealedcode` option on the previous request, and re-execute multiple times:
```
export VAULT_OPTS='standbyok=true&standbycode=200&uninitcode=200'
curl -s -v http://$VAULT_HOST/v1/sys/health?$VAULT_OPTS | jq
```

How does it change the return code on the sealed instance request?

Now apply this change of configuration on the vault Deployment readiness probe, and redeploy:
```
oc apply -f vault-deployment.yaml
```

Connect on the first vault instance:
```
oc exec -it $(oc get pods -l app=vault -o jsonpath='{.items[0].metadata.name}') -- sh
```

And unseal it:
```
export VAULT_KEY=...
export VAULT_ADDR=http://127.0.0.1:8200
vault operator unseal $VAULT_KEY
exit
```

Check readiness on both vault pods:
```
oc get pods
```

You should see something like this:
```
NAME                     READY   STATUS    RESTARTS   AGE
...
vault-69f84ccf6f-88lhw   1/1     Running   0          2m54s
vault-69f84ccf6f-tcc8v   0/1     Running   0          2m54s
```

Re-execute the secret retrieval request, and make sure you only get successes:
```
export VAULT_TOKEN=...
curl -s --header "X-Vault-Token: $VAULT_TOKEN" http://$VAULT_HOST/v1/secret/foo | jq
```

Unseal the second pod:
```
oc exec -it $(oc get pods -l app=vault -o jsonpath='{.items[1].metadata.name}') -- sh
```

Inside the pod, execute:
```
export VAULT_KEY=...
export VAULT_ADDR=http://127.0.0.1:8200
vault operator unseal $VAULT_KEY
exit
```

Check status for both pods again:
```
oc get pods
```

You should see something like this:
```
NAME                     READY   STATUS    RESTARTS   AGE
...
vault-69f84ccf6f-88lhw   1/1     Running   0          2m54s
vault-69f84ccf6f-tcc8v   1/1     Running   0          2m54s
```

Unsealing the pod made it `Ready`.