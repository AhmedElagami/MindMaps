# API Organization

## Content

### ❓ Analyze the following API group and resource mapping table and explain how different API groups correspond to different Kubernetes API paths and what does an empty set of double quotes in the apiGroups field indicate, and how should other API groups be specified?
The API group and resource mapping table shows how different API groups correspond to different Kubernetes API paths:

| apiGroup | resource | Kubernetes API path |
| --- | --- | --- |
| "" | pods | /api/v1/namespaces/{namespace}/pods |
| "" | secrets | /api/v1/namespaces/{namespace}/secrets |
| "storage.k8s.io" | storageclass | /apis/storage.k8s.io/v1/storageclasses |
| "apps" | deployments | /apis/apps/v1/namespaces/{namespace}/deployments |

This demonstrates that:
- Core resources (like pods and secrets) use the core API group (indicated by empty quotes) and map to `/api/v1/` paths
- Extended resources (like storageclass and deployments) use specific API groups and map to `/apis/{apiGroup}/v1/` paths

An empty set of double quotes ("") in the apiGroups field indicates the core API group. You need to specify all other API groups as a string enclosed in double-quotes, such as "apps" for the apps API group or "storage.k8s.io" for the storage API group.

### ❓ What are the two main types of API groups in Kubernetes, how do they differ historically, and what is the purpose of named API groups?
There are two types of API group: The core group and The named groups. The core group is where we define all the original objects from when Kubernetes was new (before it grew and we divided the API into groups). The named API groups are where we add all new resources, and we sometimes call them sub-groups. Each of the named groups is a collection of related resources.

### ❓ What is the key difference in URI path structure between named API groups and the core API group, and how is the core API group sometimes represented?
The key difference in URI path structure is that named API groups start with `/apis` (plural) and include the name of the group, while the core API group starts with `/api` (singular) and doesn't include a group name. In fact, in some places, you'll see the core API group referred to by empty double quotes (""). This is because no thought was given to groups when we originally created the API — everything was "just in the API".

### ❓ How do the REST paths for named group resources differ structurally from those in the core group?
Resources in the named groups live below the /apis REST path. The following table lists some examples:

- Resource: Ingress, Path: /apis/networking.k8s.io/v1/namespaces/{namespace}/ingresses/
- Resource: ClusterRole, Path: /apis/rbac.authorization.k8s.io/v1/clusterroles/
- Resource: StorageClass, Path: /apis/storage.k8s.io/v1/storageclasses/

Notice how the URI paths for named groups start with /apis (plural) and include the name of the group.

### ❓ What are the three main benefits of dividing the Kubernetes API into smaller groups?
Dividing the API into smaller groups makes it more scalable and easier to navigate and extend.

