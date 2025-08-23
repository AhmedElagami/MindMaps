# DNS Client Configuration

## Content

### ❓ How does Kubernetes ensure that every container can perform service discovery according to the paragraph?
Kubernetes automatically configures every container to use the cluster DNS IP for service discovery.

### ❓ How does Kubernetes enable containers to convert service names like 'cer' into IP addresses and what two methods does it use to configure containers for service discovery?
Kubernetes enables containers to convert service names into IP addresses by configuring every container to use the cluster DNS for service discovery. It does this through two methods:

1. Automatically configuring every container's resolv.conf file with the IP address of the cluster DNS Service
2. Adding search domains to append to unqualified names (short names like "cer")

This means containers automatically send DNS queries to the cluster DNS to resolve service names to ClusterIP addresses.

### ❓ What role does the kubelet play in service discovery configuration?
The kubelet on each node also configures all containers to use the cluster DNS for service discovery.

