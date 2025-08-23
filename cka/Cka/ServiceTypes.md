# Service Types

## Content

### ❓ What are the three main types of Kubernetes Services and their primary use cases, and explain the hierarchical relationship between ClusterIP, NodePort, and LoadBalancer Services and how each builds upon the previous type?
Kubernetes has three main types of Services for different use cases and requirements:

1. **ClusterIP** - The most basic Service type that provides a reliable endpoint (name, IP, and port) on the internal Pod network. It's the default Service type and is only accessible from inside the cluster.

2. **NodePort** - Builds on top of ClusterIP Services and allows external clients to connect via a dedicated port (called the "NodePort") on every cluster node.

3. **LoadBalancer** - Builds on top of both ClusterIP and NodePort Services and integrates with cloud load balancers for extremely simple access from the internet.

The hierarchical relationship shows that NodePort Services build on top of ClusterIP Services by adding external access capabilities, and LoadBalancer Services build on top of both by adding cloud load balancer integration for simplified external access.

### ❓ What are the three main types of Kubernetes Services and their primary use cases?
The three main types of Kubernetes Services are:

1. **ClusterIP:** The default Service type that provides a reliable endpoint (name, IP, and port) on the internal Pod network for accessing apps from inside the cluster

2. **NodePort:** Builds on top of ClusterIP and allows external clients to connect via a dedicated port on every cluster node

3. **LoadBalancer:** Builds on top of both ClusterIP and NodePort, integrating with cloud load balancers for extremely simple access from the internet

### ❓ Differentiate between ClusterIP Services and external endpoint Services (NodePort/LoadBalancer).
ClusterIP Services and external endpoint Services (NodePort/LoadBalancer) differ in their accessibility and use cases:

**ClusterIP Services:**
- Are the default Service type
- Provide reliable endpoints on the internal cluster network only
- Accessible only from within the cluster
- Used for internal service-to-service communication

**NodePort and LoadBalancer Services:**
- Provide external endpoints for access from outside the cluster
- NodePort exposes the service on a high port number on each node
- LoadBalancer creates a cloud load balancer with public DNS/IP
- Used for exposing services to external users or systems
- Build on top of ClusterIP Services (LoadBalancer → NodePort → ClusterIP)

### ❓ Explain the hierarchical relationship between LoadBalancer, NodePort, and ClusterIP Services as illustrated in Figure 7.5 and what Figure 7.5 illustrates about how different Kubernetes Service types build upon each other.
Second up, even though it's a LoadBalancer Service, it also created all the ClusterIP and NodePort constructs. This is because LoadBalancer Services build on top of NodePort Services, which, in turn, build on top of ClusterIP Services, as shown in Figure 7.5.

![Figure 7.5 - Service stacking](media/figure7-5.png)

### ❓ Compare and contrast the three main Service types: ClusterIP, NodePort, and LoadBalancer, including their primary use cases and network exposure characteristics.
The three main Service types in Kubernetes serve different purposes with distinct network exposure characteristics:

**ClusterIP Services**:
- Provide reliable networking on the internal Kubernetes network
- Only accessible within the cluster
- Primary use case: Internal service-to-service communication

**NodePort Services**:
- Expose a port on every cluster node
- Accessible from outside the cluster via any node's IP address and the assigned port
- Primary use case: External access to services when no cloud load balancer is available

**LoadBalancer Services**:
- Integrate with cloud platforms to create highly available internet-facing load balancers
- Automatically provision external load balancers in cloud environments
- Primary use case: Production-grade external access with cloud integration and high availability

Each service type builds upon the previous one, with LoadBalancer Services incorporating NodePort functionality, and both building on ClusterIP foundation.

### ❓ What fundamental networking difference exists between regular Kubernetes Services and the Services used by StatefulSets, and how do headless Services enable direct Pod connections?
Instead of using regular Kubernetes Services that load-balance requests across a set of Pods, StatefulSets use a special kind of Service called a headless Service. These create predictable DNS names for each StatefulSet Pod so that apps can query DNS (the service registry) for the full list of Pods and then connect directly to specific Pods.

### ❓ What is the purpose of the 'serviceName' field in a StatefulSet specification, which Service does it reference, and how does a headless Service differ from a regular Kubernetes Service in terms of ClusterIP configuration?
The 'serviceName' field in a StatefulSet specification references the governing Service for the StatefulSet. This is the headless Service that creates DNS SRV and DNS A records for every Pod matching the Service's label selector. The headless Service is listed in the StatefulSet as the governing Service.

A headless Service differs from a regular Kubernetes Service in that it has no ClusterIP address. A regular Service has a stable ClusterIP address (the "head") that forwards traffic to Pods (the "tail"), while a headless Service is specifically configured with `clusterIP: None` to indicate it has no ClusterIP address.

The primary purpose of headless Services is to create DNS SRV records for StatefulSet Pods, allowing clients to query DNS for individual Pods and send queries directly to those Pods instead of via a Service's ClusterIP.

### ❓ Explain what the following full YAML configuration does, including the purpose of both the Service and StatefulSet components and their relationship.
```yaml
apiVersion: v1
kind: Service                   <<---- Service
metadata:
  name: mongo-prod
spec:
  clusterIP: None               <<---- Make it a headless Service
  selector:
    app: mongo
    env: prod
---
apiVersion: apps/v1
kind: StatefulSet               <<---- StatefulSet
metadata:
  name: sts-mongo
spec:
  serviceName: mongo-prod       <<---- Governing Service
```

This YAML configuration defines two Kubernetes objects:

1. **Service**: A headless Service named "mongo-prod" that has `clusterIP: None`, making it a headless Service. It selects Pods with labels `app: mongo` and `env: prod`.

2. **StatefulSet**: A StatefulSet named "sts-mongo" that references the "mongo-prod" Service as its governing Service through the `serviceName: mongo-prod` field.

The relationship between these components is that the headless Service becomes the governing Service for the StatefulSet. When combined, the Service creates DNS SRV and DNS A records for every Pod matching the Service's label selector, allowing other Pods and applications to query DNS to get the names and IPs of all the StatefulSet's Pods.

