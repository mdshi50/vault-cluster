# Running Vault in docker

**Procedures:**
* Run in docker with `docker-compose up -d` 
* Exec into container with `docker exec -it vault sh`  and initialize with `vault operator init`
* Unseal with 
```
vault operator unseal <Unseal Key 1>
vault operator unseal <Unseal Key 2>
vault operator unseal <Unseal Key 3>

```
* Login with `vault login <Root Token>`
* Create vault secret inside vault cluster with the following procedure.
```
vault secrets enable -path=kvv2 kv-v2
vault kv put kvv2/webapp/config username="static-user" password="static-password"
```
* Create secret inside kubernetes for vault authentication.
```
kubectl create secret generic vault-token-secret \
  --from-literal=token=sec \
  --namespace=default`
```
 # Syncing vault secrets with kubernetes:
 * Install vault using 
 ```
helm repo add external-secrets https://charts.external-secrets.io \
helm install external-secrets external-secrets/external-secrets
 ```
* Apply `SecretStore` CR and then `ExternalSecret`.You will see the reflection as a kubernetes secret named `my-kubernetes-secret` in your applied namespace.

**Snippet of `ExternalSecret` data key:**
```
  data:
    - secretKey: user
      remoteRef:
        key: kvv2/webapp/config
        property: username
```
**Explanation:**
* secretKey: The key under which the secret will be stored in the Kubernetes secret.
* remoteRef.key: The path to the secret in Vault, relative to the kvv2 mount point. In this case, webapp/config corresponds to kvv2/webapp/config.
* remoteRef.property: The specific property within the Vault secret to fetch.
