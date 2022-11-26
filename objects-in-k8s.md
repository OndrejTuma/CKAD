# Objects in Kubernetes

They are also referred to as `kind` or `type`.

They are by default PascalCase when used in configuration files or as `--type` option,
but lowercase when used as an `<object>` argument.

> There are also aliases for k8s objects (mentioned in brackets)

## ClusterRole

ClusterRole is a cluster level, logical grouping of PolicyRules that can be 
referenced as a unit by a RoleBinding or ClusterRoleBinding.

### Create

`kubectl create clusterrole NAME --verb=verb --resource=resource.group [--resource-name=resourcename] [--dry-run=server|client|none] [options]`

### Example

`kubectl create clusterrole cluster-administrator --verb=list,get,create,delete --resource=nodes`

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

## ClusterRoleBinding

ClusterRoleBinding references a ClusterRole, but not contain it. It can 
reference a ClusterRole in the global namespace, and adds who information 
via Subject.

### Create

`kubectl create clusterrolebinding NAME --clusterrole=NAME [--user=username] [--group=groupname] [--serviceaccount=namespace:serviceaccountname] [--dry-run=server|client|none] [options]`

### Example

`kubectl create clusterrolebinding cluster-admin-role-binding --clusterrole=cluster-administrator --user=cluster-admin --group=group1`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-role-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-administrator
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: cluster-admin
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: group1
```

## ConfigMap (cm)

ConfigMap holds configuration data for pods to consume.

### Create

`kubectl create configmap NAME [--from-file=[key=]source] [--from-literal=key1=value1] [--dry-run=server|client|none] [options]`

### Example

`kubectl create cm my-map --from-literal=foo=bar --from-literal=foo1=bar1`

```yaml
apiVersion: v1
data:
  foo: bar
  foo1: bar1
kind: ConfigMap
metadata:
  name: my-map
```

## CronJob (cj)

CronJob represents the configuration of a single cron job.

### Create

`kubectl create cronjob NAME --image=image --schedule='0/5 * * * ?' -- [COMMAND] [args...] [flags] [options]`

### Example

`kubectl create cj reporting-cron-job --image=reporting-tool --schedule='*/1 * * * *'`

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: reporting-cron-job
spec:
  schedule: '*/1 * * * *'
  jobTemplate:
    spec:
      completions: 3 # Specifies the desired number of successfully finished pods the job should be run with (https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/)
      parallelism: 3 # Specifies the maximum desired number of pods the job should run at any given time
      template:
        spec:
          containers: 
            - name: reporting-cron-job
              image: reporting-tool
          restartPolicy: OnFailure
```

## Deployment (deploy)

Deployment enables declarative updates for Pods and ReplicaSets.

Whenever new deployment is created or update a rollout process is created.
That takes care of transition from one version to another
And when rollout is created, new revision is also created.

Deployment contains ReplicaSet.

When rollout or rollback it creates another ReplicaSet
and then based on `StrategyType` it destroys pods in old ReplicaSet
and creates new pods in new ReplicaSet

### Create

`kubectl create deployment NAME --image=image -- [COMMAND] [args...] [options]`

### Strategy types

- RollingUpdate (default)
- Recreate
- Blue/Green
  - this is not a value to set, but rather approach
  - each new app version gets different label (version: v1, v2, etc)
  - when its tested and ready, we switch service selector to route traffic to new version
- Canary
  - this is not a value to set, but rather approach
  - each new app version has common selector for service
  - new app version is created with only one pod or so
  - this way we can test it while only small portion of traffic routes to it
  - after its tested, replicas increse and old deployment is killed

## Ingress (ing)

Ingress exposes HTTP and HTTPS routes from outside the cluster
to services within the cluster. Traffic routing is controlled
by rules defined on the Ingress resource.

### Create

`kubectl create ingress NAME --rule=host/path=service:port[,tls[=secret]] [options]`

### Example

`kubectl create ing minimal-ingress --class=nginx-example --rule="/testpath*=test:80" --annotation=nginx.ingress.kubernetes.io/rewrite-target=/`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    # this annotation removes the need for a trailing slash when calling urls
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  # IngressClassName is the name of the IngressClass cluster resource.
  # The associated IngressClass defines which controller will implement the resource.
  # find the class name with: `kubectl get ingressclass`
  ingressClassName: nginx-example
  rules:
    - http:
        paths:
          - path: /testpath
            # Exact: Matches the URL path exactly. 
            # Prefix: Matches based on a URL path prefix split by '/' (/foo/bar matches /foo/bar/baz, but does not match /foo/barbaz)
            pathType: Prefix 
            backend:
              service:
                name: test
                port:
                  number: 80
```

## Job

Executes number of pods designed to run its instructions and the exit.

Outputs of the pods will be result of the instructions

Works like replicaset but it does not want the pods to live forever

### Create

`kubectl create job NAME --image=image [--from=cronjob/name] -- [COMMAND] [args...] [options]`

### Example

`kubectl create job math-add-job --image ubuntu -- expr 3 + 2`

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
        - name: math-add-job
          image: ubuntu
          command: ['expr', '3', '+', '2']
      restartPolicy: Never
```

## LimitRange (limits)

LimitRange sets resource usage limits for each kind of resource .
Sets resources defaults when new Pods are created (in a Namespace)

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
      memory: 512Mi # pods will have spec.containers.resources.limits.memory = 512Mi
    defaultRequest:
      memory: 256Mi # pods will have spec.containers.resources.requests.memory = 256Mi
    type: Container
```

You can run `kubectl top` to view metrics in Pod

## Namespace (ns)

### Create

`kubectl create ns NAME`

## NetworkPolicy (netpol)

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

## PersistentVolume (pv)

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
  # What happens to a persistent volume when released from its claim.
  persistentVolumeReclaimPolicy: Retain # Retain | Delete | Recycle
```

## PersistentVolumeClaim (pvc)

PersistentVolumeClaim is a user's request for and claim to a persistent volume.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```

## Pod (po)

### Create

`kubectl run NAME --image=image [--env="key=value"] [--port=port] [--dry-run=server|client] [--overrides=inline-json]
[--command] -- [COMMAND] [args...] [options]`

**Options**

- `--labels` (--labels="app=hazelcast,env=prod")

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

### Lifecycle

**Status:**

Pending - ContainerCreating - Running

### Conditions

- PodScheduled
- Initialized
- ContainersReady
- Ready

## ReplicaSet (rs)

## ResourceQuota (quota)

ResourceQuota sets aggregate quota restrictions enforced per namespace

### Create

`kubectl create quota NAME [--hard=key1=value1,key2=value2] [--scopes=Scope1,Scope2] [--dry-run=server|client|none]
[options]`

### Example

`kubectl create quota my-quota --hard=cpu=1,memory=1G,pods=2,services=3,replicationcontrollers=2,resourcequotas=1,secrets=5,persistentvolumeclaims=10 --scopes=BestEffort`

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-quota
spec:
  hard:
    cpu: "1"
    memory: 1G
    persistentvolumeclaims: "10"
    pods: "2"
    replicationcontrollers: "2"
    resourcequotas: "1"
    secrets: "5"
    services: "3"
  scopes:
    - BestEffort
```

## Role

Role is a namespaced, logical grouping of PolicyRules that can be 
referenced as a unit by a RoleBinding.

### Create 

Create a role with single rule:

`kubectl create role NAME --verb=verb --resource=resource.group/subresource [--resource-name=resourcename] [--dry-run=server|client|none] [options]`

### Example

`kubectl create role developer --verb=list,get,create,update,delete --resource=pods --resource-name=mypod`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
  - apiGroups: [""] # for core group (v1) we can leave it blank
    resources: ["pods"]
    verbs: ["list", "get", "create", "update", "delete"]
    resourceNames: ["mypod"] # pods in a node that user can access
  - apiGroups: [""]
    resources: ["ConfigMap"]
    verbs: ["create"]
```

## RoleBinding

To bind created Role to a user

### Create

`kubectl create rolebinding NAME --clusterrole=NAME|--role=NAME [--user=username] [--group=groupname] [--serviceaccount=namespace:serviceaccountname] [--dry-run=server|client|none] [options]`

### Example

`kubectl create rolebinding admin --user=admin-user --role=administrator --group=best-admins`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: administrator
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: admin-user
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: best-admins
```

## Secret

Secret holds secret data of a certain type. The total bytes of the values
in the Data field must be less than MaxSecretSize bytes.

### TLS secret
#### Create 

`kubectl create secret tls NAME --cert=path/to/cert/file --key=path/to/key/file [--dry-run=server|client|none] [options]`

#### Example

`kubectl create secret tls tls-secret --cert=path/to/tls.cert --key=path/to/tls.key`

### GENERIC secret

#### Create

`kubectl create secret generic NAME [--type=string] [--from-file=[key=]source] [--from-literal=key1=value1] [--from-env-file=[key=]source] [--dry-run=server|client|none] [options]`

#### Example

`kubectl create secret generic my-secret --from-literal=key1=supersecret`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
data:
  key1: c3VwZXJzZWNyZXQ=
```

### Encode/decode strings to secret

**Encode:**

```shell
echo -n 'secret' | base64
```

**Decode:**

```shell
echo -n 'c2VjcmV0' | base64 --decode
```

## Service (svc)

Like a virtual server inside a node.
It has its own IP address inside the cluster

### Create

Exposing (pod, deploy):

`kubectl expose (-f FILENAME | TYPE NAME) [--port=port] [--protocol=TCP|UDP|SCTP] [--target-port=number-or-name] [--name=name] [--external-ip=external-ip-of-service] [--type=type] [options]`

Creating NodePort svc:

`kubectl create service nodeport NAME [--tcp=port:targetPort] [--dry-run=server|client|none] [options]`

Creating ClusterIP svc:

`kubectl create service clusterip NAME [--tcp=<port>:<targetPort>] [--dry-run=server|client|none] [options]`

### Example

`kubectl create svc nodeport myapp-service --tcp=80:8080 --node-port=30008`

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: myapp-service
  name: myapp-service
spec:
  type: NodePort
  ports:
  - port: 80 # only one required
    targetPort: 8080 # if not provided, it is assumed to be the same as `port`
    nodePort: 30008 # if not provided, will be allocated automatically
  selector: # labels of the pod to match
    app: myapp-service
```

### Types

#### NodePort

Service makes an internal pod accessible on a port on a node.

nodePort as a value can range between 30000 - 32767

#### ClusterIP (default)

Service creates a virtual IP inside the cluster to enable communication between different services (set of FE servers to a set of BE servers)

#### LoadBalancer

Provisions a load balancer for application in supported cloud providers.

## ServiceAccount (sa)

ServiceAccount binds together: 
- a name, understood by users, and perhaps by peripheral systems, for an identity 
- a principal that can be authenticated and authorized 
- a set of secrets

When created, it creates ServiceAccount object and then
generates a token for the ServiceAccount and then
creates a Secret and stores the token there. Then it links the
secret to the ServiceAccount. This token can be used as a bearer token
for REST API requests

When creating a Pod, default ServiceAccount is created for it.
But it has limited access.

### Create

`kubectl create serviceaccount NAME [--dry-run=server|client|none] [options]`

### Example

`kubectl create serviceaccount my-service-account`

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
```
