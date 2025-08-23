# Controller Architecture

## Content

### ❓ What is the relationship between the controller manager and individual controllers, and what does Figure 2.4 illustrate about them?
In cloud-managed Kubernetes, K8s also runs a controller manager that is responsible for spawning and managing the individual controllers. Figure 2.4 gives a high-level overview of the controller manager and controllers.

![Figure 2.4. Controller manager and controllers](media/figure2-4.png)

### ❓ What are the key benefits that Pods deployed via workload resources receive from being managed by highly available controllers, including their specific capabilities for node recovery, scaling, advanced operations, and the difference between local kubelet and controller restart capabilities?
Pods deployed via workload resources receive significant benefits from being managed by highly available controllers. These controllers provide capabilities including:

- **Restarting Pods on other nodes** when nodes fail or get evicted
- **Scaling Pods when demand changes** to handle varying workloads
- **Performing advanced operations** such as rolling updates and versioned rollbacks

While the local kubelet can attempt to restart failed containers on the same node, if the node itself fails or gets evicted, the controller can restart the Pod on a different node. This provides higher availability and fault tolerance compared to static Pods that are limited to a single node.

It's important to understand that when Kubernetes documentation says "restart the Pod," it actually means replacing it with a new one rather than restarting the existing Pod in place.

