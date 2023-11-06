Slide 20
```bash
curl -fsSL https://kirr.co/f9t9jt -o bootstrap_vault.sh
sudo chmod +x bootstrap_vault.sh
sudo ./bootstrap_vault.sh
```

Slide 21

https://kirr.co/6k05g8

https://gist.github.com/codingforentrepreneurs/22995fd1411969127604fd9fbabf30d4


Slide 22
```bash
vault operator init
```

Slide 25
```bash
vault login
```

Slide 26
```bash
echo 'export VAULT_ADDR="http://127.0.0.1:8200"' >> ~/.bashrc
source ~/.bashrc
echo $VAULT_ADDR
vault login
```


Slide 28
```bash
vault secrets enable -version=2 kv
```

Slide 29
```bash
vault operator unseal
```

Slide 31
```bash
vault secrets enable -version=2 kv
```

Slide 32
```bash
vault operator seal
```

Slide 33
```bash
vault operator unseal
```

Slide 34
```bash
export VAULT_ADDR="http://<your-ip>:8200"
vault login
vault operator unseal
```

Slide 35
DEMO PAGE
```bash
vault operator init
vault login
echo "export VAULT_ADDR=\"http://127.0.0.1:8200\"" \
>> ~/.bashrc
source ~/.bashrc
echo $VAULT_ADDR
vault login
vault secrets enable -version=2 kv
vault operator unseal
vault secrets enable -version=2 kv
```


Slide 39
```bash
vault operator rekey -init -key-shares=5 -key-threshold=3
vault operator rekey
```
NOTE -> complete the rekey process



Slide 40
```bash
vault token create -field=token
```

Slide 41
```bash
vault token revoke <vault-token>
```

Slide 42
```bash
vault operator generate-root -init
vault operator generate-root
vault operator generate-root \
-decode=<encoded-token> \
-otp=<otp-from-init>
```


Slide 43
# demo
```bash
vault operator rekey -init -key-shares=8 -key-threshold=5
vault operator rekey

vault token create -field=token
vault token revoke <vault-token>

vault operator generate-root -init
vault operator generate-root
vault operator generate-root \
-decode=<encoded-token> \
-otp=<otp-from-init>
```

Slide 45
```bash
vault auth enable userpass
```


Slide 46
```bash
curl -X POST \
—data '{"password": "<pw>"}' \
$VAULT_ADDR/v1/auth/userpass/login/<un>

vault login -method=userpass username=username
```


Slide 47
```bash
vault auth enable userpass

vault write auth/userpass/users/<username> \
password=<good-password>
```

Slide 48
```bash
openssl rand -base64 32
vault write auth/userpass/users/cfe-user \
password=<good-password>


python3 -c "import secrets;print(secrets.token_urlsafe(32))"
```

Slide 49
```bash
export VAULT_ADDR="http://<your-ip>:8200"
vault login -method=userpass username=cfe-user
```

Slide 50
# DEMO
```bash
vault auth enable userpass
openssl rand -base64 32
vault write auth/userpass/users/cfe-user password=<password>

# on local machine
vault login -method=userpass username=cfe-user
vault token create
```

Slide 53


`policies/token-policy.hcl`
```hcl
path "auth/token/*" {
   capabilities =["create", "update", "patch", "read", "delete"]
}
```

```bash
vault policy write user-token-policy ./policies/token-policy.hcl
```

Slide 54
```bash
vault write auth/userpass/users/cfe-user/policies policies="user-token-policy"


vault read auth/userpass/users/cfe-user/
```

Slide 55
```bash
vault token create
vault token create -policy="user-token-policy" -policy="other"
```

Slide 56
```bash
vault secrets enable -path="cfe" -version=2 kv
```

Slide 61

`policies/cfe-kv-secret-policy.hcl`
```hcl
path "cfe/data/project/dev/web" {
   capabilities =["create", "update", "patch", "read", "list"]
}
```
```bash
vault policy write cfe-kv-secret-policy ./policies/cfe-kv-secret-policy.hcl
```


Slide 62
```bash
vault write auth/userpass/users/cfe-user/policies policies="cfe-kv-secret-policy, user-token-policy"
```

Slide 63
```bash
vault kv put -mount="cfe" project/dev/web \
username=cfe-user \
stage=web
```

Slide 64

`samples/key-value-data.json`
```json
{
    "username": "justin",
    "stage": "dev",
    "version": "1.0.1",
    "updated": "11-6-23"
}
```

```bash
vault kv put -mount="cfe" project/dev/web @samples/key-value-data.json
```


Slide 65

```bash
export VTOKEN=$(vault token create -field=token)
curl -X PUT \
--header "X-Vault-Token: $VTOKEN" \
--data '{"data": {"username": "jcurl", "stage": "curly"}}' \
$VAULT_ADDR/v1/cfe/data/project/dev/web
```


Slide 66

```bash
vault kv get -mount=cfe project/dev/web

curl -X GET \
--header "X-Vault-Token: $VTOKEN" \
$VAULT_ADDR/v1/cfe/data/project/dev/web
```

Slide 67

```bash
vault kv get -mount=cfe -version=2 project/dev/web

curl -X GET \
--header "X-Vault-Token: $VTOKEN" \
$VAULT_ADDR/v1/cfe/data/project/dev/web?version=2
```

Slide 68
DEMO

```bash
vault secrets enable -path="cfe" -version=2 kv 
mkdir -p ./policies/
nano ./policies/cfe-kv-secret-policy.hcl
```

```hcl
path "cfe/data/project/dev/web" {
   capabilities =["create", "update", "patch", "read", "list"]
}
```
```bash
vault policy write cfe-kv-secret-policy ./policies/cfe-kv-secret-policy.hcl
vault write auth/userpass/users/cfe-user/policies policies="cfe-kv-secret-policy, user-token-policy"
vault kv put -mount="cfe" project/dev/web \
username=justin \
stage=new
vault kv get -mount="cfe" project/dev/web
vault kv get -mount="cfe" -version=1 project/dev/web
```


Slide 70

```bash
export KUBECONFIG="~/.kube/config"
kubectl get nodes
```


Slide 71

`manifests/1-vault-agent-sa.yaml`
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: vault-auth
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: role-tokenreview-binding
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: vault-auth
  namespace: default
```


 Slide 72

`manifests/2-vault-agent-secret.yaml`
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: vault-auth-secret
  annotations:
    kubernetes.io/service-account.name: vault-auth
type: kubernetes.io/service-account-token
```


Slide 73

```bash
kubectl apply -f manifests/
```


Slide 74

```bash
export SA_JWT_TOKEN=$(kubectl get secret vault-auth-secret -o jsonpath={.data.token} | base64 -d)

export CLUSTER_CERT=$(kubectl config view -o 'jsonpath={.clusters[].cluster.certificate-authority-data}' --raw | base64 -d)

export K8S_HOST=$(kubectl config view -o 'jsonpath={.clusters[].cluster.server}')

# use root token
vault login 

vault auth enable kubernetes
vault write auth/kubernetes/config \
     token_reviewer_jwt="$SA_JWT_TOKEN" \
     kubernetes_host="$K8S_HOST" \
     kubernetes_ca_cert="$CLUSTER_CERT" \
     issuer="https://kubernetes.default.svc.cluster.local"
```



Slide 75

# copy result
```bash
kubectl config view -o 'jsonpath={.clusters[].cluster.certificate-authority-data}' --raw
```

```python
import base64
encoded_value = """<paste text here>"""
print(base64.b64decode(encoded_value).decode())
```


Slide 76
```bash
vault write auth/kubernetes/role/vault-auth-sa-role \
     bound_service_account_names=vault-auth \
     bound_service_account_namespaces=default \
     token_policies=cfe-kv-secret-policy \
     ttl=24h
```


Slide 77
```bash
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm search repo hashicorp/vault-secrets-operator

helm install vault hashicorp/vault  \
--set="injector.enabled=true" \
--set="global.externalVaultAddr=$VAULT_ADDR" \
--set="tlsDisable=true"
```

Slide 78

`manifests/3-deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/role: "vault-auth-sa-role"
        vault.hashicorp.com/agent-inject-secret-config: "cfe/data/project/dev/web?version=9"
        vault.hashicorp.com/agent-inject-template-config: |
          {{- with secret "cfe/data/project/dev/web" -}}
          API_USER="{{ .Data.data.username }}"
          {{- end -}}
    spec:
      serviceAccountName: vault-auth
      containers:
        - name: web
          image: codingforentrepreneurs/vault-py:latest
          env: 
              # - name: API_USER
              #   value: "Manifest test"
              - name: ENV_VERSION
                value: "1"
              - name: ENV_PATH
                value: /vault/secrets/config
              - name: PORT
                value: "8080"
          ports:
            - containerPort: 8080
```

```bash
kubectl apply -f manifests/
```

Slide 79
```bash
kubectl apply -f manifests/
kubectl get pods
kubectl port-forward deployments/web 8080:8080
kubectl logs deployments/web --container vault-agent
kubectl logs deployments/web --container vault-agent-init
```
