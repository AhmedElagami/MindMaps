# Reconciliation Loop

## Content

### ❓ How does the ReplicaSet controller function as a background process to maintain the desired number of Pod replicas and what specific actions does it take when there are too few or too many Pod replicas running?
Kubernetes implements ReplicaSets as a background controller running in a reconciliation loop, ensuring the correct number of Pod replicas are always present. If there aren't enough Pods, the ReplicaSet adds more. If there are too many, it terminates some. This reconciliation process is fundamental to desired state management.

