# API Discovery Mechanism

## Content

### ❓ What is the purpose of running the kubectl api-resources command when building RBAC rule definitions and what does its output reveal about available API resources, their groups, kinds, and supported verbs?
The purpose of running the `kubectl api-resources` command is to show all API resources and supported verbs. The output is useful when you're building rule definitions as it provides a comprehensive list of all available API resources, their API groups, kinds, and the verbs that are supported for each resource.

Here is the full command output:

```bash
$ kubectl api-resources --sort-by name -o wide
NAME          APIGROUP           KIND        VERBS
deployments   apps               Deployment  [create delete ... get list patch update watch]
ingresses     networking.k8s.io  Ingress     [create delete ... get list patch update watch]
pods                             Pod         [create delete ... get list patch update watch]
secrets                          Secret      [create delete ... get list patch update watch]
services                         Service     [create delete get list patch update watch]
<Snip>
```

This output shows the complete landscape of API resources available in the cluster, including their API groups (like "apps" for deployments and "networking.k8s.io" for ingresses), the resource kinds, and the specific verbs that can be performed on each resource type (such as create, delete, get, list, patch, update, watch).

