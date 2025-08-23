# Core Capabilities

## Content

### ❓ What are the three key features that StatefulSets provide for stateful applications that Deployments do not?
StatefulSets offer three critical features that Deployments do not provide:

1. **Predictable and persistent Pod names and DNS names** - StatefulSet Pods get predictable names following the format `<statefulset-name>-<ordinal-index>` (e.g., tkb-sts-0, tkb-sts-1, tkb-sts-2) and maintain consistent DNS hostnames throughout their lifecycle.

2. **Predictable and persistent volume bindings** - Each StatefulSet Pod gets its own persistent volumes that remain bound to that specific Pod even across failures, rescheduling, or scaling operations.

3. **Predictable startup and shutdown order** - StatefulSets create and terminate Pods in a specific order (by ordinal index), ensuring controlled startup and graceful shutdown sequences.

Points one and two form a Pod's state, often referred to as a Pod's "sticky ID." These features persist across failures, scaling operations, and other scheduling events, making StatefulSets ideal for applications requiring unique reliable Pods with persistent identities.

### ❓ How does the StatefulSet controller's architecture for self-healing and scaling differ from Deployments?
Finally, it's worth noting that the StatefulSet controller does its own self-healing and scaling. This is architecturally different from Deployments, which use the ReplicaSet controller for these operations.

### ❓ What is the primary function of the StatefulSet controller regarding cluster state management and describe the process when a specific Pod replica fails?
The StatefulSet controller observes the state of the cluster and reconciles observed state with desired state. If you have a StatefulSet called tkb-sts with five replicas and the tkb-sts-3 replica fails, the controller starts a new Pod with the same name and attaches it to the surviving volumes.

### ❓ Why were node failures more complex for StatefulSets in older Kubernetes versions and what fundamental challenge makes node failure detection difficult?
Node failures can be more complex, and some older versions of Kubernetes require manual intervention to replace Pods that were running on failed nodes. This is because it can sometimes be hard for Kubernetes to know if a node has genuinely failed or is just rebooting from a transient event.

### ❓ What specific data corruption risk occurs if a "failed" node recovers after Kubernetes has already replaced its Pods?
For example, if a "failed" node recovers after Kubernetes has replaced its Pods, you'll end up with identical Pods trying to write to the same volume. This can result in data corruption.

