## Commands

### Auth

Find out what authorization level current user has 
(find out if user has access to particular resource in the cluster)

```shell
kubectl auth can-i <action> <object>
kubectl auth can-i create deployments
kubectl auth can-i delete nodes
```

#### Find out permissions for other users

```shell
kubectl auth can-i <action> <object> --as <user>
kubectl auth can-i create pods --as dev-user
```

### Config

Config hold info about clusters in a kubernetes system and accounts created.
Contexts connect user account to cluster.
You can define specific namespace for a context (in kube config file), 
that will be automatically used when the context is used

#### See default config file:

```shell
kubectl config view
```

#### Change context:

```shell
kubectl config use-context <context-name>
```

### Create

#### From a command (imperative)

```sh
kubectl create <object> <name> [options]
```

> For Secret object, use: `kubectl create secret generic <name> [options]`

Options:

- Pod, ReplicaSet, Deployment
  - `--image=<image>`
- ConfigMap, Secret
  - `--from-literal=<key>=<value>` declarable multiple times 
  - `--from-file=<path-to-file>`

#### From a file (declarative)

```shell
kubectl create -f definition.yml
```

### Describe

```shell
kubectl describe <object> <name>
```

### Edit

Edit existing object, results are immediately applied.

```shell
kubectl edit <object> <name>
```

You can edit only limited number of things in a Pod.

- spec.containers[*].image
- spec.initContainers[*].image
- spec.activeDeadlineSeconds
- spec.tolerations

### Exec

Runs command from inside a Pod

```shell
kubectl exec -it <pod-name> <command>
```

### Explain

Shows definition of object, useful for quick
look of how to define objects in definition file. Faster than checking the docs IMHO

```shell
kubectl explain <object[.<key>]> [options] 
```

#### Options

- `--recursive` - shows all definitions

#### Example

```shell
kubectl explain pods --recursive | less
kubectl explain pods --recursive | grep envFrom -A3
```

### Expose

Expose a resource as a new Kubernetes service.

Looks up a deployment, service, replica set, 
replication controller or pod by name 
and uses the selector for that resource 
as the selector for a new service on the specified port. 
A deployment or replica set will be exposed as a service 
only if its selector is convertible to a selector 
that service supports, i.e. when the selector contains only
the matchLabels component. Note that if no port is 
specified via --port and the exposed resource has 
multiple ports, all will be re-used by the new service. 
Also if no labels are specified, the new service 
will re-use the labels from the resource it exposes.

```shell
kubectl expose deployment webapp-deployment --name=webapp-service --target-port=8080 --type=NodePort --port=8080
```

### Get

```shell
kubectl get <object> [options]
```

#### Options

- `--selector, -l <key>=<value>,<key2>=<value2>`
- `--namespace, -n`
- `--show-labels`

#### Example

Get number of pods in dev env

```shell
kubectl get pods -l env=dev --no-headers | wc -l
```

### Label

Label Node so that we can target which Pods will be placed into it (via `nodeSelector` in Pod definition)
More granular approach grant Node Affinity

```shell
kubectl label node <node-name> <label-key>=<label-value>
```

#### Example

```shell
kubectl label node node1 size=Large
```


### Logs

Show logs from a POD. Can target specific container in the POD

```shell
kubectl logs <pod-name> [<pod-container>] [options]
```

#### Options

- `-c, --container <container>`
  - show logs for specific container
- `-f, --follow` 
  - stream the logs

### Proxy

Proxy default kube api server with default credentials from kube config.
The server will be exposed at a port `8001` and you can start curling the 
server without setting up `--key` `--cert` and `--cacert`

```shell
kubectl proxy
```

### Replace

Replace running object with definition file

#### From a file

```sh
kubectl replace -f definition.yml
```

### Rollout

Rollout command of deployment.

#### History

Shows history of rollouts for deployment

```shell
kubectl rollout history deployment <name> [options]
```

Options:

- `--revision` - check the status of each revision individually


#### Status

Shows status of current rollout

```shell
kubectl rollout status deployment <name>
```

#### Undo

Rollback of deployment

```shell
kubectl rollout undo deployment <name>
```

### Run

```shell
kubectl run <name> --image=<image> [options]
```

#### Options

- `--env="<key>=<value>"`
  - Environment variables to set in the container
- `--expose`
  - If true, create a ClusterIP service associated with the pod
  - Requires `--port`
- `-l, --labels="<labels>"`
  - Comma separated labels to apply to the pod
  - Will override previous values
- `--port=<port>`
  - The port that this container exposes

### Set

```shell
kubectl set <subcommand> <object> <name> <key>=<value> [options]
```

#### subcommand

- `env` - Update environment variables on a pod template
- `image` - Update the image of a pod template
  - key=value pair represents container name and new image version
- `resources` - Update resource requests/limits on objects with pod templates
- `selector` - Set the selector on a resource
- `serviceaccount` - Update the service account of a resource
- `subject` - Update the user, group, or service account in a role binding or cluster role binding


> TODO: play with set's capabilities and update this document accordingly

#### Options

- `--record` - record what is being done. Useful for `rollout history`

#### Examples

Update image in the deployment to `nginx:1.17` and record it.
It will be shown in CHANGE-CAUSE column for `rollout history` command  

```shell
kubectl set image deployment nginx nginx=nginx:1.17 --record
```

### Scale

Commands to use when we want to increase number of replicas

#### From a file

```sh
kubectl scale replicas=<number> -f definition.yml
```

#### From running object

`<object>` - replicaset, deployment, etc.

```sh
kubectl scale replicas=<number> <object> <name>
```

### Taint

Taint node so that only Pods with **toleration** will be placed into it

```shell
kubectl taint node <node-name> key=value:taint-effect
```

- **taint-effect** - What happends to Pods that don't tolerate this taint
    - `NoSchedule` - Pods won't be placed in the node
    - `PreferNoSchedule` - try not to place Pods into the node, but no guarantee
    - `NoExecute` - like NoSchedule, but even existing Pods will be evicted if they don't tolerate the taint

#### Example

Taint a node

```shell
kubectl taint node node1 app=blue:NoSchedule
```

Untaint a node (mind the dash)

```shell
kubectl taint node node1 app=blue:NoSchedule-
```

### Top

Get resources of object (typically Pod)

```shell
kubectl top <object> <name> [--namespace=<namespace>]
```