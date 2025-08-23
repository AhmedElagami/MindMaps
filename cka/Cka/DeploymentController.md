# Deployment Controller

## Content

### ❓ What Kubernetes component is responsible for managing the deletion and replacement of Pods during a rollout?
The Kubernetes component responsible for managing the deletion and replacement of Pods during a rollout is the Deployment controller. When you post an updated manifest file to Kubernetes, the Deployment controller handles the process of deleting every Pod running the old version and replacing them with new Pods running the updated version, following the rollout strategy specified in the Deployment configuration.

### ❓ Where are Deployments defined and what type of controller implements them on the Kubernetes control plane?
Deployments are defined in the API and implement a controller running as a reconciliation loop on the control plane.

