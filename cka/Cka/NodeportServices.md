# NodePort Services

## Content

### ❓ What specific issue might occur with Docker Desktop clusters regarding EXTERNAL-IP addresses, and what workaround is required to connect to the service?
Some Docker Desktop clusters incorrectly display an IP address in the EXTERNAL-IP column of service listings. When this occurs, you need to substitute the displayed IP address with 'localhost' and connect to localhost:8080 instead of using the IP address shown in the EXTERNAL-IP column.

### ❓ Explain how NodePort Services extend ClusterIP Services to enable external access and what the 'NodePort' specifically refers to, and analyze the following YAML configuration and explain the purpose of each annotated component in this NodePort Service definition.
NodePort Services extend ClusterIP Services by adding a dedicated port on every cluster node that external clients can use. We call this dedicated port the "NodePort" - it's a specific port number that gets opened on every node in the cluster for external access.

Here's the NodePort Service YAML configuration with explanations:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: skippy           <<---- Registered with the internal cluster DNS (ClusterIP)
spec:
  type: NodePort         <<---- Service type
  ports:
  - port: 8080           <<---- Internal ClusterIP port
    targetPort: 9000     <<---- Application port in container
    nodePort: 30050      <<---- External port on every cluster node (NodePort)
  selector:
    app: hello-world
```

- **name: skippy** - The Service name that gets registered with the internal cluster DNS
- **type: NodePort** - Specifies this as a NodePort Service type
- **port: 8080** - The port that the ClusterIP Service listens on internally
- **targetPort: 9000** - The actual port where the application is running inside the container
- **nodePort: 30050** - The external port that gets opened on every cluster node for external access
- **selector: app: hello-world** - Selects Pods with the label "app: hello-world" to include in the Service

### ❓ What happens when the NodePort Service YAML is posted to Kubernetes, and how does this enable external client access, and describe the four-step traffic flow process shown in Figure 7.3 for a NodePort Service handling an external client request?
When the NodePort Service YAML is posted to Kubernetes:

1. It creates a ClusterIP Service with the usual internally routable IP and DNS name
2. It publishes the specified NodePort (e.g., 30050) on every cluster node and maps it back to the ClusterIP
3. This enables external clients to send traffic to any cluster node on the NodePort and reach the Service and its Pods

The four-step traffic flow process for a NodePort Service is:

1. An external client hits a node on the NodePort
2. The node forwards the request to the ClusterIP of the Service inside the cluster
3. The Service picks a Pod from the EndpointSlice's always-up-to-date list
4. The Service forwards the request to the chosen Pod

### ❓ What does Figure 7.3 illustrate about how NodePort Services handle external client requests and distribute traffic to backend Pods, and how does the Service's load balancing mechanism work with NodePort Services for distributing requests across multiple Pods?
![Figure 7.3 - NodePort Service](media/figure7-3.png)

Figure 7.3 illustrates a NodePort Service exposing three Pods on every cluster node. The external client can send the request to any cluster node, and the Service can send the request to any of the three healthy Pods. The Service performs simple round-robin load balancing, meaning future requests will probably go to other Pods to distribute the load evenly across all available backend Pods.

### ❓ What are the two significant limitations of NodePort Services that make LoadBalancer Services more preferable for external access?
NodePort Services have two significant limitations:

1. They use high-numbered ports between 30000-32767
2. Clients need to know the names or IPs of nodes, as well as whether nodes are healthy

This is why most people use LoadBalancer Services instead, as they provide a simpler and more reliable external access method.

