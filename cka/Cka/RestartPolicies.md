# Restart Policies

## Content

### ❓ Explain what the following kubectl explain command reveals about the structure and complexity of Pod specifications, and what are the three possible values for a Pod's restartPolicy as shown in the kubectl explain output, and what does each value mean?
The `kubectl explain pods --recursive | more` command reveals the extensive structure and complexity of Pod specifications, returning over 1,000 lines of output that detail all possible Pod attributes and their hierarchical organization.

```bash
$ kubectl explain pods --recursive | more
KIND:     Pod
VERSION:  v1
DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.
FIELDS:
   apiVersion	      <string>
   kind	            <string>
   metadata	        <Object>
      annotations	  <map[string]string>
      labels	      <map[string]string>
      name	        <string>
      namespace	    <string>
<Snip>
```

The `kubectl explain pod.spec.restartPolicy` command specifically drills into the Pod restartPolicy attribute and shows the three possible values:

```bash
$ kubectl explain pod.spec.restartPolicy
KIND:     Pod
VERSION:  v1
FIELD:    restartPolicy <string>
DESCRIPTION:
     Restart policy for all containers within the pod. One of Always, OnFailure, Never. 
     Default to Always. 
     More info: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/...
     Possible enum values:
     - `"Always"`
     - `"Never"`
     - `"OnFailure"`
```

The three possible values for a Pod's restartPolicy are:
1. **Always**: Will always attempt to restart a container
2. **Never**: Will never attempt a restart
3. **OnFailure**: Will only attempt a restart if the container fails with an error code

The policy is Pod-wide, meaning it applies to all containers in the Pod except for init containers.

### ❓ What happens when a node running a Pod managed by a Deployment controller fails, and how does this process demonstrate the concept of Pod replacement rather than restart?
When a node running a Pod managed by a Deployment controller fails, the Deployment controller notices the failed node, deletes the Pod, and replaces it with a new one on a surviving node. Even though the new Pod is based on the same Pod spec, it has a new UID, a new IP address, and no state. This demonstrates that Kubernetes never restarts Pods - when they fail, get scaled up and down, and get updated, Kubernetes always deletes old Pods and creates new ones.

### ❓ What are the three possible restart policy values for Pod containers, under what circumstances does each policy trigger container restarts, and how does the choice differ between long-living and short-living containers?
The three possible restart policy values for Pod containers are: Always, Never, and OnFailure.

The values are self-explanatory: Always will always attempt to restart a container, Never will never attempt a restart, and OnFailure will only attempt a restart if the container fails with an error code. The policy is Pod-wide, meaning it applies to all containers in the Pod except for init containers.

The restart policy you choose depends on the nature of the app - whether it's a long-living container or a short-living container. Long-living containers host apps such as web servers, data stores, and message queues that run indefinitely. If they fail, you normally want to restart them, so you'll typically give them the Always restart policy.

Short-living containers are different and typically run batch-style workloads that run a task through to completion. Most of the time, you're happy when they complete, and you only want to restart them if they fail. As such, you'll probably give them the OnFailure restart policy. If you don't care if they fail, give them the Never policy.

