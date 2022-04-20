# WIFI

```
SSID hands on lab: devoxxfr-hol
Password: ...
```

# Download and install following tools

- Openshift CLI: `https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable-4.9/`
- helm CLI: `https://helm.sh/docs/intro/install/`
- jq: `https://stedolan.github.io/jq/download/`

for Mac OS
```
brew install openshift-cli
brew install helm
brew install jq
```

# IDE

It is recommended to install a lightweight IDE such as `https://code.visualstudio.com/`, which offers `md` files preview, and treat `TAB` characters as spaces in `yaml` files.


# OpenShift

Access the openshift console at `https://tinyurl.com/devoxxfr-ocp`

Log your openshift client in openshift cluster

```
oc login --username=<username> --password=<password> --server=https://api.cluster-4zmbp.4zmbp.sandbox688.opentlc.com:6443
```

# Create k8s namespace / openshift project:

```
oc new-project vault-<username>
```

with `<username>` as `userX` (as provided on the login card).
