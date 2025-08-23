# Soft Isolation Benefits

## Content

### ❓ Describe the concept of 'soft isolation' and 'soft multi-tenancy' as it relates to Kubernetes Namespaces, including both benefits and limitations.
Namespaces are a form of soft isolation and enable soft multi-tenancy. For example, you can create Namespaces for your dev, test, and qa environments and apply different quotas and policies to each. However, a compromised workload in one Namespace can easily impact workloads in other Namespaces.

