# Multiple ReplicaSet Coordination

## Content

### ❓ What information does the 'Annotations' line provide about the deployment's revision history?
The `Annotations` line in the deployment description shows that the object is on revision 2. This indicates that revision 1 was the initial rollout (the original deployment), and the current update being performed is revision 2. Kubernetes maintains this revision history to enable rollback capabilities, keeping track of each version change to the deployment configuration.

### ❓ How does Kubernetes handle scale-up events during a rollout, and how are additional replicas distributed between old and new ReplicaSets?
If a scale-up event occurs during a rollout, Kubernetes will balance the additional replicas across both the old and new ReplicaSets. The distribution is proportional to the current number of replicas each ReplicaSet is managing. 

For example, if the Deployment scales from 10 to 20 replicas (adding 10 new replicas) during a rollout where the old ReplicaSet is managing 3 replicas and the new ReplicaSet is managing 6 replicas, Kubernetes will assign approximately 3 of the new replicas to the old ReplicaSet and approximately 6 to the new one. This proportional distribution ensures that the scaling maintains the same ratio between old and new versions that existed before the scale-up event.

### ❓ What is the purpose of Kubernetes keeping old ReplicaSets, what does the 'kubectl rollout history deployment hello-deploy' command reveal, which image versions were used in Revisions 1 and 2, and what changes were implemented?
Kubernetes keeps old ReplicaSets as a documented revision history and an easy way to roll back. The 'kubectl rollout history deployment hello-deploy' command shows the history of the Deployment with two revisions:

```bash
$ kubectl rollout history deployment hello-deploy
deployment.apps/hello-deploy
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

This output reveals two revisions: Revision 1 was the initial release based on the 1.0 image, and Revision 2 is the rollout that updated the Pods to run version 2.0 of the image.

### ❓ How does Kubernetes maintain revision history through ReplicaSets to enable rollback capabilities?
Kubernetes maintains revision history by keeping old ReplicaSets as documented revision history, which provides an easy way to roll back deployments. Each time a deployment is updated, Kubernetes creates a new ReplicaSet and keeps the old ones. The `kubectl rollout history deployment hello-deploy` command shows this history with two revisions:

```bash
$ kubectl rollout history deployment hello-deploy
deployment.apps/hello-deploy
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

Revision 1 represents the initial release based on the 1.0 image, while Revision 2 represents the rollout that updated the Pods to run version 2.0 of the image. Each revision corresponds to a specific ReplicaSet that maintains the configuration for that version, allowing Kubernetes to easily revert to previous versions by reactivating the appropriate ReplicaSet.

### ❓ What does the 'kubectl get rs' command show about the relationship between ReplicaSets and revisions, and what does 'kubectl describe rs' reveal about the old ReplicaSet's configuration?
The 'kubectl get rs' command shows the two ReplicaSets associated with each revision:

```bash
$ kubectl get rs
NAME                      DESIRED   CURRENT   READY   AGE
hello-deploy-5f84c5b7b7   10        10        10      27m
hello-deploy-54f5d46964   0         0         0       93m
```

The 'kubectl describe rs' command against the old ReplicaSet proves its configuration still references the old image version:

$ kubectl describe rs hello-deploy-54f5d46964Name:           hello-deploy-54f5d46964Namespace:      defaultSelector:       app=hello-world,pod-template-hash=54f5d46964Labels:         app=hello-worldpod-template-hash=54f5d46964Annotations:    deployment.kubernetes.io/desired-replicas: 10deployment.kubernetes.io/max-replicas: 11deployment.kubernetes.io/revision: 1Controlled By:  Deployment/hello-deployReplicas:       0 current / 0 desiredPods Status:    0 Running / 0 Waiting / 0 Succeeded / 0 FailedPod Template:Containers:hello-pod:Image:        nigelpoulton/k8sbook:1.0         <<---- Still configured with old versionPort:         8080/TCP

The key line shows the old ReplicaSet is still configured with the 1.0 image version, meaning flipping the Deployment back to this ReplicaSet will automatically replace all Pods with new ones running the 1.0 image.

