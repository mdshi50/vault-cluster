# Using vault in kubernetes cluster
**Helm installation:**
```
helm repo add hashicorp https://helm.releases.hashicorp.com
helm repo update
helm install vault hashicorp/vault --values vault-values.yaml
```
**Unsealing:**
* Exec pod using the following command `kubectl exec -it vault-0 -- /bin/sh`
* You will see the list of pods with `kubectl get pods -l app.kubernetes.io/name=Display` 
```
NAME                                    READY   STATUS    RESTARTS   AGE
vault-0                                 0/1     Running   0          2m12s
vault-1                                 0/1     Running   0          2m12s
vault-2                                 0/1     Running   0          2m12s
vault-agent-injector-56b65c5cd4-k7lbt   1/1     Running   0          2m13s
```
* Initialize vault-0 with one key share and one key threshold.
```
 kubectl exec vault-0 -- vault operator init \
    -key-shares=1 \
    -key-threshold=1 \
    -format=json > cluster-keys.json

```
* Display the unseal key found in cluster-keys.json.`jq -r ".unseal_keys_b64[]" cluster-keys.json`
* Create a variable named VAULT_UNSEAL_KEY to capture the Vault unseal key with`VAULT_UNSEAL_KEY=$(jq -r ".unseal_keys_b64[]" cluster-keys.json)`
* Unseal vault-0 with `kubectl exec vault-0 -- vault operator unseal $VAULT_UNSEAL_KEY`
* Join your remining pods with unsealed pod service.
```
kubectl exec -ti vault-1 -- vault operator raft join http://vault-0.vault-internal:8200
kubectl exec -ti vault-2 -- vault operator raft join http://vault-0.vault-internal:8200
```
* Unseal remaining replicas after joining. Execute following command in remaining pods `kubectl exec vault-{replica-number} -- vault operator unseal $VAULT_UNSEAL_KEY`.

* Display root token with `jq -r ".root_token" cluster-keys.json`
* Start an interactive shell session on the vault-0 pod with `kubectl exec --stdin=true --tty=true vault-0 -- /bin/sh`
* login to vault using `vault login` and put the root token.
