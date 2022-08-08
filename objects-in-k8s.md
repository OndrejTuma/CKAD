## Objects in Kubernetes

They are also reffered to as `kind` or `type`.

The are by default PascalCase when used in configuration files or as `--type` option,
but lowercase when used as an `<object>` argument.

> There are also aliases for k8s objects (mentioned in brackets)

### ClusterRole

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

### ClusterRoleBinding

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
  # What happens to a persistent volume when released from its claim.
  persistentVolumeReclaimPolicy: Retain # Retain | Delete | Recycle
    
```

### PersistentVolumeClaim (pvc)

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

### Role

Creates a role object that can be used for authorization (RBAC)
By default it has access to default namespace, but we can define
another one in metadata section

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

#### Get roles:

```shell
kubectl get roles
```

#### Get more information about a role:

```shell
kubectl describe role <name>
```

### RoleBinding

To bind created Role to a user

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

#### Get role bindings:

```shell
kubectl get rolebinding
```

#### Get more information about a role binding:

```shell
kubectl describe rolebinding <name>
```

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