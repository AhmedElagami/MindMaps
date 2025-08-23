# Replica and Volume Management

## Content

### ❓ What does the following kubectl apply command accomplish when scaling the StatefulSet?
The kubectl apply command is used to update the StatefulSet configuration and trigger scaling operations:

```bash
$ kubectl apply -f sts.yml
statefulset.apps/tkb-sts configured
```

This command re-posts the updated configuration from the sts.yml file to the cluster. When you edit the sts.yml file to change the replica count (for example, from 3 to 2 or back to 3), running this command applies those changes and causes the StatefulSet controller to either scale down by terminating Pods or scale up by creating new Pods while reconnecting them to existing PVCs.

### ❓ What specific verification step should be performed after scaling a StatefulSet to confirm the Pod count has been reduced, and what does the kubectl get sts output reveal about the current state?
After scaling a StatefulSet down, you should check the StatefulSet and verify the Pod count has reduced to the desired number. The kubectl get sts command output shows:

```bash
$ kubectl get sts tkb-sts
NAME      READY   AGE
tkb-sts   2/2     12h
```

This output reveals that the StatefulSet named 'tkb-sts' now has 2 out of 2 Pods ready, confirming the successful scale-down operation.

### ❓ What command sequence is used to re-deploy the StatefulSet after modifying the replica count, and what does the subsequent kubectl get sts output indicate about scaling back up?
After modifying the replica count in the YAML file, you use the following command to re-deploy the StatefulSet:

```bash
$ kubectl apply -f sts.yml
statefulset.apps/tkb-sts configured
```

After the new Pod deploys, the kubectl get sts output shows:

```bash
$ kubectl get sts tkb-sts
NAME      READY   AGE
tkb-sts   3/3     12h
```

This indicates the StatefulSet has successfully scaled back up to 3 replicas with all 3 Pods ready.

### ❓ What is the recommended procedure for safely deleting a StatefulSet according to the analysis, and what command sequence is provided to implement this procedure?
```
- Earlier in the chapter, we said that Kubernetes doesn't terminate Pods in order when you delete a StatefulSet
  - Therefore, if your applications and data are sensitive to ordered shutdown, you should scale the StatefulSet to zero before deleting it
- Scale your StatefulSet to 0 replicas and confirm the operation
  - It may take a few seconds to scale all the way down to 0
- ```bash
$ kubectl scale sts tkb-sts --replicas=0
statefulset.apps/tkb-sts scaled

$ kubectl get sts tkb-sts
NAME      READY   AGE
tkb-sts   0/0     13h
```

The recommended procedure for safely deleting a StatefulSet is to scale it to zero replicas before deletion. This is necessary because Kubernetes doesn't terminate Pods in order when you delete a StatefulSet directly. If your applications and data are sensitive to ordered shutdown, scaling to zero ensures proper shutdown sequence. The command sequence provided is:

```bash
$ kubectl scale sts tkb-sts --replicas=0
$ kubectl get sts tkb-sts
```

This scales the StatefulSet down to 0 replicas (which may take a few seconds) and then confirms the operation by checking the StatefulSet status.

### ❓ What is the significance of being able to delete a StatefulSet as soon as it reaches zero replicas, and what does this indicate about StatefulSet management?
The ability to delete a StatefulSet as soon as it reaches zero replicas indicates that StatefulSet management in Kubernetes is designed to be clean and efficient. This means that once all pods of the StatefulSet have been terminated and removed, the StatefulSet controller has completed its work and the StatefulSet object itself can be safely deleted without leaving orphaned resources. This demonstrates that StatefulSets are managed as complete units where the controller properly handles the lifecycle from creation through termination.

### ❓ Explain the command and its output shown for deleting the StatefulSet 'tkb-sts'.
The command `$ kubectl delete sts tkb-sts` is used to delete the StatefulSet named 'tkb-sts'. The output `statefulset.apps "tkb-sts" deleted` confirms that the StatefulSet resource was successfully deleted from the Kubernetes cluster. This command removes the StatefulSet controller but does not automatically delete the associated PersistentVolumeClaims, Services, or StorageClass resources, which must be cleaned up separately.

