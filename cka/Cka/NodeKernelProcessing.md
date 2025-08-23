# Node Kernel Processing

## Content

### ❓ How does the node's kernel ultimately ensure that ClusterIP traffic reaches healthy Pods matching the Service's label selector and what is the critical processing step?
The node's kernel ensures that ClusterIP traffic reaches healthy Pods through the rules created by kube-proxy. Every time a node's kernel processes traffic for a ClusterIP, it redirects it to the IP of a healthy Pod matching the Service's label selector.

The critical processing step occurs when the traffic reaches the node's kernel. Since the node doesn't have a route to the service network either, it sends the traffic to its own default gateway. This causes the node's kernel to process the traffic, and this is where the magic happens - the kernel intercepts the ClusterIP traffic using the rules that kube-proxy installed and redirects it to the actual Pod IP address.

This redirection is the fundamental mechanism that bridges the gap between the virtual ClusterIP address and the actual Pod endpoints, ensuring that service traffic eventually reaches the intended destination Pods.

### ❓ How does the node's kernel handle traffic destined for a service's ClusterIP, and what mechanism causes the redirection to an actual application pod?
The node's kernel handles traffic destined for a service's ClusterIP through a trapping mechanism. En route to the node's default gateway, the traffic caused a trap in the node's kernel, resulting in the kernel redirecting it to a Pod hosting the app. This mechanism ensures that service traffic is properly load-balanced to the actual application pods backing the service.

