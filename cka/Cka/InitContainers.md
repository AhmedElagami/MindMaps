# Init Containers

## Content

### ❓ What are the defining characteristics of init containers in the Kubernetes API, what execution guarantees does Kubernetes provide, what is their primary purpose, how do they handle API availability checks, how are multiple init containers managed, what happens when they fail, and what are their main limitations?
**Init Container Characteristics:**
Init containers are a special type of container defined in the Kubernetes API. You run them in the same Pod as application containers, but Kubernetes guarantees they'll start and complete before the main app container starts. It also guarantees they'll only run once.

**Primary Purpose:**
The purpose of init containers is to prepare and initialize the environment so it's ready for application containers.

**API Availability Check Sequence:**
When using an init container to check remote API availability, Kubernetes starts the init container first when you deploy the Pod. The init container sends periodic requests to the API server until it receives a response. While it's doing this, Kubernetes prevents the application container from starting. However, as soon as the init container receives a response from the API server, it completes, and Kubernetes starts the application.

**Multiple Init Container Management:**
You can list multiple init containers per Pod and Kubernetes runs them in the order they appear in the Pod manifest. They all have to complete before Kubernetes moves on to start regular application containers.

**Failure Handling:**
If any init container fails, Kubernetes attempts to restart it. However, if you've set the Pod's restart policy appropriately, Kubernetes will fail the Pod.

**Main Limitation:**
A drawback of init containers is that they're limited to running tasks before the main app container starts. For something that runs alongside the main app container, you need a sidecar container.

### ❓ Explain what the following full YAML configuration does, including the purpose of the init container and the main application container, and how does Kubernetes handle containers defined under the initContainers block versus those under the containers block, and what is the relationship between init container completion and regular container startup in Kubernetes Pods?
The following YAML defines a multi-container Pod with an init container and an app container:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: initpod
  labels:
    app: initializer
spec:
  initContainers:
  - name: init-ctr
    image: busybox:1.28.4
    command: ['sh', '-c', 'until nslookup k8sbook; do echo waiting for k8sbook service;\
              sleep 1; done; echo Service found!']
  containers:
    - name: web-ctr
      image: nigelpoulton/web-app:1.0
      ports:
        - containerPort: 8080
```

Defining containers under the `initContainers` block makes them init containers that Kubernetes guarantees will run and complete before it starts regular containers. Regular containers are defined under the `containers` block and will not start until all init containers successfully complete.

This example has a single init container called `init-ctr` and a single app container called `web-ctr`. The init container runs a loop looking for a Kubernetes Service called `k8sBook`. It will remain running in this loop until you create the Service. Once you create the Service, the init container will see it and exit, allowing the app container to start.

### ❓ Analyze the output of the following kubectl get pods --watch command and explain what the Init:0/1 status indicates about the Pod's state, and what does the Pending status in the kubectl describe pod output reveal about the initpod's current condition, and interpret the sequence of status changes shown in the following kubectl get pods --watch output after creating the k8sbook Service?
The output of the `kubectl get pods --watch` command shows:

```bash
$ kubectl get pods --watch
NAME      READY   STATUS     RESTARTS   AGE
initpod   0/1     Init:0/1   0          6s
```

The status `Init:0/1` tells you that the init container is still running, meaning the main container hasn't started yet.

If you run a `kubectl describe pod` command, you'll see the overall Pod status is `Pending`:

```bash
$ kubectl describe pod initpod
Name:             initpod
Namespace:        default
Priority:         0
Service Account:  default
Node:             docker-desktop/192.168.65.3
Labels:           app=initializer
Annotations:      <none>
Status:           Pending              <<---- Pod status
<Snip>
```

The Pod will remain in this phase until you create a Service called `k8sbook`.

After creating the Service with `kubectl apply -f initsvc.yml`, the status changes show the following sequence:

```bash
$ kubectl get pods --watch
NAME      READY   STATUS            RESTARTS   AGE
initpod   0/1     Init:0/1          0          15s
initpod   0/1     PodInitializing   0          3m39s
initpod   1/1     Running           0          3m57s
```

This sequence shows that the init container completes when it sees the Service (`Init:0/1` changes to `PodInitializing`), and then the main application container starts (`PodInitializing` changes to `Running` with `1/1` containers ready).

