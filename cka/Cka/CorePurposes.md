# Core Purposes

## Content

### ❓ What is the fundamental purpose of Kubernetes Namespaces according to the introductory paragraph?
Namespaces are a way of dividing a Kubernetes cluster into multiple virtual clusters.

### ❓ What are the two primary purposes of Kubernetes Namespaces as described in the chapter summary?
Kubernetes uses Namespaces to divide clusters for two primary purposes: resource management and accounting purposes.

### ❓ Explain the concept of cluster address space and how it relates to Kubernetes object naming.
The cluster address space is a DNS domain that we usually call the cluster domain. Every Kubernetes object gets a name in this cluster address space, and you can partition the address space with Namespaces. On most clusters, object names have to be unique within the cluster domain, but namespaces allow partitioning below the cluster domain level.

