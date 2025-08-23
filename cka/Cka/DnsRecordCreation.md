# DNS Record Creation

## Content

### ❓ What specific DNS records does the cluster DNS register when it observes a new Service?
The cluster DNS observes the new Service and registers the appropriate DNS A and SRV records.

### ❓ What is the role of the headless Service in registering StatefulSet Pods and their IPs?
The headless Service is responsible for registering StatefulSet Pods and their IPs against the DNS name. When you deploy StatefulSet Pods (such as tkb-sts-0, tkb-sts-1, and tkb-sts-2) governed by a headless Service, the Service ensures that these Pods have predictable and reliable fully qualified DNS names that are registered in the cluster's DNS system.

