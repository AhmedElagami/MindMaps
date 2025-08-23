# Naming and Identity

## Content

### ❓ How do Pods get their hostnames, what field in the YAML determines this, analyze the following YAML configuration and identify which field sets the Pod hostname for all containers, and what character restrictions apply to Pod names to ensure they are valid DNS hostnames?
Pods get their hostnames from their YAML file's `name` field in the metadata section, and Kubernetes uses this as the hostname for every container in the Pod.

In the YAML configuration:

```bash
kind: Pod
apiVersion: v1
metadata:
  name: hello-pod      <<---- Pod hostname. Inherited by all containers.
  labels:
  <Snip>
```

The `name: hello-pod` field in the metadata section sets the Pod hostname, which is inherited by all containers in the Pod.

Pod names must be valid DNS names, meaning they can only contain characters from a-z, 0-9, the minus sign (-), and the period sign (.).

### ❓ How does the StatefulSet controller ensure predictable Pod naming and what format do these names follow?
The StatefulSet controller ensures predictable Pod naming by following a specific naming convention. Every Pod created by a StatefulSet gets a predictable name following the format `<statefulset-name>-<ordinal-index>`, where the integer is a zero-based index ordinal (a number starting from zero).

For example, with a StatefulSet called "tkb-sts" with three replicas, Kubernetes will name the first replica tkb-sts-0, the second tkb-sts-1, and the third tkb-sts-2.

Pod names are at the core of how StatefulSets start, self-heal, scale, delete Pods, and even how they connect Pods to volumes. Kubernetes also uses the StatefulSet name to create each replica's DNS name, so it's important to avoid using exotic characters that would create invalid DNS names.

### ❓ What command should be run to list StatefulSet Pods and their corresponding nodes, and what information does the kubectl get pods -o wide output provide about each StatefulSet Pod?
To list StatefulSet Pods and their corresponding nodes, run the following command:

```bash
$ kubectl get pods -o wide
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE                         
tkb-sts-0   1/1     Running   0          11m   10.2.0.132   lke343745-544835-5547cbe00000
tkb-sts-1   1/1     Running   0          12h   10.2.0.3     lke343745-544835-1fbc7b870000
tkb-sts-2   1/1     Running   0          21m   10.2.1.7     lke343745-544835-2b6286320000
```

This output provides comprehensive information about each StatefulSet Pod:
- **NAME**: The Pod names (tkb-sts-0, tkb-sts-1, tkb-sts-2)
- **READY**: All Pods are 1/1 ready
- **STATUS**: All Pods are in Running state
- **RESTARTS**: No restarts (0) for all Pods
- **AGE**: Different ages (11m, 12h, 21m) indicating staggered creation
- **IP**: Unique IP addresses for each Pod (10.2.0.132, 10.2.0.3, 10.2.1.7)
- **NODE**: Each Pod is scheduled on a different node (lke343745-544835-5547cbe00000, lke343745-544835-1fbc7b870000, lke343745-544835-2b6286320000)

