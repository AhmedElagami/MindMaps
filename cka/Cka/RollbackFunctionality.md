# Rollback Functionality

## Content

### ❓ What does Figure 6.5 demonstrate about the rollback process in Kubernetes Deployments?
Figure 6.5 demonstrates the rollback process in Kubernetes Deployments by showing the same app rolled back to the previous configuration with the earlier ReplicaSet on the left managing all the Pods. This illustrates how Kubernetes can easily revert to previous versions using the preserved ReplicaSet configurations.

![Rollback Process Diagram](media/figure6-5.png)

### ❓ Explain the three specific capabilities Kubernetes provides for fine-grained control over rollouts and rollbacks as listed in the text.
Kubernetes provides three specific capabilities for fine-grained control over rollouts and rollbacks:

1. Insert delays
2. Control the pace and cadence of releases
3. Probe the health and status of updated replicas

These capabilities allow for precise management of deployment processes, ensuring stability and reliability during updates and rollbacks.

### ❓ How does the revisionHistoryLimit setting in a Deployment manifest affect rollback capabilities, and what are the trade-offs of setting this value too high?
The revisionHistoryLimit setting tells Kubernetes to keep the previous five ReplicaSets so you can roll back to the last five versions. Keeping more ReplicaSets gives you more rollback options, providing greater flexibility to revert to previous configurations. However, keeping too many can bloat the object and cause problems on large clusters with lots of releases, as it increases the storage and management overhead for the cluster.

