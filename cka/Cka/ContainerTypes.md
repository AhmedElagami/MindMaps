# Container Types

## Content

### ❓ How does the custom-columns kubectl command demonstrate the difference between init containers and regular containers in the Pod specification?
The custom-columns kubectl command clearly demonstrates the difference between init containers and regular containers in the Pod specification:

```bash
$ kubectl get pod -o "custom-columns="\
  "NAME:.metadata.name,"\
  "INIT:.spec.initContainers[*].name,"\
  "CONTAINERS:.spec.containers[*].name"

NAME         INIT         CONTAINERS
git-sync     ctr-sync     ctr-web
```

This output shows that:
- The Pod has one init container named "ctr-sync" (shown in the INIT column)
- The Pod has one regular container named "ctr-web" (shown in the CONTAINERS column)
- Kubernetes deployed the ctr-sync container as an init container and the ctr-web container as a regular container

### ❓ What are the two structural configurations that Pods can have?
Pods can be single-container or multi-container.

