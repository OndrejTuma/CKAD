When request to k8s is processed, it first goes through 

1. `authentication`
   1. to see if the user performing the request has credentials
2. `authorization`
   1. to see if the user is authorized for the action (list, get, delete, create, ...etc)
3. `admission-controllers`

## Admission Controllers

A layer that is performed before executing the request to validate it even further.

### Basic types of Admission Controllers:

1. `mutating` - can change the request (for example DefaulStorageClass)
2. `validating` - validate request and allow/deny it
3. there can be combination of both as well

### Flow of Admission Controllers

There is specific order in which all allowed Admission Controllers are called:

1. Mutating
2. Validating 
3. Webhooks

### Enabled by default

- AlwaysPullImages
  - ensures that every time a pod is created, the images are always pulled
- DefaulStorageClass
  - observes creation of PVCs and automatically adds a default storage class to them
- EventRateLimit
  - to limit number of requests that API server can handle at a time
- NamespaceExists (deprecated in favor of NamespaceLifecycle)
  - rejects requests to namespaces that don't exist
- NamespaceLifecycle
  - makes sure that requests to a non-existent namespace is rejected and that the default namespaces such as default, kube-system and kube-public cannot be deleted
- etc...

### Not enabled by default

- NamespaceAutoProvision (deprecated)
  - automatically create a namespace if it doesn't exist

### List Enabled Admission Controllers

```shell
kube-apiserver -h | grep enable-admission-plugins
```

ADM-based setup:

```shell
kubectl exec kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep enable-admission-plugins
```

Minikube:

```shell
kubectl exec kube-apiserver-minikube -n kube-system -- kube-apiserver -h | grep enable-admission-plugins
```

### Enable/Disable Admission Controllers

#### edit kube-apiserver service

#### edit config file `/etc/kubernetes/manifests/kube-apiserver.yaml`

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
    - commands:
        - kube-apiserver
        - --authorization-mode=Node,RBAC
        - --enable-bootstrap-token=true
        - ...
        - --enable-admission-plugins=NodeRestriction,NamespaceAutoProvision
        - --disable-admission-plugins=DefaultStorageClass
      image: k8s.gcr.io/kube-apiserver-amd64:v1.11.3
      name: kube-apiserver
```

### Create your own Admission Controller Webhook

We use `MutatingAdmissionWebhook` and `ValidationAdmissionWebhook` for that.
Those webhooks are called as a last of all enabled Admission Controllers.

You need to deploy your own Admission Webhook server that handles this I/O:

#### Input

An `AdmissionReview` object is passed to the webhook as a json, which contains all the information about the request.

#### Output

Webhook responds with `AdmissionReview` object with a result of: 

##### Validating Admission Webhook:

```json
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": true
  }
}
```

##### Mutating Admission Webhook:

```json
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "<value from request.uid>",
    "allowed": true,
    "patch": [{
      "op": "add",
      "path": "/metadata/labels/users",
      "value": "<user-name>"
    }],
    "patchType": "JSONPatch"
  }
}
```

> This server can be deployed as a k8s `deployment` in a cluster.
> We also need to configure a `service` so it can be accessed

We also need to create a `ValidatingWebhookConfiguration` object:

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: "pod-policy.example.com"
webhooks:
  - name: "pod-policy.example.com"
    clientConfig:
      # if we uploaded our webhook server somewhere else
      url: "https://external-server.example.com"
      # if we have our webhook server in our cluster
      service:
        namespace: "webhook-namespace"
        name: "webhook-service"
      # communication should be over tls, so certificate is required
      caBundle: "asdasdasd..."
    # Specify when to call this Admission Controller
    rules:
      - apiGroups: [""]
        apiVersions: ["v1"]
        operations: ["CREATE"]
        resources: ["pods"]
        scope: ["Namespaced"]
```