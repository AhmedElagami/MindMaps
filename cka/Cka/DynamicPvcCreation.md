# Dynamic PVC Creation

## Content

### ❓ Analyze the following PVC output table to understand how StatefulSet manages persistent volume claims for each replica.
The PVC output shows three bound volumes, each created at different times corresponding to when each Pod replica was created:

```bash
$ kubectl get pvc
NAME                 STATUS    VOLUME             CAPACITY    MODES    STORAGECLASS    AGE
webroot-tkb-sts-0    Bound     pvc-1146...f274    10Gi        RWO      block           100s
webroot-tkb-sts-1    Bound     pvc-3026...6bcb    10Gi        RWO      block           70s
webroot-tkb-sts-2    Bound     pvc-2ce7...e56d    10Gi        RWO      block           40s
```

This demonstrates that StatefulSet creates a unique PVC for each Pod replica, with each PVC having 10GB capacity, ReadWriteOnce (RWO) access mode, using the 'block' StorageClass. The age differences (100s, 70s, 40s) correspond to the sequential creation pattern of the StatefulSet controller.

