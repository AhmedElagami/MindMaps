# Rollout Management

## Content

### ❓ Explain the process of StatefulSet rollouts, including the order of Pod replacement and the controller's behavior during the update.
StatefulSets support rolling updates (a.k.a. rollouts). You update the image version in the YAML file and re-post it to the API server, and the controller replaces the old Pods with new ones. However, it always starts with the highest numbered Pod and works down through the list, one at a time, until all Pods are on the new version. The controller also waits for each new Pod to be ready before replacing the one with the next lowest index ordinal.

