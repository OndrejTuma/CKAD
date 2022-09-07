# Generate definition files

## Get definition from running object

```sh
kubectl get <object> <name> -o yaml
```

## Generate definition file from create/run/expose command

### ClusterRole

#### Create with dry run

```shell
kubectl create clusterrole <name> --verb=<verbs> --resource=<resources> --dry-run=client -o yaml
```

`<verbs>` - list,get,create,delete,watch,...
`<resources>` - nodes,pods,...

### Deployment

#### Create with dry run

```sh
kubectl create deployment <name> --image <image> --replicas <replicas> --dry-run=client -o yaml
```

### Pod

#### Create with dry run

`<labels>` - comma separated key=value pairs
`<port>` - on which container port to expose the pod
`--expose` - when expose flag is defined along with port, it automatically creates a clusterip service with the same name and port

```sh
kubectl run <name> --image <image> [--labels "<labels>" --port <port> --expose] --dry-run=client -o yaml
```

### Secret

#### Create a tls secret with dry run

```shell
kubectl create secret tls <name> --key <key> --cert <cert> --dry-run=client -o yaml
```

### Service

#### Create from existing Pod

This will automatically use the `<pod>`'s labels as selectors and
exposes the `<pod>` on port `<port>`

`<type>` - NodePort, ClusterIp, etc.

```shell
kubectl expose pod <pod> --port <port> --name <svc-name> --type <type> --dry-run=client -o yaml
```

#### Create with dry run

This will not use the pods labels as selectors, instead it will **assume selectors as app=<name>**

`<type>` - clusterip, nodeport, etc.
`<nodePort>` - only when <type>=nodeport

```shell
kubectl create service <type> <name> --tcp=<port>:<targetPort> [--node-port=<nodePort>] --dry-run=client -o yaml
```

## Formatting Output with kubectl

```sh
kubectl [command] [TYPE] [NAME] -o <output_format>
```

1. `-o json` Output a JSON formatted API object.
2. `-o name` Print only the resource name and nothing else.
3. `-o wide` Output in the plain-text format with any additional information.
4. `-o yaml` Output a YAML formatted API object.

### Example

```sh
kubectl create namespace test-123 --dry-run=client -o yaml
```