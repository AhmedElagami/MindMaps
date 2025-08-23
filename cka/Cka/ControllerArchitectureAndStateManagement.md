# Controller Architecture and State Management

## Controller Architecture

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


## State Management
## Content

### ❓ Explain the concept of controllers 'reconciling observed state with desired state' through background watch loops and provide a specific example of how a controller ensures application requirements are met.
They all run as background watch loops, reconciling observed state with desired state. This means controllers ensure the cluster runs what you asked it to run. For example, if you ask for three replicas of an app, a controller will ensure you have three healthy replicas and take appropriate actions if you don't.

### ❓ What are the three fundamental principles that form the core of Kubernetes' declarative model, and define desired state, observed state, and reconciliation in this context?
They work on three basic principles:
- Desired state
- Observed state
- Reconciliation

Desired state is what you want. Observed state is what you have. Reconciliation is the process of keeping observed state in sync with desired state.

### ❓ Describe the complete workflow of the Kubernetes declarative model from YAML creation to continuous reconciliation.
In Kubernetes, the declarative model works like this:
- You describe the desired state of an application in a YAML manifest file
- You post the YAML file to the API server
- Kubernetes records this in the cluster store as a record of intent
- A controller notices the observed state of the cluster doesn't match the new desired state
- The controller makes the necessary changes to reconcile the differences
- The controller keeps running in the background, ensuring observed state always matches desired state

### ❓ What happens to the YAML configuration after it's authenticated and authorized by the API server?
Once authenticated and authorized, Kubernetes persists the configuration to the cluster store as a record of intent.

### ❓ What triggers the reconciliation process in Kubernetes and what types of changes does it typically involve, including the specific actions that controllers perform?
The reconciliation process is triggered when the observed state of the cluster doesn't match the desired state. A controller will notice this difference and begin reconciliation, which typically involves making all the changes described in the YAML file including:
- scheduling new Pods
- pulling images
- starting containers
- attaching them to networks
- starting application processes

### ❓ What happens once reconciliation completes and how do controllers continue to operate?
Once the reconciliation completes, observed state will match desired state, and everything will be OK. However, the controllers keep running in the background, ready to reconcile any future differences.

### ❓ What are the three fundamental state concepts that are vital to understand in Kubernetes deployment theory?
The three fundamental state concepts are: Desired state, Observed state (sometimes called actual state or current state), and Reconciliation.

### ❓ Define desired state and observed state in Kubernetes, and explain the relationship between them.
Desired state is what you want, observed state is what you have, and the goal is for them to always match.

