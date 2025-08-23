# Kube-proxy

## Functions

- Implements cluster networking for Services.
- Load balances traffic to Pods on the node.

## Operations

- Watches the API server for Service and EndpointSlice changes.
- Programs kernel iptables or IPVS rules to direct ClusterIP traffic to Pod IPs.
- IPVS mode offers higher performance; iptables is the traditional default.
