# Health Assessment

## Content

### ❓ What does the following kubectl get pods output indicate about the current state of the StatefulSet Pods?
The kubectl get pods output shows that all three StatefulSet Pods are healthy and running:

```bash
$ kubectl get pods
NAME        READY   STATUS   AGE
tkb-sts-0   1/1     Running  12h
tkb-sts-1   1/1     Running  12h
tkb-sts-2   1/1     Running  9m49s
```

This indicates that the StatefulSet has three replicas (tkb-sts-0, tkb-sts-1, and tkb-sts-2), all with their containers ready (1/1), in the Running status, with ages of 12 hours for the first two and 9 minutes 49 seconds for the third Pod.

