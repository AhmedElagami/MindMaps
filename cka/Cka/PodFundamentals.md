# Pod Fundamentals

## Content

### ❓ What are Pods? What types of applications can Kubernetes run, what requirement do they all share, and what is the fundamental purpose of Pods in Kubernetes?
Kubernetes runs containers, VMs, Wasm apps, and more. All of them need wrapping in Pods before they'll run on Kubernetes. Pods serve as a thin wrapper that abstracts different types of tasks so they can run on Kubernetes.

### ❓ What complex logistics does Kubernetes handle once an application is wrapped in a Pod?
This includes the complex logistics of:
- Choosing appropriate nodes
- Joining networks
- Attaching volumes
- And more

### ❓ What is the atomic unit of scheduling in Kubernetes and how does it compare to VMware's approach, and what types of workloads does Kubernetes run with what requirements for all of them?
The atomic unit of scheduling in VMware is the virtual machine (VM). In Kubernetes, it's the Pod. Kubernetes runs containers, VMs, Wasm apps, and more. But they all need wrapping in Pods.

### ❓ Why does Kubernetes always schedule all containers in a Pod to the same node and what does it mean that starting a Pod is an atomic operation in terms of readiness?
Kubernetes always schedules containers in the same Pod to a single node because:

- Kubernetes schedules Pods, not individual containers
- Pods are a shared execution environment, and you can't easily share memory, networking, and volumes across different nodes

### ❓ Why can't Pods easily share memory, networking, and volumes across different nodes?
Pods are a shared execution environment, and you can't easily share memory, networking, and volumes across different nodes.

### ❓ Why are Pods considered the minimum unit of scheduling in Kubernetes and how does Kubernetes handle application scaling in relation to Pods?
Pods are the minimum unit of scheduling in Kubernetes. As such: scaling an application up adds more Pods, scaling it down deletes Pods. You do not scale by adding more containers to existing Pods.

### ❓ What does the diagram in Figure 2.10 illustrate about scaling microservices using Pods as the unit of scaling?
Figure 2.10 shows how to scale the web-fe microservice using Pods as the unit of scaling.

![Figure 2.10 - Scaling with Pods](media/figure2-10.png)

### ❓ What are the fundamental limitations of static Pods compared to controller-managed Pods in terms of self-healing, scaling, and update capabilities, and how do workload controllers provide advantages over static Pod management?
Deploying directly from a Pod manifest creates a static Pod that cannot self-heal, scale, or perform rolling updates. This is because they're only managed by the kubelet on the node they're running on, and kubelets are limited to restarting containers on the same node. Also, if the node fails, the kubelet fails with it and cannot do anything to help the Pod.

On the flip side, Pods deployed via workload resources get all the benefits of being managed by a highly available controller that can restart them on other nodes, scale them when demand changes, and perform advanced operations such as rolling updates and versioned rollbacks. The local kubelet can still attempt to restart failed containers, but if the node fails or gets evicted, the controller can restart it on a different node.

### ❓ What is the fundamental unit in Kubernetes where all applications are deployed?
Kubernetes deploys all applications inside Pods.

### ❓ What three core capabilities do Pods provide beyond application abstraction?
As well as abstracting different types of applications, Pods provide:
- a shared execution environment
- advanced scheduling
- application health probes
- and lots more

