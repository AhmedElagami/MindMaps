# Anti-Affinity Distribution

## Content

### ❓ Based on the NODE column in the kubectl output, what scheduling pattern did Kubernetes use for the StatefulSet replicas, and what specific node is tkb-sts-0 running on?
Looking closely at the NODE column, you'll see that Kubernetes has scheduled each replica on different nodes. This demonstrates that Kubernetes uses an anti-affinity scheduling pattern for StatefulSet replicas, ensuring they are distributed across different nodes for high availability.

In the example, tkb-sts-0 is running on the lke343...cbe00000 node.

