# Pod Replica Management

## Content

### ❓ How does the ReplicaSet controller respond when there are only eight Pod replicas present but the desired state requires ten, and what capabilities does the reconciliation process enable beyond just maintaining replica counts?
Assume a scenario where your desired state is ten replicas, but only eight are present. It makes no difference if this is due to failures or if an autoscaler has requested an increase. Either way, the ReplicaSet controller creates two new replicas to sync observed state with desired state. The exact same reconciliation process enables self-healing, scaling, rollouts, and rollbacks.

