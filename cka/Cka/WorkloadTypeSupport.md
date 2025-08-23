# Workload Type Support

## Content

### ❓ What does the diagram in Figure 4.1 illustrate about how Kubernetes handles different workload types through Pod abstraction, and how do Pods enable different workload types like containers, VMs, serverless functions, and Wasm apps to run on the same Kubernetes cluster?
Figure 4.1 illustrates how Kubernetes uses Pods as an abstraction layer to run different workload types on the same cluster. Pods abstract the workload details, meaning you can run containers, VMs, serverless functions, and Wasm apps inside Pods and Kubernetes doesn't know the difference. Each workload is wrapped in a Pod, managed by a controller, and uses a standard runtime.

![Figure 4.1 - Different workloads wrapped in Pods](media/figure4-1.png)

This abstraction benefits both Kubernetes and workloads:
- Kubernetes can focus on deploying and managing Pods without having to care what's inside them
- Heterogenous workloads can run side-by-side on the same cluster, leverage the full power of the declarative Kubernetes API, and get all the other benefits of Pods
- Containers and Wasm apps work with standard Pods, standard workload controllers, and standard runtimes

However, serverless functions and VMs need a bit of extra help:
- Serverless functions run in standard Pods but require apps like Knative to extend the Kubernetes API with custom resources and controllers
- VMs are similar and need apps like KubeVirt to extend the API
- VM workloads run in a VirtualMachineInstance (VMI) instead of a Pod, but these are very similar to Pods and utilize a lot of Pod features

### ❓ What are the eight ways that Pods augment workloads according to the listed features, and why do serverless functions and VMs require additional tools like Knative and KubeVirt to work with standard Pods?
Pods augment workloads in eight main ways:
1. **Resource sharing**
2. **Advanced scheduling**
3. **Application health probes**
4. **Restart policies**
5. **Security policies**
6. **Termination control**
7. **Volumes**
8. Many other features (as shown by the over 1,000 lines of Pod attributes returned by `kubectl explain pods --recursive`)

Serverless functions and VMs require additional tools like Knative and KubeVirt to work with standard Pods because:

**Serverless functions** run in standard Pods but require apps like Knative to extend the Kubernetes API with custom resources and controllers. **VMs** are similar and need apps like KubeVirt to extend the API. 

This is necessary because VMs are fundamentally different from containers - they are designed to be mutable and immortal (you can restart them, change their configurations, and even migrate them), which is very different from the design goals of Pods. This is why KubeVirt wraps VMs in a modified Pod called a VirtualMachineInstance (VMI) and manages them using custom workload controllers.

While containers and Wasm apps work with standard Pods, standard workload controllers, and standard runtimes, serverless functions and VMs need this extra layer of abstraction to bridge the gap between their operational requirements and Kubernetes' Pod-based execution model.

### ❓ What are the four types of applications that Kubernetes Pods can abstract and execute?
The apps can be containers, serverless functions, Wasm apps, and VMs.

### ❓ Explain how Wasm containers differ from traditional containers in terms of packaging, scheduling, and execution security model.
Wasm containers package Wasm (WebAssembly) apps in OCI containers that can execute on Kubernetes. These apps only use containers for packaging and scheduling, but at run time they execute inside a secure deny-by-default Wasm host, which provides a different security model than traditional container execution.

