# Isolation Concepts

## Content

### ❓ What does Figure 5.1 illustrate about the differences between soft multi-tenancy using Namespaces and strong isolation using separate clusters?
Figure 5.1 illustrates the differences between soft and hard isolation approaches:

- A cluster on the left using Namespaces for soft multi-tenancy
- All apps on this cluster share the same nodes and control plane
- Compromised workloads can impact both Namespaces
- The two clusters on the right provide strong isolation by implementing two separate clusters, each on dedicated hardware

![Soft vs Hard Isolation Diagram](media/figure5-1.png)

### ❓ What distinguishes development environments from production environments in terms of image usage policies?
As a general rule, development environments have fewer rules and are places where developers can experiment. This can involve non-standard images your developers eventually want to use in production. Production environments require more stringent security measures and only approved images.

### ❓ According to the text, what is the fundamental limitation of Kubernetes regarding secure multi-tenant clusters and what are the three main types of controls that can be applied through Kubernetes Namespaces?
The fundamental limitation is that "Kubernetes does not support secure multi-tenant clusters. The only way to isolate two workloads is to run them on their own clusters with their own hardware."

The three main types of controls that can be applied through Kubernetes Namespaces are:
1. Limits
2. Quotas
3. RBAC rules

Namespaces are described as "little more than a way of grouping resources and applying things such as" these controls.

### ❓ Why should Kubernetes Namespaces not be used as security boundaries according to the text?
Kubernetes Namespaces should not be used as security boundaries because "Namespaces do not prevent compromised workloads in one Namespace from impacting workloads in other Namespaces. This means you should never run hostile workloads on the same Kubernetes cluster." Despite this limitation, the text notes that "Kubernetes Namespaces are useful, and you should use them. Just don't use them as security boundaries."

### ❓ How does the text define 'soft multi-tenancy' and what characterizes the workloads in this scenario, and what is the key distinction between 'soft multi-tenancy' and 'hard multi-tenancy'?
The text defines "soft multi-tenancy" as "hosting multiple trusted workloads on shared infrastructure. By trusted, we mean workloads that don't require absolute guarantees that one workload cannot impact another." An example given is "an e-commerce application with a web front-end service and a back-end recommendation service" where both services are part of the same application and therefore not hostile.

The key distinction from "hard multi-tenancy" is that hard multi-tenancy involves "hosting untrusted and potentially hostile workloads on shared infrastructure," which "isn't currently possible with Kubernetes." Workloads requiring strong security boundaries need to run on separate Kubernetes clusters for hard multi-tenancy scenarios.

### ❓ What are three specific examples of workloads that require separate Kubernetes clusters for hard multi-tenancy?
Three specific examples of workloads that require separate Kubernetes clusters for hard multi-tenancy are:
1. "Isolating production and non-production workloads"
2. "Isolating different customers"
3. "Isolating sensitive projects and business functions"

The text emphasizes that "workloads requiring strong separation need their own clusters" and notes that the Kubernetes project has a dedicated Multitenancy Working Group actively working on multitenancy models for future releases.

