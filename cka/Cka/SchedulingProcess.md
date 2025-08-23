# Scheduling Process

## Content

### ❓ Describe the three-step process implemented by the Kubernetes scheduler and what four criteria it checks during node capability assessment?
The scheduler implements the following process: Watch the API server for new tasks, Identify capable nodes, and Assign tasks to nodes. It checks for: Taints, Affinity and anti-affinity rules, Network port availability, and Available CPU and memory.

### ❓ What is the scheduler's role when it encounters a container requiring 256Mi of memory and half a CPU, and what happens if no suitable node is found?
When the scheduler encounters a container requiring 256Mi of memory and half a CPU, it reads these requirements and assigns the Pod to a node with enough resources. If it can't find a suitable node, it marks the Pod as pending, and the Cluster Autoscaler will attempt to provision a new cluster node.

### ❓ What does the output of the kubectl get pods -o wide command reveal about pod scheduling and node placement?
The output of the `kubectl get pods -o wide` command reveals that all three replicas of the wasm-spin Deployment are successfully scheduled and running on the same node (k3d-wasm-agent-1), demonstrating that the RuntimeClass and node labeling worked correctly to constrain pod scheduling to the appropriate node:

```bash
$ kubectl get pods -o wide
NAME                          READY    STATUS     ...    NODE            
wasm-spin-5f6fccc557-5jzx6    1/1      Running    ...    k3d-wasm-agent-1
wasm-spin-5f6fccc557-c2tq7    1/1      Running    ...    k3d-wasm-agent-1
wasm-spin-5f6fccc557-ft6nz    1/1      Running    ...    k3d-wasm-agent-1
```

