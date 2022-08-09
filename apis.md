# API

K8s API goes through kube-apiserver

> groups -> versions -> resources -> verbs

### API groups

- /apps
- /extensions
- /networking.k8s.io
- /storage.k8s.io
- /authentication.k8s.io
- /certificates.k8s.io

### API versions

API versions can be `alpha` (v1alpha1, v1alpha2, ...), `beta` (v1beta1, ...) and `GA` (v1, v2, ...)

When new API version is created, previous version can be deprecated:

- alpha - immediately
- beta - after 9 months or 3 releases
- GA - after 12 months or 3 releases

GA is also split to preferred version and storage version described below.

You can **find the preferred version** for particular API group
by running `kubectl proxy 8001&` and `curl localhost:8001/apis/<api-group>`

### API resources

represent objects in k8s. 
You can find to which API group this resource belong 
by running `kubectl explain <resource>` 

- /deployments
- /replicasets
- /statefulsets

### API verbs

- list
- get
- create
- delete
- update
- watch

## Preferred vs Storage version

They are usually the same, but _can_ be different

`Preferred` version is the one used by default. Can be detected via `explain` command.

For example, want to determine preferred version for deployments:

```shell
kubectl explain deployment | grep VERSION
```

`Storage` version on the other hand is the version stored in ETCD, and 
you can't access it by simple command, although you can query etcdctl directly

## Convert config file to different API version

It is a separate plugin, you may need to install in manually. 
Instruction are in k8s web.

```shell
kubectl convert -f <old-file> --output-version <new-api>
```

Example:

```shell
kubectl convert -f nginx.yaml --output-version apps/v1
```