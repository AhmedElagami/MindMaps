# Node Infrastructure

## Content

### ❓ What types of infrastructure can Kubernetes nodes represent and what guarantee does Kubernetes provide about container scheduling within a Pod?
Nodes are host servers that can be physical servers, virtual machines, or cloud instances.

Kubernetes guarantees that all containers in a Pod will be scheduled to the same cluster node.

### ❓ What makes the custom k3d image used in this cluster setup different from standard k3d images?
The custom k3d image used in this cluster setup (ghcr.io/deislabs/containerd-wasm-shims/examples/k3d:v0.11.1) is different from standard k3d images because it includes pre-installed Wasm shims that other clusters might not include. This special image contains containerd Wasm shims pre-configured and ready to use, making it specifically designed for running WebAssembly workloads on Kubernetes. Standard k3d images typically only include the default runc shim for traditional Linux containers, whereas this custom image includes multiple Wasm shims (Spin, Slight, Lunatic, and wws) in addition to the standard runc shim.

### ❓ What does the following k3d cluster creation command do, and what is the purpose of each flag used?
The k3d cluster create command builds a new multi-node Kubernetes cluster specifically configured for Wasm workloads. Here's the command with an explanation of each flag:

```bash
$ k3d cluster create wasm \
      --image ghcr.io/deislabs/containerd-wasm-shims/examples/k3d:v0.11.1 \
      -p "5005:80@loadbalancer" --agents 2
```

- The first line creates a new cluster called "wasm"
- The `--image` flag specifies a custom k3d image that includes pre-installed containerd Wasm shims (ghcr.io/deislabs/containerd-wasm-shims/examples/k3d:v0.11.1)
- The `-p` flag creates a load balancer that connects to an Ingress on the cluster and maps port 5005 on your host machine to an Ingress on port 80 inside the cluster
- The `--agents` flag creates two worker nodes

This command creates a 3-node cluster (one control plane node and two workers) with all nodes running the same Wasm shims, making them capable of running Wasm applications.

