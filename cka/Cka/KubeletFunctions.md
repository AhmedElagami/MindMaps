# Kubelet Functions

## Content

### ❓ What are the three key tasks performed by the kubelet as the main Kubernetes agent on worker nodes, and how does it handle communication and problem reporting when tasks cannot be executed?
The kubelet performs the following key tasks: Watches the API server for new tasks, Instructs the appropriate runtime to execute tasks, and Reports task status to the API server. If a task won't run, the kubelet reports the problem to the API server and lets the control plane decide what actions to take.

### ❓ Describe the complete process from scheduler assignment to runtime execution, including how the kubelet handles resource requests and limits.
Assuming the scheduler finds a suitable node, it assigns the Pod to the node, and the kubelet downloads the Pod spec and asks the local runtime to start it. As part of the process, the kubelet reserves the requested CPU and memory, guaranteeing the resources will be there when needed. It also tells the runtime to set a resource cap based on each container's resource limits. In the example, it asks the runtime to set a cap of one CPU and 512Mi of memory. Most runtimes will enforce these limits, but how each runtime implements this can vary.

