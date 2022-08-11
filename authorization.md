Objects in k8s can be scoped based on `namespaces` and `clusters`.

Users can be authorized to either one or both.

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

Authorization to namespaces is done via `roles` and `rolebindings`.

In role definition file you can list any of the namespace scoped objects.

When you create a role, you specify to which objects (resources) 
can access and what operations are allowed.
But the role is always bound to a namespace.

For example if you create a role that can create and delete pods, 
you can do so only in defined namespace. If you want to create access
to all pods, use `clusterroles`

### Role definition

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
  - apiGroups: [""] # for core group we can leave it blank
    resources: ["pods"]
    verbs: ["list", "get", "create", "update", "delete"]
    # optional: pods in a namespace that user can access
    resourceNames: ["mypod"] 
```

### RoleBinding definition

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
subjects:
  - kind: User
    name: dev-user
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
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

Authorization to clusters is done via `clusterroles` and 
`clusterrolebindings`.

It works the same way as `roles` and `rolebindings` and you can specify 
any object listed in cluster scoped list, but also namespaced 
list as well - this way you grant access to the object across all 
namespaces.

### ClusterRole definition

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administrator
rules:
  - apiGroups: [""] 
    resources: ["nodes"]
    verbs: ["list", "get", "create", "delete"]
```

### ClusterRoleBinding definition

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
subjects:
  - kind: User
    name: cluster-admin
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: cluster-administrator
  apiGroup: rbac.authorization.k8s.io
```
