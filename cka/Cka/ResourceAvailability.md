# Resource Availability

## Content

### ❓ What are the two types of resource specifications that Kubernetes allows for every container in a Pod, and what do resource requests and resource limits represent?
Kubernetes lets you specify two types of resource specifications for every container in a Pod: resource requests and resource limits. Requests are minimum values that represent the guaranteed minimum resources a container needs. Limits are maximum values that represent the upper bound of resources a container is allowed to use.

### ❓ Analyze the following resource specification YAML snippet and explain the purpose of both the requests and limits sections with their specific values.
The resource specification YAML snippet demonstrates both requests (minimums for scheduling) and limits (maximums for kubelet to cap):

```bash
resources:
  requests:              <<---- Minimums for scheduling
    cpu: 0.5
    memory: 256Mi
  limits:                <<---- Maximums for kubelet to cap
    cpu: 1.0
    memory: 512Mi
```

The requests section specifies minimum requirements: 0.5 CPU (half a CPU) and 256Mi of memory - these are the resources guaranteed to the container. The limits section specifies maximum allowed usage: 1.0 CPU (one full CPU) and 512Mi of memory - these are the upper bounds that the container cannot exceed.

### ❓ What three resource access guarantees does Kubernetes provide for executing containers?
Kubernetes provides three resource access guarantees for executing containers: 1) While a container executes, it is guaranteed access to its minimum requirements (requests), 2) It can also use more if the node has additional resources available, 3) But it's never allowed to use more than what you specify in its limits.

### ❓ How does the scheduler handle resource allocation for multi-container Pods?
For multi-container Pods, the scheduler combines the requests for all containers and finds a node with enough resources to satisfy the full Pod.

### ❓ What is the consequence of not specifying resource requests when limits are defined in a Kubernetes Pod?
This is because Kubernetes automatically sets requests to match limits if you don't specify requests.

### ❓ What automatic behavior does Kubernetes exhibit when resource requests are not explicitly specified but limits are provided?
Kubernetes automatically sets requests to match limits if you don't specify requests. This explains why some command outputs display both limits and requests even when only limits were specified in the deployment configuration.

