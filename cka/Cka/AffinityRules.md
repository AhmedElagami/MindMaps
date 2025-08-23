# Affinity Rules

## Content

### ❓ What is the purpose of labeling a worker node with 'wasm=yes' in a heterogeneous cluster environment?
The purpose of labeling a worker node with 'wasm=yes' in a heterogeneous cluster environment is to help Kubernetes identify which nodes are capable of running Wasm applications. In real-world environments where different nodes have different shims and runtimes (heterogeneous configurations), node labeling allows Kubernetes to properly schedule Wasm workloads to nodes that have the appropriate Wasm capabilities. This labeling, when combined with a RuntimeClass that targets nodes with this specific label, ensures that Pods referencing the Wasm runtime class will only be scheduled to nodes that have the necessary Wasm shims installed and configured, preventing scheduling failures on nodes that lack Wasm support.

### ❓ What is the key difference between k3d cluster node configuration and real-world environments regarding Wasm shims and what two Kubernetes mechanisms are required to handle scheduling in heterogeneous node environments with different Wasm runtimes?
The key difference is that k3d clusters have homogeneous node configurations where all nodes run the same shims, meaning every node can run Wasm apps without additional configuration. In contrast, most real-world environments have heterogeneous node configurations where different nodes have different shims and runtimes.

To handle scheduling in heterogeneous environments, two Kubernetes mechanisms are required:
1. **Node labeling** - to identify which nodes have specific Wasm capabilities
2. **RuntimeClasses** - to help Kubernetes schedule work to the correct nodes based on their capabilities

### ❓ What is the purpose of the following command sequence that includes exiting an exec session and labeling a node?
The command sequence first exits an exec session (`exit`) to return to the host's terminal, then uses `kubectl label nodes` to add the `wasm=yes` label to the k3d-wasm-agent-1 worker node. This labeling operation marks the node as capable of running Wasm workloads.

```bash
# exit

$ kubectl label nodes k3d-wasm-agent-1 wasm=yes
node/k3d-wasm-agent-1 labeled
```

### ❓ What does the output of the following command confirm about the node labeling operation?
The output confirms that the node labeling operation was successful and shows which nodes have the `wasm=yes` label. The grep filter specifically shows nodes with this label, verifying that the k3d-wasm-agent-0 node (and potentially others) now have the Wasm capability label.

```bash
$ kubectl get nodes --show-labels | grep wasm=yes
NAME                STATUS    ROLES     VERSION         LABELS
k3d-wasm-agent-0    Ready     <none>    v1.27.8+k3s2    beta.kubernetes...,wasm=yes
```

### ❓ What is the purpose of creating a RuntimeClass in the context of Wasm workload scheduling and what two functions does the RuntimeClass serve in ensuring proper Wasm workload execution?
The purpose of creating a RuntimeClass is to help Kubernetes schedule Wasm workloads to appropriate nodes that have the required Wasm capabilities. The RuntimeClass serves two critical functions:

1. **Scheduling constraint**: The RuntimeClass ensures that Pods referencing it will only be scheduled to nodes with the specific label (e.g., `wasm=yes`)
2. **Runtime selection**: The RuntimeClass tells containerd on the node to use the specific shim (e.g., `spin`) to execute Wasm apps

The RuntimeClass is created using:
```bash
$ kubectl apply -f rc-spin.yml
runtimeclass.node.k8s.io/rc-spin created
```

### ❓ What does the output of the following command confirm about the RuntimeClass configuration?
The output confirms that the RuntimeClass was created successfully and shows its details:
- **NAME**: rc-spin (the name of the RuntimeClass)
- **HANDLER**: spin (specifies which containerd runtime handler to use)
- **AGE**: 14s (how long it has been active)

This verification ensures the RuntimeClass is properly configured and available for use by Pods.

```bash
$ kubectl get runtimeclass
NAME      HANDLER   AGE
rc-spin   spin      14s
```

### ❓ What does the following YAML code define and what is the purpose of the runtimeClassName field, and how does the RuntimeClass reference in the Pod spec affect Kubernetes scheduling decisions for the wasm-spin Deployment?
The YAML code defines a Kubernetes Deployment named "wasm-spin" with 3 replicas. The purpose of the `runtimeClassName: rc-spin` field is to reference a RuntimeClass that ensures Kubernetes will schedule all three replicas to a node that meets the requirements specified in the RuntimeClass - specifically nodes with the required label. In the example provided, Kubernetes schedules all three replicas to the agent-1 node, which means the label and RuntimeClass worked as expected.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wasm-spin
spec:
  replicas: 3
  <Snip>
  template:
    metadata:
      labels:
        app: wasm
    <Snip>
    spec:
      runtimeClassName: rc-spin                   <<---- Referencing the RuntimeClass
      containers:
        - name: testwasm
          image: nigelpoulton/k8sbook:wasm-0.1    <<---- Pre-created image
          command: ["/"]
```

