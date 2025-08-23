# Status Sequence

## Content

### ❓ Analyze the full kubectl watch output and describe the complete lifecycle of the tkb-sts-0 Pod, including all status changes, IP address changes, node reassignments, and explain the sequence of events when Kubernetes detects a node failure and reschedules a StatefulSet Pod, including the pattern of status transitions.
The kubectl watch output shows the complete lifecycle of the tkb-sts-0 Pod during a node failure scenario:

```
$ kubectl get pods -o wide --watch
NAME        READY   STATUS                RESTARTS   AGE   IP           NODE
tkb-sts-0   1/1     Running               0          14m   10.2.0.132   lke343745...cbe00000
tkb-sts-1   1/1     Running               0          12h   10.2.0.3     lke343745...7b870000
tkb-sts-2   1/1     Running               0          30m   10.2.1.7     lke343745...86320000
tkb-sts-0   1/1     Terminating           0          14m   10.2.0.132   lke343745...cbe00000
tkb-sts-0   0/1     Completed             0          14m   10.2.0.132   lke343745...cbe00000
tkb-sts-0   0/1     Pending               0          0s             
tkb-sts-0   0/1     Pending               0          0s             lke343745...7b870000
tkb-sts-0   0/1     ContainerCreating     0          0s             lke343745...7b870000
tkb-sts-0   0/1     ContainerCreating     0          110s           lke343745...7b870000
tkb-sts-0   1/1     Running               0          111s  10.2.0.4     lke343745...7b870000
```

The Pod undergoes the following status transition pattern: Running → Terminating → Completed → Pending → ContainerCreating → Running.

When Kubernetes detects the missing node, the StatefulSet drops from three replicas to two. This creates a mismatch between the observed state and desired state, triggering the StatefulSet controller to create a new copy of the missing tkb-sts-0 replica. The new replica enters the Pending state while the scheduler allocates it to a surviving node (lke343745...7b870000). Once assigned, it enters ContainerCreating state while the node downloads the appropriate image and starts the container (this may appear to hang while Kubernetes releases the previous PVC attachment). Finally, it binds the new replica to the PVC, enters Running state, and the StatefulSet returns to three replicas.

The IP address changes from 10.2.0.132 to 10.2.0.4, and the node reassignment moves from lke343745...cbe00000 to lke343745...7b870000 because the previous node no longer exists.

### ❓ Analyze the full output from the `kubectl get pods -o wide --watch` command and describe the complete lifecycle of pod 'tkb-sts-0' including its status changes, IP address changes, and node reassignment, and what specific status sequence does a StatefulSet pod go through when it's being terminated and rescheduled to a different node?
The `kubectl get pods -o wide --watch` output shows the complete lifecycle of pod 'tkb-sts-0' when it's terminated and rescheduled to a different node. The pod starts on node 'lke343745...cbe00000' with IP address 10.2.0.132 in Running status. The sequence is:

1. **Terminating**: Pod begins termination process (still on original node)
2. **Completed**: Pod has finished termination
3. **Pending**: Pod is scheduled to a new node 'lke343745...7b870000'
4. **ContainerCreating**: Pod is being created on the new node (takes 110 seconds)
5. **Running**: Pod is fully operational on new node with new IP address 10.2.0.4

The pod transitions from the original node 'lke343745...cbe00000' to node 'lke343745...7b870000' and receives a new IP address (from 10.2.0.132 to 10.2.0.4), demonstrating how StatefulSet pods maintain their identity while being rescheduled to different nodes.

