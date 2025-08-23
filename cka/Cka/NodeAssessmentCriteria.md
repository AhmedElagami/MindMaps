# Node Assessment Criteria

## Content

### ❓ Explain how the scheduler ranks nodes and what factors contribute to this ranking system, and what happens when it cannot find a suitable node for a task?
It ranks the remaining ones according to factors such as: Whether it already has the required image, The amount of available CPU and memory, and Number of tasks it's currently running. Each is worth points. The nodes with the most points are selected to run the tasks. The scheduler marks tasks as pending if it can't find a suitable node. If the cluster is configured for node autoscaling, the pending task kicks off a cluster autoscaling event that adds a new node to the cluster. The scheduler assigns the task to the new node.

### ❓ How does a nodeSelector work to schedule Pods on specific nodes and what are the key differences between affinity and anti-affinity rules in Kubernetes scheduling?
nodeSelectors are the simplest way of running Pods on specific nodes. You give it a list of labels, and the scheduler will only assign the Pod to a node with all the labels.

Affinity and anti-affinity rules are like a more powerful nodeSelector. As the names suggest, they support scheduling alongside resources (affinity) and away from resources (anti-affinity).

### ❓ What capabilities do affinity rules provide beyond basic nodeSelectors and what is the fundamental difference between hard and soft scheduling rules?
Affinity rules provide capabilities beyond basic nodeSelectors by supporting hard and soft rules, and they can select on Pods as well as nodes.

Hard rules must be obeyed, while soft rules are only suggestions and best effort.

### ❓ How do Pod-based affinity rules differ from node-based affinity rules in their selection criteria and what would happen with a hard node affinity rule if no nodes match the specified label?
For Pod-based affinity rules: You provide a list of labels, and Kubernetes ensures the Pod will run on the same nodes as other Pods with those labels. For node-based rules: Selecting on nodes is common and works like a nodeSelector where you supply a list of labels, and the scheduler assigns the Pod to nodes with those labels.

With a hard node affinity rule: It won't schedule the Pod if it can't find a node with that label.

### ❓ How do topology spread constraints help with infrastructure management and why are resource requests and limits critical for proper Pod scheduling?
Topology spread constraints are a flexible way of intelligently spreading Pods across your infrastructure for availability, performance, locality, or any other requirements. A typical example is spreading Pods across your cloud or data center's underlying availability zones for high availability (HA).

Resource requests and limits are critical because they tell the scheduler how much CPU and memory a Pod needs, and the scheduler uses them to ensure they run on nodes with enough resources.

### ❓ What are the consequences of not specifying resource requirements for a Pod and what are the key steps involved in deploying a Pod to a Kubernetes cluster?
If you don't specify resource requirements, the scheduler cannot know what resources a Pod requires and may schedule it to a node with insufficient resources.

Deploying a Pod includes the following steps:
- Define the Pod in a YAML manifest file
- Post the manifest to the API server
- The request is authenticated and authorized
- The Pod spec is validated
- The scheduler filters nodes based on nodeSelectors, affinity and anti-affinity rules, topology spread constraints, resource requirements and limits, and more
- The Pod is assigned to a healthy node meeting all requirements
- The kubelet on the node watches the API server and notices the Pod assignment
- The kubelet downloads the Pod spec and asks the local runtime to start it
- The kubelet monitors the Pod status and reports status changes to the API server

### ❓ What criteria does the scheduler use to filter nodes during Pod deployment and what role does the kubelet play in the Pod deployment process?
The scheduler filters nodes based on nodeSelectors, affinity and anti-affinity rules, topology spread constraints, resource requirements and limits, and more.

The kubelet on the node watches the API server and notices the Pod assignment, downloads the Pod spec and asks the local runtime to start it, and monitors the Pod status and reports status changes to the API server.

### ❓ What Kubernetes technologies are mentioned for targeting workloads to specific nodes for isolation purposes?
Kubernetes offers several technologies for targeting workloads to specific nodes for isolation purposes, including:
- "labels"
- "affinity and anti-affinity rules"
- "taints"

These technologies help with node isolation for applications that "require non-standard privileges, such as running as root or executing non-standard syscalls" where isolating them on their own clusters might be overkill, but running them on a "ring-fenced subset of worker nodes" can restrict compromised workloads from impacting other workloads on the same node.

