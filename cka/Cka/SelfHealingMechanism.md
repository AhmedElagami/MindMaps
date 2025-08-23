# Self-healing Mechanism

## Content

### ❓ What safety mechanism does Kubernetes implement regarding StatefulSet scale-down operations when Pods are in a failed state?
Kubernetes puts scale-down operations on hold if any of the Pods are in a failed state. This protects the resiliency of the app and the integrity of any data by preventing the deletion of Pods that might be experiencing issues.

### ❓ Analyze the complete sequence of events shown in the following kubectl delete and watch commands when a StatefulSet Pod is manually deleted.
The kubectl delete and watch commands show the complete sequence of events when a StatefulSet Pod is manually deleted:

```bash
$ kubectl delete pod tkb-sts-0
pod "tkb-sts-0" deleted

$ kubectl get pods --watch
NAME        READY   STATUS              RESTARTS   AGE
tkb-sts-0   1/1     Running             0          12h
tkb-sts-1   1/1     Running             0          12h
tkb-sts-2   1/1     Running             0          10m
tkb-sts-0   0/1     Terminating         0          12h
tkb-sts-0   0/1     Pending             0          0s
tkb-sts-0   0/1     ContainerCreating   0          0s
tkb-sts-0   1/1     Running             0          8s
```

The sequence shows:
1. The Pod tkb-sts-0 is initially Running
2. After deletion command, it enters Terminating state
3. Then moves to Pending state (0s)
4. Progresses to ContainerCreating state (0s)
5. Finally reaches Running state again after 8 seconds

This demonstrates the StatefulSet controller observing the terminated Pod and immediately creating a replacement with the same name, maintaining the StatefulSet's desired state.

### ❓ How does the complexity of node failure recovery differ between modern and older Kubernetes versions, and what was the reasoning behind this design?
Recovering from potential node failures is a lot more complex and may depend on your Kubernetes version. Modern Kubernetes clusters are far better at automatically replacing Pods from failed nodes, whereas older versions may require manual intervention. This was to prevent Kubernetes from misdiagnosing transient events as catastrophic node failures.

### ❓ What two methods are described for simulating a node failure, and what are the specific procedures for each?
Two methods are described for simulating a node failure: one for LKE clusters and one for Docker Desktop multi-node (kind) clusters.

**Delete a node on LKE:**
Go to your LKE Dashboard, click the Kubernetes tab on the left, and click your cluster's name to open its summary page. Scroll down to your cluster's Node Pool and Recycle one of the nodes running a StatefulSet replica. This will delete and replace the node.

![Figure 13.3 - Delete an LKE cluster node](media/figure13-3.png)

**Delete a node on a Docker Desktop multi-node cluster (kind):**
This only works for Docker Desktop multi-node (kind) clusters, and you'll need to delete and recreate your cluster to restore it back to three nodes. You cannot follow along if you have a Docker Desktop single node (kubeadm) cluster.

Docker Desktop runs cluster nodes as containers, and a three-node cluster will have three containers called desktop-control-plane, desktop-worker, and desktop-worker2.

Open Docker Desktop and navigate to the Containers tab in the left navigation pane. Locate the container with the same name as the worker node you want to delete and click the trash icon to the right of the container to delete it. Be sure to delete a node running a StatefulSet replica.

![Figure 13.4 - Delete a Docker Desktop cluster node](media/figure13-4.png)

### ❓ What is the role of the StatefulSet controller in the recovery process after a node failure, and how long might this process typically take?
Once you've deleted your node, you can run the following command to witness the StatefulSet controller recover from the failure. It may take a minute or two for the process to complete while the StatefulSet controller observes the missing Pod and decides how to act.

The StatefulSet controller's role is to observe the missing Pod (caused by the node failure) and decide how to act to restore the desired state. The recovery process involves detecting the missing Pod, determining the appropriate remediation action, and orchestrating the creation of a replacement Pod while maintaining StatefulSet ordering and identity guarantees.

### ❓ What is the role of the StatefulSet controller when it observes a missing Pod, how long might this process typically take, and what is the complete lifecycle sequence of the tkb-sts-0 Pod including termination, rescheduling, and final state?
The StatefulSet controller observes missing Pods and decides how to act, which may take a minute or two for the process to complete.

Here is the complete lifecycle sequence of the tkb-sts-0 Pod from the kubectl watch output:

```bash
$ kubectl get pods -o wide --watch
NAME        READY   STATUS                RESTARTS   AGE   IP           NODE
tkb-sts-0   1/1     Running               0          14m   10.2.0.132   lke343745...cbe00000
tkb-sts-1   1/1     Running               0          12h   10.2.0.3     lke343745...7b870000
tkb-sts-2   1/1     Running               0          30m   10.2.1.7     lke343745...86320000
tkb-sts-0   1/1     Terminating           0          14m   10.2.0.132   lke343745...cbe00000
tkb-sts-0   0/1     Completed             0          14m   10.2.0.132   lke343745...cbe00000
tkb-sts-0   0/1     Pending               0          0s           
tkb-sts-0   0/1     Pending               0          0s           lke343745...7b870000
tkb-sts-0   0/1     ContainerCreating     0          0s           lke343745...7b870000
tkb-sts-0   0/1     ContainerCreating     0          110s         lke343745...7b870000
tkb-sts-0   1/1     Running               0          111s  10.2.0.4     lke343745...7b870000
```

The tkb-sts-0 Pod was originally on node `lke343745...cbe00000` with IP `10.2.0.132` and was rescheduled to node `lke343745...7b870000` with IP `10.2.0.4` after termination.

The Pod went through the following status sequence: Running → Terminating → Completed → Pending → Pending → ContainerCreating → ContainerCreating → Running. The ContainerCreating phase lasted 110 seconds.

