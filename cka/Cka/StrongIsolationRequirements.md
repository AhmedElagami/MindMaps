# Strong Isolation Requirements

## Content

### ❓ Why is using Namespaces to divide a cluster among external tenants not as common, what limitation do they have, and what is the most common method for achieving strong tenant isolation at the time of writing?
Using Namespaces to divide a cluster among external tenants isn't as common because they only provide soft isolation and cannot prevent compromised workloads from escaping the Namespace and impacting workloads in other Namespaces. At the time of writing, the most common way of strongly isolating tenants is to run them on their own clusters and their own hardware.

In summary:
- Namespaces are lightweight and easy to manage but only provide soft isolation
- Running multiple clusters costs more and introduces more management overhead, but it offers strong isolation

![Soft vs Hard Isolation Diagram](media/figure5-1.png)

### ❓ Why are Namespaces not suitable for hard multi-tenancy despite providing some isolation capabilities?
Namespaces are not suitable for hard multi-tenancy because they're not a strong workload isolation boundary. While they provide some level of separation for resource management and policy application, they lack the robust security and isolation features required for true hard multi-tenancy scenarios.

