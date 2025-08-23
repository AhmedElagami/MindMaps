# StatefulSet Definition

## Content

### ❓ Explain what the following full StatefulSet YAML configuration does, including how it manages replicas, volumes, the governing service, and the Pod template components.
The StatefulSet YAML configuration defines a StatefulSet called "tkb-sts" that manages stateful applications with ordered deployment and stable network identities. Here's what it does:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: tkb-sts                          <<---- Call the StatefulSet "tkb-sts"
spec:
  replicas: 3                            <<---- Deploy three replicas
  selector:
    matchLabels:
      app: web
  serviceName: "dullahan"                <<---- Make this the governing Service
  template:
    metadata:
      labels:
        app: web
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: ctr-web
        image: nginx:latest
        ports:
        - containerPort: 80
          name: web
        volumeMounts:                         ----┐
        - name: webroot                           | Mount this volume
          mountPath: /usr/share/nginx/html    ----┘
  volumeClaimTemplates:                       ----┐
  - metadata:                                     |
      name: webroot                               |
    spec:                                         | 
      accessModes: [ "ReadWriteOnce" ]            | Dynamically create a 10GB volume
      storageClassName: "block"                   | via the "block" storage class
      resources:                                  |
        requests:                                 |
          storage: 10Gi                       ----┘
```

**Replica Management:** The StatefulSet creates 3 replicas (tkb-sts-0, tkb-sts-1, tkb-sts-2) in sequential order, waiting for each one to be running and ready before starting the next.

**Volume Management:** It uses `volumeClaimTemplates` to dynamically create 10GB persistent volumes via the "block" storage class with ReadWriteOnce access mode. Each replica gets its own dedicated volume mounted at `/usr/share/nginx/html`.

**Governing Service:** The `serviceName: "dullahan"` field designates the headless Service that creates DNS SRV records for the StatefulSet replicas and manages the StatefulSet's DNS subdomain.

**Pod Template:** The template section defines the container configuration using nginx:latest image, exposes port 80, and includes volume mounts for the persistent storage.

### ❓ What command is used to deploy a StatefulSet from a YAML file, and what output confirms successful creation?
The following command is used to deploy the StatefulSet:

```bash
$ kubectl apply -f sts.yml
statefulset.apps/tkb-sts created
```

The output 'statefulset.apps/tkb-sts created' confirms successful creation of the StatefulSet.

