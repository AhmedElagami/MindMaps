# Registration Process

## Content

### ❓ What is the most important characteristic of service registration on Kubernetes according to the paragraph?
The most important thing to know about service registration on Kubernetes is that it's automatic!

### ❓ What three steps are involved in the service registration process and what are the respective responsibilities of developers and Kubernetes?
There are three steps in service registration:

1. Give the Service a name (developers are responsible for this)
2. Assign the Service an IP (Kubernetes handles this)
3. Register the name and IP with the cluster DNS (Kubernetes handles this)

Developers are responsible for point one (naming the Service), while Kubernetes handles points two and three (IP assignment and DNS registration).

### ❓ What does the diagram in Figure 10.4 illustrate about the Kubernetes service registration flow?
![Figure 10.4 - Service registration flow](media/figure10-4.png)

Figure 10.4 summarises the service registration process and adds some of the details from Chapter 7. It shows the flow where you deploy a new app defined in a YAML file describing a Deployment and a Service, post it to Kubernetes where it's authenticated and authorized, Kubernetes allocates a ClusterIP to the Service and persists its configuration to the cluster store, the cluster DNS observes the new Service and registers the appropriate DNS A and SRV records, associated EndpointSlice objects are created, and kube-proxy creates local routing rules.

### ❓ How does Kubernetes implement automatic service registration and discovery? What are the three main components of Kubernetes' built-in service registry and their functions? Where are all Kubernetes DNS components located and how are they labeled for identification?
Kubernetes makes service registration and service discovery automatic using the cluster DNS as its built-in service registry. This runs as one or more managed Pods with a Service object providing a stable endpoint.

The important components are:

- **Pods**: Managed by the coredns Deployment
- **Service**: A ClusterIP Service called kube-dns listening on port 53 TCP/UDP
- **EndpointSlice objects**: Names pre-fixed with kube-dns

All of these objects are in the kube-system Namespace and tagged with the k8s-app=kube-dns label to help you find them.

