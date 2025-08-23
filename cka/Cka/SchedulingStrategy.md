# Scheduling Strategy

## Content

### ❓ What is the primary function of the Vertical Pod Autoscaler (VPA) in Kubernetes?
The Vertical Pod Autoscaler (VPA) increases and decreases the CPU and memory allocated to running Pods to meet current demand.

### ❓ What are three key characteristics that differentiate the Vertical Pod Autoscaler from other scaling solutions?
The Vertical Pod Autoscaler isn't installed by default, has several known limitations, and is less widely used.

### ❓ How does the current implementation of Vertical Pod Autoscaling handle resource scaling, and what is a significant drawback of this approach?
Current implementations work by deleting the existing Pod and replacing it with a new one every time it scales the Pods resources. This is disruptive and can even result in Kubernetes scheduling the new Pod to a different node.

### ❓ What future improvement is being developed for Vertical Pod Autoscaling, and what is its current development status?
Work is underway to enable in-place updates to live Pods, but it's currently an early alpha feature.

### ❓ Describe the collaborative scaling process between the Horizontal Pod Autoscaler (HPA) and Cluster Autoscaler (CA) when demand increases beyond available node resources.
You deploy an application to your cluster and configure an HPA to autoscale the number of application Pods between two and ten. Demand increases, and the HPA asks the scheduler to increase the number of Pods from two to four. This works, but demand continues rising, and the HPA asks the scheduler for another two Pods. However, the scheduler can't find a node with sufficient resources and marks the two new Pods as pending. The CA notices the pending Pods and dynamically adds a new cluster node. Once the node joins the cluster, the scheduler assigns the pending Pods to it.

### ❓ What process occurs when Kubernetes removes a cluster node during scaling down operations?
When removing a cluster node, Kubernetes evicts all Pods on the node and replaces them with new Pods on surviving nodes.

### ❓ What does the term 'multi-dimensional autoscaling' refer to in Kubernetes, and what two scaling dimensions does it combine?
This is jargon for combining multiple scaling methods --- scaling Pods and nodes, or scaling apps horizontally (adding more Pods) and vertically (adding more resources to existing Pods).

