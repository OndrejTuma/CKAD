## Generate definition files

### Get definition from running object

```sh
kubectl get <object> <name> -o yaml
```

### Pod

#### Create with dry run

`<labels>` - comma separated key=value pairs
`<port>` - on which container port to expose the pod
`--expose` - when expose flag is defined along with port, it automatically creates a clusterip service with the same name and port

```sh
kubectl run <name> --image <image> [--labels "<labels>" --port <port> --expose] --dry-run=client -o yaml
```

### Deployment

#### Create with dry run

```sh
kubectl create deployment <name> --image <image> --replicas <replicas> --dry-run=client -o yaml
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

## Commands

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

## Objects in Kubernetes

They are also reffered to as `kind` or `type`.

The are by default PascalCase when used in configuration files or as `--type` option, 
but lowercase when used as an `<object>` argument. 

> There are also aliases for k8s objects

### ConfigMap (cm)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-name
data:
  KEY: value
```

### CronJob

Runs scheduled Job

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: reporting-cron-job
spec:
  schedule: '*/1 * * * *'
  jobTemplate:
    spec:
      completions: 3
      parallelism: 3
      template:
        spec:
          containers: 
            - name: reporting-tool
              image: reporting-tool
          restartPolicy: Never
```

### Deployment

Whenever new deployment is created or update a rollout process is created.
That takes care of transition from one version to another
And when rollout is created, new revision is also created.

Deployment contains ReplicaSet.

When rollout or rollback it creates another ReplicaSet
and then based on `StrategyType` it destroys pods in old ReplicaSet
and creates new pods in new ReplicaSet

#### Strategy types

- RollingUpdate (default)
- Recreate

### Ingress

Ingress exposes HTTP and HTTPS routes from outside the cluster 
to services within the cluster. Traffic routing is controlled 
by rules defined on the Ingress resource.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
    - http:
        paths:
          - path: /testpath
            pathType: Prefix
            backend:
              service:
                name: test
                port:
                  number: 80
```

### Job

Executes number of pods designed to run its instructions and the exit.

Outputs of the pods will be result of the instructions

Works like replicaset but it does not want the pods to live forever

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3 # how many pods shall complete successfully
  parallelism: 3 # by default pod are created sequentially, this will create 3 in parallel
  backoffLimit: 10 # to prevent a Job from quitting before it succeeds
  template:
    spec:
      containers: 
        - name: math-add
          image: ubuntu
          command: ['expr', '3', '+', '2']
      restartPolicy: Never
```

### LimitRange

Sets resources defaults when new Pods are created

[Docs](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/)

[Source](https://kubernetes.io/docs/tasks/configure-pod-container/assign-memory-resource)

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
spec:
  limits:
  - default:
      memory: 512Mi
    defaultRequest:
      memory: 256Mi
    type: Container
```

You can run `kubectl top` to view metrics in Pod

### Namespace (ns)

```shell
kubectl create ns <name>
```

### NetworkPolicy (netpol)

Pods in K8s can communicate with each other. To set restrictions, use NetworkPolicy.
NetworkPolicy describes what network traffic is allowed for a set of Pods

**Ingress** policy set on a Pod(A) allows receiving requests from a certain Pod(s) and allows Pod(A) to send a response to the Pod(s).

**Egress** means to allow Pod(A) to post requests to another Pod(s) and receive response from the Pod. 

```mermaid
WEB -> API -> DB
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              name: api-pod
          # this setup joins podSelector and namespaceSelector -> podSelector && namespaceSelector 
          # we could set `- namespaceSelector` and achieve -> podSelector || namespaceSelector
          namespaceSelector: # to allow communication only for certain namespace
            matchLabels:
              name: prod
        - ipBlock: # to allow communication from certain IP (or IP ranges)
            cidr: 192.168.5.10/32
      ports:
        - protocol: TCP
          port: 3306
  egress:
    - to:
        - ipBlock:
            cidr: 192.168.5.10/32
      ports:
        - protocol: TCP
          port: 80
```

### PersistentVolume (pv)

PersistentVolume (PV) is a storage resource provisioned by an
administrator. It is analogous to a node. [More info](https://kubernetes.io/docs/concepts/storage/persistent-volumes)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-vol1
spec:
  # AccessModes contains all ways the volume can be mounted. More info: https://kubernetes.io/docs/concepts/storage/persistent-volumes#access-modes
  accessModes:
    - ReadWriteOnce # ReadOnlyMany, ReadWriteMany
  capacity:
    storage: 1Gi
  # HostPath represents a directory on the host. Provisioned by a developer or
  # tester. This is useful for single-node development and testing only!
  hostPath:
    path: /tmp/data
  # any storage solution for the volume
  awsElasticBlockStore:
    volumeId: <volume-id>
    fsType: ext4
```

### Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-name
  labels:
    foo: bar
spec:
  securityContext: # overrides security context for all containers
    runAsUser: userId
    capabilities:
      add: ["MAC_ADMIN"]
  serviceAccountName: svc-account-name
  automountServiceAccountToken: false # to prevent loading token from default ServiceAccount 
  tolerations: # so that Pod tolerate tained Node (double quotes mandatory)
    - key: "app"
      operator: "Equal"
      value: "blue"
      effect: "NoSchedule"
  nodeSelector: # select on which Node to place this Pod (same key=value must be set as a label on Node)
    size: Large
  affinity: # more granular way to place Pod in Node
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              key: size
              operator: In # NotIn | Exists etc.
              values:
                - Large
  containers:
    - name: container-name
      image: container-image
      ports:
        - containerPort: 8080 # on which port to expose the container
      securityContext: # overrides security context in container
        runAsUser: userId
        capabilities:
          add: ["MAC_ADMIN"]
      resources: # find more at `kubectl explain pod --recursive | less`
        requests:
          cpu: 1 # 1 core, hyperthread
          memory: '1Gi' # G != Gi
        limits:
          cpu: 2
          memory: '2Gi'
      envFrom: # define env from external source
        - configMapRef:
            name: configmap-name
        - secretRef:
            name: secret-name
      env: # define env variables
        - name: FOO
          value: foo
        - name: BAR
          valueFrom:
            configMapKeyRef:
              name: configmap-name
              key: BAR
        - name: BAZ
          valueFrom:
            secretKeyRef:
              name: secret-name
              key: BAZ
      readinessProbe: # K8s sets POD to ready state when the api responds with 200 status
        httpGet: # For http test (web apps)
          path: /api/ready
          port: 8080
        tcpSocket: # For tcp test (databases)
          port: 3306
        exec: # For executing command
          command:
            - cat
            - /app/is_ready
        initialDelaySeconds: 10 # how long to wait until probe
        periodSeconds: 5 # how often to probe
        failureThreshold: 8 # how many times to attempt to probe (default 3)
      livenessProbe: # check if application in the POD is running properly
        # same API as in readinessProbe
``` 

#### Lifecycle

**Status:**

Pending - ContainerCreating - Running

#### Conditions

- PodScheduled
- Initialized
- ContainersReady
- Ready

### ReplicaSet (rs)

### ResourceQuota

### Secret

Encode data to use in secret:

```shell
echo -n 'secret' | base64
```

Decode hashed secrets:

```shell
echo -n 'c2VjcmV0' | base64 --decode
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-name
data:
  KEY: encodedvalue
```

### Service (svc)

Like a virtual server inside a node. 
It has its own IP address inside the cluster 
and that IP is called the cluster IP of the service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort
  ports:
    - targetPort: 80 # if not provided, it is assumed to be the same as `port`
      port: 80 # only one required
      nodePort: 30008 # if not provided, will be allocated automatically
  selector: # labels from the pod
    app: myapp
```

### ServiceAccount (sa)

#### NodePort

Service makes an internal pod accessible on a port on a node. 

nodePort as a value can range between 30000 - 32767 

#### ClusterIP

Service creates a virtual IP inside the cluster to enable communication between different services (set of FE servers to a set of BE servers) 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP # default
  ports:
    - targetPort: 80
      port: 80
  selector:
    app: myapp
```

#### LoadBalancer

Provisions a load balancer for application in supported cloud providers.

### ServiceAccount

When created, it creates ServiceAccount object and then
generates a token for the ServiceAccount and then
creates a Secret and stores the token there. Then it links the
secret to the ServiceAccount. This token can be used as a bearer token
for REST API requests

When creating a Pod, default ServiceAccount is created for it.
But it has limited access.