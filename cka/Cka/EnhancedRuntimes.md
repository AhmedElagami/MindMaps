# Enhanced Runtimes

## Content

### ❓ How do cloud providers contribute to Kubernetes security through confidential computing services?
Many cloud providers implement confidential computing services such as confidential virtual machines and confidential containers that Kubernetes can leverage to secure data in use by enabling memory encryption for container workloads, etc. Some cloud providers even offer it as part of their hosted Kubernetes services.

### ❓ What five security-related technologies are mentioned for augmenting container security and improving isolation?
The five security-related technologies mentioned for augmenting container security and improving isolation are:
1. AppArmor
2. SELinux
3. seccomp
4. capabilities
5. user namespaces

The text notes that "most container runtimes and hosted Kubernetes services do a good job of implementing sensible defaults for them all. However, they can still be complex, especially when troubleshooting."

### ❓ What two specific container runtime examples are provided that offer stronger workload isolation levels?
Two specific container runtime examples that provide stronger levels of workload isolation are:
1. gVisor
2. Kata Containers

The text notes that both are "easy to integrate with Kubernetes thanks to the Container Runtime Interface (CRI) and Runtime Classes."

### ❓ What is the primary security challenge mentioned regarding workload isolation decisions in Kubernetes environments?
The primary security challenge is that while you might feel overwhelmed by the various options, you need to consider all of this when determining the isolation levels your workloads require.

### ❓ How do different runtime classes provide a method for implementing varying levels of workload isolation within a Kubernetes environment?
Using different runtime classes allows you to run all workloads as containers, but you target the workloads requiring stronger isolation to an appropriate container runtime. This provides a flexible approach where you can apply different isolation levels to different workloads based on their security requirements.

