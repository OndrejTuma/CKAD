Objects in k8s can be scoped based on `namespaces` and `clusters`.

Users can be authorized to either one or both.

Authorization to namespaces is done via `roles` and `rolebindings` 
whereas to clusters its done via `clusterroles` and `clusterrolebindings` 

## Namespaced

- pods
- replicasets
- jobs
- deployments
- services
- secrets
- roles
- rolebindings
- PVC

See full list by running:

```shell
kubectl api-resources --namespaced=true
```

## Cluster scoped

- nodes
- PV
- clusterroles
- clusterrolebindings
- certificatesigningrequests
- namespaces

See full list by running:

```shell
kubectl api-resources --namespaced=false
```