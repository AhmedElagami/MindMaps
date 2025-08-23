# Pod Manifest Structure

## Content

### ❓ Explain the complete sequence of commands in the Git clone script and what each step accomplishes for setting up the repository.
The Git clone script sets up the repository by performing the following steps:

```bash
$ git clone https://github.com/nigelpoulton/TKB.git
Cloning into 'TKB'...

$ cd TKB

$ git fetch origin

$ git checkout -b 2025 origin/2025
```

1. **git clone**: Downloads the entire repository from GitHub to your local machine, creating a directory called 'TKB'
2. **cd TKB**: Changes into the newly created TKB directory
3. **git fetch origin**: Retrieves the latest changes and branch information from the remote repository without merging them
4. **git checkout -b 2025 origin/2025**: Creates a new local branch called '2025' that tracks the remote 'origin/2025' branch and switches to it

This sequence ensures you have the correct version of the repository with the 2025 branch checked out and ready for use.

### ❓ Analyze the structure of the basic Pod manifest YAML, identifying the purpose of each of the four top-level fields and their sub-components.
The basic Pod manifest YAML structure consists of four top-level fields:

```bash
kind: Pod
apiVersion: v1
metadata:
  name: hello-pod
  labels:
    zone: prod
    version: v1
spec:
  containers:
  - name: hello-ctr
    image: nigelpoulton/k8sbook:1.0
    ports:
    - containerPort: 8080
    resources:
      limits:
        memory: 128Mi
        cpu: 0.5
```

1. **kind**: Tells Kubernetes what type of object you're defining (Pod in this case)
2. **apiVersion**: Specifies what version of the API to use when creating the object (v1 for Pods)
3. **metadata**: Contains identifying information about the Pod:
   - name: hello-pod (names the Pod)
   - labels: zone=prod, version=v1 (used for grouping and selecting Pods)
4. **spec**: Defines the desired state and configuration of the Pod:
   - containers: Array of container specifications
     - name: hello-ctr (container name)
     - image: nigelpoulton/k8sbook:1.0 (container image)
     - ports: containerPort: 8080 (network port)
     - resources: limits: memory: 128Mi, cpu: 0.5 (resource constraints)

### ❓ What does the Pod manifest example define about the container configuration and what container specifications are defined regarding image, ports, and resource requirements?
The Pod manifest example defines a single-container Pod with an application container called hello-ctr. The container is based on the image, listens on port, and tells the scheduler it needs a maximum of 128MB of memory and half a CPU.

### ❓ What are the key benefits of Kubernetes YAML files as documentation sources and how can new team members benefit from reading them?
Kubernetes YAML files are excellent sources of documentation, and you can use them to get new team members up to speed quickly and help bridge the gap between developers and operations. For example, new team members can read your YAML files and quickly learn your application's basic functions and requirements.

### ❓ What specific application requirements can operations teams understand from YAML files?
Operations teams can also use them to understand application requirements such as network ports, CPU and memory requirements, and much more.

### ❓ What additional benefits come from storing YAML files in source control repositories?
You can also store them in source control repositories for easy versioning and running diffs against other versions.

### ❓ What additional information do the -o wide and -o yaml flags provide compared to the basic kubectl get command?
The -o wide flag gives a few more columns but is still a single line of output, while the -o yaml flag gets you everything Kubernetes knows about the object.

### ❓ Analyze the following YAML output from kubectl get -o yaml and explain the purpose of the spec and status sections, including the fundamental difference between them.
The YAML output from `kubectl get pods hello-pod -o yaml` is divided into two main parts: spec and status. The spec section shows the desired state of the object, and the status section shows the observed state. The spec section contains the desired configuration (containers, images, ports, etc.), while the status section shows the current state of the Pod with conditions like Initialized status and timestamps.

```bash
$ kubectl get pods hello-pod -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      <Snip>
  name: hello-pod
  namespace: default
spec:                           <<---- Desired state is in this block
  containers:
  - image: nigelpoulton/k8sbook:1.0
    imagePullPolicy: IfNotPresent
    name: hello-ctr
    ports:
    <Snip>
status:                         <<---- Observed state is in this block
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2024-01-03T18:21:51Z"
    status: "True"
    type: Initialized
  <Snip>
```

### ❓ Where does Kubernetes obtain the additional information that appears in kubectl get -o yaml output beyond what was in the original YAML file, and how does it handle Pod properties that aren't explicitly defined?
Kubernetes obtains additional information from two main sources: Pods have a lot of properties, and anything you don't explicitly define in a YAML file gets populated with defaults. The status section shows you the current state of the Pod and isn't part of your YAML file.

### ❓ What specific objects would be present in the cluster if following along with the hands-on exercises?
If you've been following along, you'll have the following deployed to your cluster:

**Pods**
- hello-pod
- initpod
- k8sbookgit-sync

**Services**
- svc-sidecar

### ❓ What does the following full command sequence do when executed, and what specific objects does it delete?
The command sequence deletes specific pods and services from the Kubernetes cluster:

```bash
$ kubectl delete pod hello-pod initpod git-sync
pod "hello-pod" deleted
pod "initpod" deleted
pod "git-sync" deleted

$ kubectl delete svc k8sbook svc-sidecar
service "k8sbook" deleted
service "svc-sidecar" deleted
```

This command deletes three pods (hello-pod, initpod, git-sync) and two services (k8sbook, svc-sidecar).

### ❓ What alternative method to command-line deletion is mentioned for removing Kubernetes objects?
You can also delete objects via their YAML files.

### ❓ Explain what happens when executing this full command that uses YAML files for deletion, and list all objects it removes.
The command uses YAML manifest files to delete Kubernetes objects:

```bash
$ kubectl delete -f initsidecar.yml -f initpod.yml -f pod.yml -f initsvc.yml
pod "git-sync" deleted
service "svc-sidecar" deleted
pod "initpod" deleted
pod "hello-pod" deleted
service "k8sbook" deleted
```

This command removes the following objects:
- pod "git-sync"
- service "svc-sidecar"
- pod "initpod"
- pod "hello-pod"
- service "k8sbook"

### ❓ Analyze the full output of these kubectl get commands and explain what each column in the pod and service listings represents for the shield Namespace.
The kubectl get commands show the following objects in the shield Namespace:

**Pod listing:**
```bash
$ kubectl get pods -n shield
NAME         READY   STATUS    RESTARTS   AGE
triskelion   1/1     Running   0          48s
```

- **NAME**: The name of the Pod (triskelion)
- **READY**: Container readiness status (1/1 means 1 container out of 1 is ready)
- **STATUS**: Current state of the Pod (Running)
- **RESTARTS**: Number of times containers have restarted (0)
- **AGE**: How long the Pod has been running (48 seconds)

**Service listing:**
```bash
$ kubectl get svc -n shield
NAME      TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
the-bus   LoadBalancer   10.43.30.174   localhost     8080:31112/TCP   52s
```

- **NAME**: The name of the Service (the-bus)
- **TYPE**: Service type (LoadBalancer)
- **CLUSTER-IP**: Internal cluster IP address (10.43.30.174)
- **EXTERNAL-IP**: External access IP (localhost)
- **PORT(S)**: Port mapping (8080:31112/TCP means external port 8080 maps to node port 31112 using TCP)
- **AGE**: How long the Service has been running (52 seconds)

### ❓ Explain the purpose and function of each annotated section in the following full Deployment YAML configuration.
The Deployment YAML configuration contains several key sections with specific purposes:

```bash
kind: Deployment
apiVersion: apps/v1
metadata:
  name: hello-deploy       <<---- Deployment name (must be valid DNS name)
spec:
  replicas: 10             <<---- Number of Pod replicas to deploy & manage
  selector:                
    matchLabels:
      app: hello-world
  revisionHistoryLimit: 5
  progressDeadlineSeconds: 300    
  minReadySeconds: 10
  strategy:                <<---- This block defines rolling update settings
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:                <<---- Below here is the Pod template
    metadata:
      labels:
        app: hello-world
    spec:
      containers:
      - name: hello-pod
        image: nigelpoulton/k8sbook:1.0
        ports:
        - containerPort: 8080
```

- **Deployment name**: Must be a valid DNS name (alphanumerics, dot, and dash only)
- **Replicas**: Number of Pod replicas to deploy and manage (10 in this case)
- **Strategy block**: Defines rolling update settings including type and update parameters
- **Pod template**: Defines the actual Pod specification including container image, labels, and ports

The first two lines tell Kubernetes to create a Deployment object based on the apps/v1 API. The selector contains labels that Deployment and ReplicaSet controllers use to identify which Pods they manage, and it must match the Pod labels in the template section.

### ❓ What is the function of the Pod template section in a Deployment manifest?
The Pod template section in a Deployment manifest defines the Pod that the Deployment will manage. Everything below the template section defines the Pod this Deployment will manage. This example defines a single-container Pod using the specified image.

### ❓ What directory structure is created by the Spin new command, what are the key files generated, and what is the function of each critical file?
The Spin new command creates a new directory called "tkb-wasm" containing everything needed to build and run the app. The directory structure is:

```bash
$ cd tkb-wasm

$ tree
├── Cargo.toml
├── spin.toml
└── src
    └── lib.rs

2 directories, 3 files
```

The two critical files for a Spin Wasm application are:

1. `spin.toml` - tells Spin how to build and run the app (configuration file)
2. `lib.rs` - is the app source code (the actual Rust code that gets compiled to Wasm)

### ❓ What specific modification needs to be made to the Rust source code to change the application's response and which line in the code structure must be modified?
The specific modification needed is to edit the `lib.rs` file so that it returns the text "The Kubernetes Book loves Wasm!" instead of the default response. Only the text on the line indicated by the annotation in the snippet needs to be changed.

The code structure shows where to make this change:

```bash
use spin_sdk::http::{IntoResponse, Request, Response};
<Snip>
fn handle_tkb_wasm(req: Request) -> anyhow::Result<impl IntoResponse> {
    println!("Handling request to {:?}", req.header("spin-full-url"));
    Ok(Response::builder()
        .status(200)
        .header("content-type", "text/plain")
        .body("The Kubernetes Book loves Wasm!")             <<---- Only change this line
        .build())
}
```

The line with the annotation `<<---- Only change this line` must be modified to contain the new response text.

### ❓ What is the purpose of the Pod template section in a StatefulSet definition?
The Pod template section in a StatefulSet definition is where you define the specifications for the Pods that will be created. This includes configuration details such as which container image to use and which ports to expose.

