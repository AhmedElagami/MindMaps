# Reconciliation Process

## Overview

When desired state and observed state don't match, a controller starts a reconciliation process to bring observed state into sync with desired state.

## Loop Mechanics
## Content

### ❓ How does the ReplicaSet controller function as a background process to maintain the desired number of Pod replicas and what specific actions does it take when there are too few or too many Pod replicas running?
Kubernetes implements ReplicaSets as a background controller running in a reconciliation loop, ensuring the correct number of Pod replicas are always present. If there aren't enough Pods, the ReplicaSet adds more. If there are too many, it terminates some. This reconciliation process is fundamental to desired state management.


## Pod Replica Management
## Content

### ❓ How does the ReplicaSet controller respond when there are only eight Pod replicas present but the desired state requires ten, and what capabilities does the reconciliation process enable beyond just maintaining replica counts?
Assume a scenario where your desired state is ten replicas, but only eight are present. It makes no difference if this is due to failures or if an autoscaler has requested an increase. Either way, the ReplicaSet controller creates two new replicas to sync observed state with desired state. The exact same reconciliation process enables self-healing, scaling, rollouts, and rollbacks.


## State Synchronization
## Content

### ❓ Explain the process of how a Deployment controller handles Pod failures, scaling demands, and application updates according to the theory section.
The Deployment controller handles three main scenarios through reconciliation:

1. **Pod failures**: If the Pod fails, the Deployment controller replaces it with a new one.
2. **Scaling demands**: If demand increases, the Deployment controller can scale the app by deploying more identical Pods.
3. **Application updates**: When you update the app, the Deployment controller deletes the old Pods and replaces them with new ones.

Behind the scenes, Deployments use ReplicaSets that provide self-healing and scaling, while the Deployment controller adds mechanisms for rollouts and rollbacks. The controller follows standard Kubernetes architecture, watching Deployments and reconciling observed state with desired state.

