# Build demo application

Go to `Add` menu item in developer view: `https://console-openshift-console.apps.cluster-4zmbp.4zmbp.sandbox688.opentlc.com/add/all-namespaces`

Select your project in dropdown list (`vault-<userId>-app`)

Click on `Import from Git` card

Enter the following repo git URL: `https://github.com/neoludo/vault-demo.git`
And click on the `Create` button at the bottom of the page

A build Config is created and a build launched: you can check the logs in Openshift console interface: `https://console-openshift-console.apps.cluster-4zmbp.4zmbp.sandbox688.opentlc.com/k8s/ns/vault-<userId>-app/builds`

Once build successful, you shoud see a new deployment: `https://console-openshift-console.apps.cluster-4zmbp.4zmbp.sandbox688.opentlc.com/k8s/ns/vault-<userId>-app/deployments`

Edit deployment, and add an emvironment variable: `VAULT_HOST` with value `vault.vault-<userId>.svc.cluster.local`.

Check the pod logs: you should see the following line after application initialization (disregard the first `permission denied` errors on accessing `secret/vault-demo` and `secret/application`, and scroll down to the bottom of the log): 
```
i.m.vaultdemo.VaultDemoApplication       : My password is: *** bar ***
```

Check the secret retrieval endpoint:
```
curl http://vault-demo-git-vault-<userId>-app.apps.cluster-4zmbp.4zmbp.sandbox688.opentlc.com/secret
```

You should see:
```
my secret is: *** bar ***
```
