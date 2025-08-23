# Cloud-Native Features

## Content

### ❓ What does the term 'orchestrator' refer to in the context of Kubernetes, and what are its key capabilities including the four specific capabilities mentioned?
In the context of Kubernetes, an orchestrator is a system that deploys applications and dynamically responds to changes. The key capabilities of Kubernetes as an orchestrator include:

- Deploy applications
- Scale them up and down based on demand
- Self-heal them when things break
- Perform rollouts and rollbacks
- Lots more

The best part is that it does all of this without you having to get involved. You need to configure a few things up front, but once you've done that, you sit back and let Kubernetes work its magic.

### ❓ What specific cloud-native features distinguish cloud-native applications from regular applications running in the cloud?
Cloud-native applications possess cloud-like features that distinguish them from regular applications running in the cloud. These features include:

- Auto-scaling
- Self-healing
- Automated updates
- Rollbacks
- And more

Simply running a regular application in the public cloud does not make it cloud-native. Cloud-native applications are specifically designed with these cloud-native characteristics built-in.

### ❓ What specific aspect of cloud and data center management does Kubernetes perform similarly to traditional operating systems?
Kubernetes performs application scheduling similarly to traditional operating systems. Just as traditional operating systems schedule processes onto CPU cores, Kubernetes schedules applications onto the resources of clouds and data centers. At a high level, a cloud or data center is a complex collection of resources and services, and Kubernetes abstracts most of this, making the resources easier to consume.

### ❓ What are the three specific infrastructure components mentioned that developers typically don't need to worry about when using Kubernetes?
The three specific infrastructure components that developers typically don't need to worry about when using Kubernetes are: 1) which compute node, 2) which failure zone, and 3) which storage volume their app uses. Most of the time, developers are happy to let Kubernetes decide these infrastructure details.

### ❓ How do third-party tools like Knative and KubeVirt extend Kubernetes capabilities?
Third-party tools, such as Knative and KubeVirt, extend the Kubernetes API with:
- custom resources
- and custom workload controllers that allow Kubernetes to run serverless and VM workloads

### ❓ What are the four cloud-native features that Deployments add to stateless apps on Kubernetes?
Deployments add four key cloud-native features to stateless apps on Kubernetes:

1. **Self-healing**: If a Pod fails, the Deployment controller replaces it with a new one.
2. **Scaling**: If demand increases, the Deployment controller can scale the app by deploying more identical Pods.
3. **Rolling updates (rollouts)**: When you update the app, the Deployment controller deletes the old Pods and replaces them with new ones in a controlled manner.
4. **Rollbacks**: Kubernetes keeps old ReplicaSet configurations to easily roll back to earlier versions.

