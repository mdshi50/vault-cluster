# Using vault secret operator:
### Cloning example codebase:
```
git clone https://github.com/hashicorp-education/learn-vault-secrets-operator
```
### Configuration from vault server:
* Exec shell using `kubectl exec --stdin=true --tty=true vault-0 -n vault -- /bin/sh`.fisrtly login to vault using `vault login` and put your root token
* move into tmp and enable auth method
```
cd tmp
vault auth enable -path demo-auth-mount kubernetes
vault write auth/demo-auth-mount/config \
   kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"
```
* Enable kv v2 secret engine with`vault secrets enable -path=kvv2 kv-v2`.
* Create vault policy using following commands.
```
tee webapp.json <<EOF
path "kvv2/data/webapp/config" {
   capabilities = ["read", "list"]
}
EOF
vault policy write webapp webapp.json 
```
* Create role that uses above policy.
```
vault write auth/demo-auth-mount/role/role1 \
   bound_service_account_names=demo-static-app \
   bound_service_account_namespaces=app \
   policies=webapp \
   audience=vault \
   ttl=24h
```
* Create a secret with `vault kv put kvv2/webapp/config username="static-user" password="static-password"`

### Applying VSO:
* Install secret operator using `helm install vault-secrets-operator hashicorp/vault-secrets-operator -n vault-secrets-operator-system --create-namespace --values vault/vault-operator-values.yaml`
* Create a ns where secret needs to be synced.`kubectl create ns app`
* Apply auth secret with `kubectl apply -f vault/vault-auth-static.yaml`
* Apply static secret with `kubectl apply -f vault/static-secret.yaml`

### Rotating secret:
* To rotate secret enter container shell of vault repica with `kubectl exec --stdin=true --tty=true vault-0 -n vault -- /bin/sh`
* update secret with `vault kv put kvv2/webapp/config username="static-user2" password="static-password2"` and see the reflection from kubernetes.
