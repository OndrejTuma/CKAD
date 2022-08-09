# Create new API resource

You can create almost anything as a resource.
But first, you need to add Custom Resource Definition (CRD) file,
that tells k8s what to do when this new resource is used (created).

## Create a CRD

### FlightTicket resource

```yaml
apiVersion: flights.com/v1
kind: FlightTicket
metadata:
  name: my-flight-ticket
spec:
  from: Prague
  to: Allborgh
  number: 2
```

### Define a CRD

```yaml
apiVersion: extensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: flighttickets.flights.com
spec:
  scope: Namespaced
  groups: flights.com
  names:
    kind: FlightTicket
    singular: flightticket
    plural: flighttickets # will be listed in kubectl api-resources
    shortNames:
      - ft # you can use kubectl get ft
  versions:
    - name: v1
      served: true
      storage: true # ! only one version can be marked as storage
  schema:
    openAPIV3schema:
      type: object
      properties:
        spec:
          type: object
          properties:
            from:
              type: string
            to:
              type: string
            number:
              type: number
              minimum: 1
              maximum: 10
```

### Create a CRD

```shell
kubectl create -f flightticket-custom-definition.yaml
```

### Operate on FlightTicket objects

```shell
kubectl get flightticket
```

But this only allows to create new resource and store in ETCD datastore.
It does not actually do anything.

For that, we need to create a controller fot that object
