# Container vs VM Comparison

## Content

### ❓ How does containerization differ from traditional virtual machines according to the comparison provided?
Containerization differs from traditional virtual machines in several ways. It can be useful to think of containers as the next generation of virtual machines (VM). Both are ways of packaging and running applications, but containers are smaller, faster, and more portable. Despite these advantages, containers haven't entirely replaced VMs, and you'll see them running side-by-side in most environments. However, containers are the first-choice solution for most new applications.

### ❓ How do Virtual Machines (VMs) fundamentally differ from Pods in terms of mutability and lifecycle management, and how does KubeVirt address these differences?
VMs are the opposite of containers, in that they are designed to be mutable and immortal. For example, you can restart them, change their configurations, and even migrate them. This is very different from the design goals of Pods and is why KubeVirt wraps VMs in a modified Pod called a VirtualMachineInstance (VMI) and manages them using custom workload controllers.

### ❓ What does Figure 9.1 illustrate about the evolution of cloud computing waves, and what are the three waves of cloud computing with their key advantages according to the chapter introduction?
Figure 9.1 illustrates the evolution of cloud computing waves showing how each subsequent wave enables smaller, faster, and more portable applications that can go places and do things the previous waves couldn't.

![Figure 9.1](media/figure9-1.png)

The three waves of cloud computing are:
1. **Virtual Machines** - The first wave
2. **Containers** - The second wave  
3. **Wasm (WebAssembly)** - The third wave

Each subsequent wave enables smaller, faster, and more portable applications that can go places and do things the previous waves couldn't.

### ❓ What are the compatibility considerations when implementing user namespaces in Kubernetes?
However, you should check if it is fully-supported by your version of Kubernetes and your container runtime.

### ❓ What fundamental architectural difference exists between containers and virtual machines regarding workload isolation, and what are the main trade-offs between performance and security when choosing between them?
The fundamental architectural difference is that "most container platforms implement namespaced containers" where "every container shares the host's kernel, and isolation is provided by kernel constructs, such as namespaces and cgroups, that were never designed as strong security boundaries." In contrast, "every virtual machine gets its own dedicated kernel and is strongly isolated from other virtual machines using hardware enforcement."

The main trade-offs are:
- **Virtual Machines**: "Every workload gets its own dedicated kernel. It provides excellent isolation but is comparatively slow and resource-intensive."
- **Namespaced containers**: "All containers share the host's kernel. These are fast and lightweight but require extra effort to improve workload isolation."

This means virtual machines provide stronger security isolation but at the cost of performance and resource efficiency, while containers offer better performance but weaker inherent isolation that requires additional security measures.

### ❓ Compare and contrast the isolation characteristics of Virtual Machines versus namespaced containers in terms of kernel usage, performance, and security requirements.
Virtual Machines provide excellent isolation as every workload gets its own dedicated kernel, but they are comparatively slow and resource-intensive. Namespaced containers are fast and lightweight as all containers share the host's kernel, but they require extra effort to improve workload isolation.

### ❓ What are the advantages and disadvantages of running every container in its own virtual machine, and why are these solutions not very popular?
Running every container in its own virtual machine attempts to combine the versatility of containers with the security of VMs by running every container in its own dedicated VM. The advantage is improved security through VM-level isolation. However, despite using specialized lightweight VMs, these solutions lose much of the appeal of containers (lightweight nature and fast startup times), and they're not very popular due to this trade-off.

