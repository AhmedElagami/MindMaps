# Imperative Model

## Content

### ❓ Explain what the following imperative command does to create a new Namespace called hydra.
The following imperative command creates a new Namespace called "hydra":

```bash
$ kubectl create ns hydra
namespace/hydra created
```

This command uses the `kubectl create ns` syntax to immediately create the Namespace resource in the Kubernetes cluster, and the output confirms that the namespace/hydra was successfully created.

### ❓ What does the following command do to delete the hydra Namespace?
The following command deletes the hydra Namespace:

```bash
$ kubectl delete ns hydra
namespace "hydra" deleted
```

This imperative command removes the hydra Namespace from the Kubernetes cluster, and the output confirms that the namespace "hydra" was successfully deleted.

### ❓ What two cleanup actions are required when deleting a Namespace to ensure proper cluster management?
The two cleanup actions required when deleting a Namespace are:

1. Delete the Namespace itself using `kubectl delete ns <namespace-name>`
2. Reset your kubeconfig to use the default Namespace using `kubectl config set-context --current --namespace default`

These actions ensure that both the cluster resources are cleaned up and future kubectl commands target the appropriate Namespace.

### ❓ What happens to objects deployed within a Namespace when you delete the Namespace itself, and what should you expect regarding command completion time?
When you delete a Namespace, it automatically deletes all objects deployed within it, including Pods, Services, and ServiceAccounts. The deletion process may take a few seconds to complete, so you should expect the command to not complete immediately but rather take some time to finish cleaning up all associated resources.

### ❓ Explain the exact command and expected output when deleting a Kubernetes Namespace called 'shield'.
The exact command to delete the 'shield' Namespace is:

```bash
$ kubectl delete ns shield
namespace "shield" deleted
```

The expected output confirms that the Namespace has been successfully deleted with the message "namespace \"shield\" deleted".

### ❓ What specific problems arise from using imperative instructions in Kubernetes, particularly regarding container runtime variations?
For example, the commands to pull and start containerd containers are different from the commands to pull and start CRI-O containers. This results in more work and is prone to more errors, and because it's not declaring a desired state, there's no self-healing.

