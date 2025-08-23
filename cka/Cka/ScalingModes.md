# Scaling Modes

## Content

### ❓ Explain the key difference between scaling up and scaling down StatefulSets in terms of Pod and PVC management, how does Kubernetes determine which Pod to delete when scaling down a StatefulSet, and what happens to the associated PVC?
The key difference between scaling up and scaling down StatefulSets is in how they handle Pods and Persistent Volume Claims (PVCs). Each time Kubernetes scales up a StatefulSet, it creates new Pods and new PVCs. However, when scaling down, Kubernetes only terminates Pods - it does not delete the associated PVCs. This means future scale-up operations only need to create new Pods and connect them back to the original PVCs.

When scaling down, Kubernetes deletes the Pod with the highest index ordinal first. For example, when scaling from 3 to 2 replicas, Kubernetes will delete tkb-sts-2 (the Pod with the highest ordinal number). The PVC associated with the deleted Pod (webroot-tkb-sts-2) remains intact and shows a status of "Bound" even though the Pod no longer exists. This preservation of PVCs during scale-down operations ensures that when you scale back up, the new Pod can automatically connect to the correct existing volume without data loss.

### ❓ According to the paragraph, which specific Pod gets deleted when scaling down a StatefulSet, what happens to the associated PVCs, and how many PVCs remain with what statuses?
When scaling a StatefulSet down, Kubernetes deletes the Pod with the highest index ordinal. Scaling a StatefulSet down does not delete PVCs, so all three PVCs remain. The PVC status output shows:

```bash
$ kubectl get pvc
NAME                 STATUS    VOLUME             CAPACITY    MODES    STORAGECLASS    AGE
webroot-tkb-sts-0    Bound     pvc-5955...d71c    10Gi        RWO      block           12h
webroot-tkb-sts-1    Bound     pvc-d62c...v701    10Gi        RWO      block           12h
webroot-tkb-sts-2    Bound     pvc-2e2f...5f95    10Gi        RWO      block           12h
```

All three PVCs maintain their 'Bound' status even though the tkb-sts-2 Pod no longer exists.

### ❓ Compare and contrast the default StatefulSet scaling behavior with the parallel scaling option, including the key differences in Pod creation and deletion patterns.
The default StatefulSet scaling behavior (OrderedReady) enforces starting one Pod at a time and waiting for the previous Pod to be running and ready before starting the next. Changing the value to Parallel will cause the StatefulSet to act more like a Deployment where Pods are created and deleted in parallel. For example, scaling from 2 > 5 Pods will instantly create all three new Pods, whereas scaling down from 5 > 2 will delete three Pods in parallel. StatefulSet naming rules are still enforced, as the setting only applies to scaling operations and does not impact rollouts and rollbacks.

