# High Availability

## Content

### ❓ What does Figure 2.3 demonstrate about etcd configurations and split-brain conditions, and why does etcd prefer an odd number of replicas?
Figure 2.3 demonstrates HA and split-brain conditions in etcd configurations:

**What the diagram shows:**
- Cluster A on the left has four nodes and is experiencing a split brain with two nodes on either side and neither having a majority
- Cluster B on the right only has three nodes but is not experiencing a split-brain as Node A knows it does not have a majority, whereas Node B and Node C know they do

**Why etcd prefers an odd number of replicas:**
etcd prefers an odd number of replicas to help avoid split brain conditions. Split brain occurs where replicas experience communication issues and cannot be sure if they have a quorum (majority). With an odd number of nodes, it's always possible to achieve a clear majority decision, preventing the scenario where two equal-sized partitions both believe they have authority.

If a split-brain occurs, etcd goes into read-only mode preventing updates to the cluster. User applications will continue working, but Kubernetes won't be able to scale or update them until the split-brain is resolved.

![Figure 2.3. HA and split-brain conditions](media/figure2-3.png)

### ❓ What three recommendations help protect etcd, the Kubernetes cluster store, against DoS attacks and why should large production clusters consider a dedicated etcd cluster?
The following recommendations help protect etcd against DoS attacks: Configure an HA etcd cluster with either 3 or 5 nodes, configure monitoring and alerting of requests to etcd, and isolate etcd at the network level so that only members of the control plane can interact with it.

Large production clusters should seriously consider a dedicated etcd cluster because a default installation of Kubernetes installs etcd on the same servers as the rest of the control plane, which is fine for development and testing. However, a dedicated etcd cluster will provide better performance and greater resilience. On the performance front, etcd is the most common choking point for large Kubernetes clusters, and operating a dedicated etcd cluster also provides additional resilience by protecting it from other parts of the control plane that might be compromised.

