# PersistentVolumes

## Content

### ❓ Analyze the following full table output to determine the capacity, access modes, reclaim policy, status, and storage class association of the PV.
The output shows the PV has a CAPACITY of 10Gi, ACCESS MODES of RWO (ReadWriteOnce), RECLAIM POLICY of Delete, STATUS of Bound (to the pvc-test claim), and is associated with the STORAGECLASS 'linode-block-storage'.

```bash
$ kubectl get pv
                              ACCESS   RECLAIM
NAME                   CAPACITY   MODES    POLICY    STATUS   CLAIM      STORAGECLASS         
pvc-cc3dd8716c7d46c9   10Gi       RWO      Delete    Bound    pvc-test   linode-block-storage
```

