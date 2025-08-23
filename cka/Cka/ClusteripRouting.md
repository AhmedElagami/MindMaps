# ClusterIP Routing

## Content

### ❓ What is the fundamental problem with ClusterIP routing that requires 'additional magic' to reach the actual Pods and why do containers send ClusterIP traffic to their default gateway?
The fundamental problem with ClusterIP routing is that ClusterIPs are on a special network called the service network and there are no routes to it! This means containers cannot send traffic directly to ClusterIP addresses because the container's network stack doesn't know how to reach the service network.

Containers send ClusterIP traffic to their default gateway because a default gateway is where a system sends network traffic when it doesn't know where else to send it. Since the container has no route to the service network where ClusterIPs reside, it follows standard networking behavior and sends the traffic to its default gateway, hoping that the gateway will know how to handle it.

