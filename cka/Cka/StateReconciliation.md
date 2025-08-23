# State Reconciliation

## Content

### ❓ Explain the process of how a Deployment controller handles Pod failures, scaling demands, and application updates according to the theory section.
The Deployment controller handles three main scenarios through reconciliation:

1. **Pod failures**: If the Pod fails, the Deployment controller replaces it with a new one.
2. **Scaling demands**: If demand increases, the Deployment controller can scale the app by deploying more identical Pods.
3. **Application updates**: When you update the app, the Deployment controller deletes the old Pods and replaces them with new ones.

Behind the scenes, Deployments use ReplicaSets that provide self-healing and scaling, while the Deployment controller adds mechanisms for rollouts and rollbacks. The controller follows standard Kubernetes architecture, watching Deployments and reconciling observed state with desired state.

