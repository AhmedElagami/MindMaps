# kube-proxy Operations

## Content

### ❓ How does kube-proxy contribute to making Service ClusterIPs reachable from Pods?
The kube-proxy process on every cluster node observes the new objects and creates local routing rules (IPVS) so that requests to the Service's ClusterIP get routed to Pods.

### ❓ What is the role of kube-proxy in the ClusterIP routing process and how does it accomplish this?
kube-proxy plays a critical role in the ClusterIP routing process. Every Kubernetes node runs a system service called kube-proxy that implements a controller watching the API server for new Services and EndpointSlice objects. Whenever it sees new Services or EndpointSlices, it creates rules in the kernel to intercept ClusterIP traffic and forward it to individual Pod IPs.

kube-proxy accomplishes this by:
1. Continuously watching the Kubernetes API server for changes to Services and EndpointSlices
2. Creating and maintaining kernel-level rules (using iptables, IPVS, or other mechanisms depending on the mode)
3. Intercepting traffic destined for ClusterIP addresses
4. Redirecting that traffic to the IP addresses of healthy Pods that match the Service's label selector

This enables the "magic" that makes ClusterIPs work despite there being no direct routes to the service network.

### ❓ How does kube-proxy handle ClusterIP traffic since there are no routes to the service network?
There are no routes to this network, but the kube-proxy configures all cluster nodes to redirect ClusterIP traffic to Pod IPs on the Pod network.

