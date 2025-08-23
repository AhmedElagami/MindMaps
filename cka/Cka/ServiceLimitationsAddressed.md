# Service Limitations Addressed

## Content

### ❓ What are the two main limitations of NodePort and LoadBalancer Services that Ingress addresses?
NodePort Services only work on high port numbers, and clients need to keep track of node IP addresses. LoadBalancer Services fix this but only provide a one-to-one mapping between internal Services and cloud load balancers. This means a cluster with 25 internet-facing apps will need 25 cloud load balancers, and cloud load balancers cost money! Your cloud may also limit the number of load balancers you can create.

