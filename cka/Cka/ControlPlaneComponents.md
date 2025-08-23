# Control Plane Components

## Content

### ❓ Under what conditions does a Kubernetes cluster run a cloud controller manager, what cloud services does it integrate with, and provide a specific example of how it provisions infrastructure?
If your cluster is on a public cloud, such as AWS, Azure, GCP, or Civo Cloud, it will run a cloud controller manager. The cloud controller manager integrates the cluster with cloud services, such as: Instances, Load balancers, and Storage. For example, if you're on a cloud and an application requests a load balancer, the cloud controller manager provisions one of the cloud's load balancers and connects it to your app.

### ❓ What are the three main components of the Kubernetes control plane, what high-availability recommendations are provided, and what does Figure 2.5 show?
The control plane includes: The API Server, The scheduler, and The cluster store. You should run three or five control plane nodes for high availability. Large busy clusters might run a separate etcd cluster for better cluster store performance. Figure 2.5 shows a high-level view of a Kubernetes control plane node.

![Figure 2.5 - Control plane node](media/figure2-5.png)

