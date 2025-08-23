# Containerd Runtime

## Content

### ❓ What is the role of containerd in a standard Kubernetes architecture regarding container management and in the standard workflow, what component does containerd instruct to build and start containers?
The most common configuration is Kubernetes using containerd to manage these lower-level tasks. In the example, containerd receives the work tasks and instructs runc to build the containers and start the apps. Once the containers are running, runc exits, leaving the shim processes to maintain the connections between the running containers and containerd.

### ❓ What does Figure 9.2 illustrate about the relationship between Kubernetes, containerd, and runc on a worker node?
![Figure 9.2](media/figure9-2.png)

### ❓ How does the architectural abstraction below containerd enable the integration of Wasm runtimes into Kubernetes and what does Figure 9.3 demonstrate about running both standard containers and Wasm workloads?
In this architecture, everything below containerd is hidden from Kubernetes. This means you can replace runc and the standard shim with a Wasm runtime and a Wasm shim. Figure 9.3 shows the same node running two additional Wasm workloads.

![Figure 9.3](media/figure9-3.png)

### ❓ What does Figure 9.4 illustrate about the structure of a Wasm shim and what does Figure 9.5 demonstrate about Kubernetes cluster configuration with different Wasm shims?
Figure 9.4 illustrates the structure of a Wasm shim, showing that it is a single binary that includes both the shim code and the Wasm runtime code. The Wasm shim code that interfaces with containerd is runwasi, but each shim can embed a specific Wasm runtime. For example, the Spin shim embeds the runwasi Rust library and the Spin runtime code, and the Slight shim embeds runwasi and the Slight runtime. In each shim, the embedded Wasm runtime creates the Wasm host and executes the Wasm app, while runwasi does all the translating and interfacing with containerd.

![Wasm Shim Architecture Diagram](media/figure9-4.png)

Figure 9.5 demonstrates a Kubernetes cluster with two nodes running different shims - one running the WasmEdge shim and the other running the Spin shim. In configurations like this, Kubernetes needs help scheduling workloads to nodes with the correct shims, which is provided through node labels and RuntimeClass objects. Node 2 in the diagram has the =spin label, and Kubernetes has a RuntimeClass object that selects on this label and specifies the target runtime in the handler property, ensuring any Pod referencing this RuntimeClass will be scheduled to Node 2 and use the Spin runtime.

![Multi-Shim Kubernetes Cluster Diagram](media/figure9-5.png)

### ❓ Explain the relationship between runwasi and specific Wasm runtimes within the shim architecture and how node labels and RuntimeClass objects work together to schedule Wasm workloads to appropriate nodes.
The relationship between runwasi and specific Wasm runtimes within the shim architecture is that runwasi provides the generic interface code that translates between containerd and the Wasm runtime, while each specific Wasm runtime (Spin, Slight, etc.) is embedded within its respective shim. As stated: "The Wasm shim code that interfaces with containerd is runwasi, but each shim can embed a specific Wasm runtime" and "In each shim, the embedded Wasm runtime creates the Wasm host and executes the Wasm app, while runwasi does all the translating and interfacing with containerd."

Node labels and RuntimeClass objects work together to schedule Wasm workloads to appropriate nodes by using a labeling system where nodes are marked with specific labels (e.g., =spin) and RuntimeClass objects are created that specify both the node selector requirements and the target runtime handler. The nodeSelector field in the RuntimeClass ensures that Pods referencing this RuntimeClass will only be scheduled to nodes with the matching label, while the handler field tells containerd on the node which specific shim to use for executing Wasm apps. This ensures proper scheduling in heterogeneous environments where different nodes have different Wasm runtimes installed.

### ❓ What are the two essential requirements for a Kubernetes node to be capable of running Wasm workloads?
A Kubernetes node requires both of the following to be capable of running Wasm workloads:

1. **containerd installed and running** - The container runtime must be properly installed and operational on the node
2. **A containerd Wasm shim installed and registered** - The node must have the appropriate Wasm shim files present in the filesystem AND registered with containerd as part of the containerd configuration

Both conditions must be met - the presence of shim files alone is insufficient as they also need to be properly configured and registered with containerd to handle Wasm container execution.

### ❓ What does the docker exec command sequence demonstrate about checking containerd status on a worker node?
The docker exec command sequence demonstrates how to verify that containerd is installed and running on a worker node:

```bash
$ docker exec -it k3d-wasm-agent-1 ash

$ ps | grep containerd
PID   USER     COMMAND
98    0        containerd
<Snip>
```

This sequence shows:
1. Using `docker exec -it k3d-wasm-agent-1 ash` to execute into the worker node's shell
2. Running `ps | grep containerd` to check running processes and filter for containerd
3. The output confirms that containerd is running with PID 98 under user 0 (root), demonstrating that the container runtime is properly installed and operational on the worker node

This is an essential check because containerd must be installed and running for the node to be capable of running container workloads, including Wasm applications.

### ❓ Interpret the output from the ls command filtering for shim files. What types of shims are present and which one is specifically important for Wasm workloads?
The ls command output shows the containerd shim files present on the node:

```bash
$ ls /bin | grep shim
containerd-shim-lunatic-v1
containerd-shim-runc-v2
containerd-shim-slight-v1
containerd-shim-spin-v2
containerd-shim-wws-v1
```

This reveals five different shim types:
- **containerd-shim-runc-v2**: The default shim for executing traditional Linux containers
- **Four Wasm shims**:
  - containerd-shim-lunatic-v1 (Lunatic runtime)
  - containerd-shim-slight-v1 (Slight runtime)
  - containerd-shim-spin-v2 (Spin runtime)
  - containerd-shim-wws-v1 (Wasm Workers Server runtime)

The **containerd-shim-spin-v2** is specifically important for Wasm workloads as it's the Spin shim that will be used to execute Spin applications in Wasm containers. The shim files follow the containerd naming convention, prefixed with "containerd-shim-" and including a version number at the end.

### ❓ Why is the presence of Wasm shim files in the filesystem alone insufficient for running Wasm containers?
The presence of Wasm shim files in the filesystem alone is insufficient for running Wasm containers because the shims also need to be registered with containerd and loaded as part of the containerd configuration. Simply having the shim executable files available doesn't mean containerd knows about them or how to use them. The shims must be properly configured in containerd's runtime configuration so that containerd can invoke the appropriate shim when it needs to execute Wasm containers. This registration process ensures that containerd recognizes the shims as valid runtime handlers and can properly delegate container execution to them.

### ❓ What is the purpose of checking the containerd configuration file in the context of Wasm on Kubernetes?
The purpose of checking the containerd configuration file is to verify that Wasm shim entries are present and properly configured. This check confirms that containerd has the necessary runtime configurations to execute Wasm containers using specific shims like Spin, Slight, WWS, and Lunatic.

### ❓ Explain what the following full configuration block reveals about the Wasm runtimes configured in containerd and what alternative method can be used to verify the active containerd configuration for Wasm shims?
The configuration block reveals that containerd has four Wasm runtimes configured:

1. **spin** runtime using `io.containerd.spin.v2`
2. **slight** runtime using `io.containerd.slight.v1`
3. **wws** runtime using `io.containerd.wws.v1`
4. **lunatic** runtime using `io.containerd.lunatic.v1`

```bash
$ cat /var/lib/rancher/k3s/agent/etc/containerd/config.toml

<Snip>
[plugins.cri.containerd.runtimes.spin]
  runtime_type = "io.containerd.spin.v2"

[plugins.cri.containerd.runtimes.slight]
  runtime_type = "io.containerd.slight.v1"

[plugins.cri.containerd.runtimes.wws]
  runtime_type = "io.containerd.wws.v1"

[plugins.cri.containerd.runtimes.lunatic]
  runtime_type = "io.containerd.lunatic.v1"
```

An alternative method to verify the active containerd configuration is to use the `containerd config dump` command with grep to parse the output for references to specific Wasm shims like Spin.

### ❓ What does the following command sequence do to verify the Spin Wasm shim configuration in containerd?
The command sequence uses `containerd --config` to load the specific configuration file and then `config dump` to output the full configuration, which is then piped to `grep spin` to filter for only the Spin-related configuration entries. This confirms that the Spin runtime is properly configured with the correct runtime type.

```bash
$ containerd --config \
  /var/lib/rancher/k3s/agent/etc/containerd/config.toml \
  config dump | grep spin

<Snip>
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.spin]
    runtime_type = "io.containerd.spin.v2"
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.spin.options]
```

### ❓ What capability does the presence of the Spin Wasm shim provide to a Kubernetes node?
The presence of the Spin Wasm shim means the node can run Spin apps in Wasm containers. This confirms that containerd is running and the Spin Wasm shim is present and registered, enabling the execution of WebAssembly applications using the Spin runtime.

### ❓ What three components are required for a Kubernetes cluster to successfully run Wasm workloads?
Three components are required for a Kubernetes cluster to successfully run Wasm workloads:

1. **Labeled worker nodes**: Nodes must be labeled to identify their Wasm capabilities
2. **Wasm shims installed**: The nodes must have the appropriate Wasm shims (like Spin, Slight, WWS, Lunatic) installed in containerd
3. **RuntimeClass**: A RuntimeClass must be created to schedule Wasm tasks to the appropriate nodes and specify which shim to use

### ❓ What command is used to delete a Wasm image from the local machine, and what important substitution is required?
The command used to delete a Wasm image from the local machine is `docker rmi` followed by the image name. It's important to substitute the name of your specific image, as the example shows `nigelpoulton/k8sbook:wasm-0.1` but you need to use your actual image name.

### ❓ What does the following Docker command do to remove a specific Wasm image?
The Docker command removes a specific Wasm image from the local machine:

```bash
$ docker rmi nigelpoulton/k8sbook:wasm-0.1
```

This command deletes the Docker image with the repository name "nigelpoulton/k8sbook" and tag "wasm-0.1" from the local image storage. You must substitute this with the name of your actual Wasm image.

### ❓ What directory and its contents need to be manually deleted after creating the app, and what caution should be exercised?
After creating the app, you need to manually delete the directory that was created containing all the application artifacts. The specific caution that should be exercised is to "be sure to delete the correct directory!" to avoid accidentally deleting important files or directories.

### ❓ How are Docker and Kubernetes evolving to support Wasm, and what specific capabilities does Docker currently offer for Wasm applications?
Docker and Kubernetes are evolving to support Wasm as it powers the third wave of cloud computing. Docker currently offers three specific capabilities for Wasm applications:

1. **Build Wasm apps into container images**: Docker can compile Wasm applications into standard container images
2. **Run Wasm applications**: Docker has the capability to execute Wasm apps
3. **Host Wasm images on Docker Hub**: Wasm container images can be stored and distributed through Docker Hub

Additionally, projects like containerd and runwasi make it possible to run Wasm containers on Kubernetes, and projects like SpinKube simplify the process.

### ❓ What infrastructure requirements and configuration steps are needed to run Wasm applications on a Kubernetes cluster using containerd?
To run Wasm applications on a Kubernetes cluster using containerd, you need to complete these infrastructure requirements and configuration steps:

1. **Install and register a Wasm shim** on at least one worker node (Kubernetes clusters running containerd have a growing choice of Wasm runtimes implemented as containerd shims)

2. **Label the node** where the Wasm shim is installed

3. **Create a RuntimeClass** that references the node label so the scheduler can properly assign Wasm apps to the appropriate node

In the real world, projects like SpinKube and cloud services automate many of these tasks to simplify the setup process.

