# StatefulSet Architecture

## Content

### ❓ Explain what the following StatefulSet YAML configuration does, including how it manages Pod naming, service discovery, and replica management.
The following YAML defines a simple StatefulSet called tkb-sts with three replicas running the mongo:latest image:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: tkb-sts
spec:
  selector:
    matchLabels:
      app: mongo
  serviceName: "tkb-sts"
  replicas: 3
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: ctr-mongo
        image: mongo:latest
        ...
```

**Pod Naming Management:**
- Kubernetes will name the Pods using the format `<statefulset-name>-<ordinal-index>`
- The three replicas will be named: tkb-sts-0, tkb-sts-1, and tkb-sts-2
- These names are predictable and persistent throughout the Pod lifecycle

**Service Discovery:**
- The `serviceName: "tkb-sts"` field designates the governing Service
- This Service (which should be a headless Service) creates DNS records for each Pod
- Pods get DNS names like: tkb-sts-0.tkb-sts.default.svc.cluster.local

**Replica Management:**
- The `replicas: 3` field specifies three Pod replicas
- The StatefulSet controller creates Pods in order (0, then 1, then 2)
- It waits for each Pod to be "running and ready" before starting the next
- The controller handles self-healing, scaling, and ensures observed state matches desired state

**Deployment Process:**
When posted to the API server, the configuration gets persisted to the cluster store, the scheduler assigns the replicas to worker nodes, and the StatefulSet controller continuously reconciles the actual state with the desired state.

