# Node Types

## Content

### ❓ Explain the two fundamental aspects that define Kubernetes according to the 'Kubernetes from 40K feet' section and what are the key differences between control plane nodes and worker nodes in terms of operating system requirements and primary functions?
Kubernetes is both of the following:

1. **A cluster**: A Kubernetes cluster is one or more nodes providing CPU, memory, and other resources for application use. Kubernetes supports two node types: control plane nodes and worker nodes.
2. **An orchestrator**: Orchestrator is jargon for a system that deploys and manages applications. Kubernetes is the industry-standard orchestrator that can intelligently deploy applications across nodes and failure zones for optimal performance and availability. It can fix things when they break, scale things when demand changes, and manage rollouts and rollbacks.

### ❓ **What are the key differences between control plane nodes and worker nodes?**
- **Control plane nodes**: Must be Linux, implement the Kubernetes intelligence, run control plane services (API server, scheduler, controllers), and every cluster needs at least one control plane node (three or five recommended for high availability).

- **Worker nodes**: Can be Linux or Windows, run business applications, and almost all cloud-native apps are Linux apps requiring Linux worker nodes. A single Kubernetes cluster can have a mix of Linux and Windows worker nodes, and Kubernetes is intelligent enough to schedule apps to the correct nodes.

### ❓ What does the diagram in Figure 2.1 illustrate about the composition of a typical Kubernetes cluster?
Figure 2.1 shows a cluster with three control plane nodes and three workers. The diagram illustrates that a Kubernetes cluster consists of both control plane nodes and worker nodes, with control plane nodes implementing the Kubernetes intelligence and worker nodes running business applications.

![Figure 2.1](media/figure2-1.png)

### ❓ Explain what the following kubectl command output reveals about the cluster's node status, roles, and version information.
The kubectl get nodes command output reveals the following information about the cluster:

- There are three nodes in the cluster
- All nodes are in "Ready" status
- One node (desktop-control-plane) has the "control-plane" role
- Two nodes (desktop-worker and desktop-worker2) have no specific role indicated
- All nodes are running version v1.32.2 of Kubernetes
- All nodes have been running for approximately 10 minutes

```bash
$ kubectl get nodes
NAME                    STATUS   ROLES           AGE   VERSION
desktop-control-plane   Ready    control-plane   10m   v1.32.2
desktop-worker          Ready    <none>          10m   v1.32.2
desktop-worker2         Ready    <none>          10m   v1.32.2
```

### ❓ What does the output of 'kubectl get nodes' indicate about a properly configured LKE cluster, and what specific patterns should you observe in the node names and ROLES column?
```bash
$ kubectl get nodes
NAME                             STATUS    ROLES     AGE    VERSION
lke349416-551020-184a46360000    Ready     <none>    19m    v1.32.1
lke349416-551020-1c6f99c20000    Ready     <none>    19m    v1.32.1
lke349416-551020-47ad6c5c0000    Ready     <none>    19m    v1.32.1
```

For a properly configured LKE cluster, the `kubectl get nodes` command should show:
- **Three nodes** (indicating a healthy cluster)
- **Node names starting with "lke"** (the Linode Kubernetes Engine prefix)
- **STATUS column showing "Ready"** for all nodes
- **ROLES column showing "\<none\>"** for all nodes (indicating they are worker nodes)
- **Consistent version numbers** across all nodes

The specific patterns to observe are that all node names should begin with "lke" and the ROLES column should show "\<none\>" for all nodes, indicating they're all worker nodes.

### ❓ Analyze the node status output from the kubectl get nodes command. What does this reveal about the cluster architecture and node roles?
The `kubectl get nodes` output reveals the cluster architecture and node roles:

```bash
$ kubectl get nodes
NAME                 STATUS    ROLES            AGE    VERSION
k3d-wasm-server-0    Ready     control-plane    17s    v1.27.8+k3s2
k3d-wasm-agent-1     Ready     <none>           15s    v1.27.8+k3s2
k3d-wasm-agent-0     Ready     <none>           15s    v1.27.8+k3s2
```

This output shows a 3-node Kubernetes cluster architecture consisting of:
- **One control plane node**: `k3d-wasm-server-0` with the `control-plane` role, responsible for managing the cluster
- **Two worker nodes**: `k3d-wasm-agent-0` and `k3d-wasm-agent-1` with no specific roles listed (`<none>`), indicating they are worker nodes for running applications

All nodes are in "Ready" status, running Kubernetes version v1.27.8+k3s2, and were created within seconds of each other, indicating a properly functioning multi-node cluster.

