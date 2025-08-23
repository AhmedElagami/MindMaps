# PVC Lifecycle

## Naming Convention

## Content

### ❓ What naming convention pattern is used for PVCs created by a StatefulSet volume claim template, and what components are included in each name?
The PVC naming convention includes three components in each name: the volume claim template name, the StatefulSet name, and the associated Pod replica number. The pattern is:

volumeClaimTemplate name + StatefulSet name + Pod replica number

For example:
- webroot (volumeClaimTemplate) + tkb-sts (StatefulSet) + 0 (Pod replica) = webroot-tkb-sts-0
- webroot (volumeClaimTemplate) + tkb-sts (StatefulSet) + 1 (Pod replica) = webroot-tkb-sts-1  
- webroot (volumeClaimTemplate) + tkb-sts (StatefulSet) + 2 (Pod replica) = webroot-tkb-sts-2


## Dynamic PVC Creation
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


## PVC Binding Process
## Content

### ❓ Why was the PVC immediately bound to a volume, and what specific volume binding mode enables this automatic provisioning behavior?
The PVC was immediately bound to a volume because the linode-block-storage StorageClass uses the Immediate volume binding mode. This mode automatically creates the PV and backend storage without waiting for a Pod to claim it, enabling immediate provisioning when the PVC is created.

### ❓ What would be different if the storage class were configured with WaitForFirstConsumer binding mode instead of Immediate?
If the storage class were configured with WaitForFirstConsumer binding mode instead of Immediate, the storage class wouldn't create the PV or back-end volume until you deployed a Pod that used them. This delays volume creation until a Pod requests it, ensuring the volume is created in the same region and zone as the Pod.

### ❓ What is topology-aware provisioning and how does it ensure optimal volume placement relative to Pod location?
Topology-aware provisioning is where the storage class delays volume creation until a Pod requests it. This ensures the storage class will create the volume in the same region and zone as the Pod, providing optimal performance by keeping the storage geographically close to the workload that uses it.

### ❓ What alternative term is sometimes used to describe topology-aware provisioning in Kubernetes storage?
Topology-aware provisioning is sometimes referred to as 'provision on demand' in Kubernetes storage terminology.

### ❓ Explain what the command output reveals about the status of the PVC and PV after deployment but before Pod creation, and why does the PVC remain in Pending state and no PV is created immediately after PVC deployment?
The command output shows the status of the PVC and PV after deployment but before any Pod references the PVC:

```bash
$ kubectl get pvc
NAME            STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS      
pvc-wait-keep   Pending                                      block-wait-keep   

$ kubectl get pv
No resources found
```

The output reveals that:
1. The PVC `pvc-wait-keep` is in the **Pending** state
2. No volume is yet bound to the PVC (VOLUME column is empty)
3. No capacity or access modes are assigned yet
4. No PersistentVolumes (PVs) exist in the cluster

The PVC remains in Pending state and no PV is created immediately because the PVC uses a storage class with the **WaitForFirstConsumer** volume binding mode. This binding mode implements topology-aware provisioning by waiting for a Pod to reference the PVC before creating the PV and back-end volume on LKE. This ensures the storage is provisioned in the same topology/availability zone as the Pod that will use it.

### ❓ What is the relationship between the volume binding mode and topology-aware provisioning as described in the explanation?
The relationship between volume binding mode and topology-aware provisioning is that the **WaitForFirstConsumer** volume binding mode enables topology-aware provisioning. This binding mode delays the creation of the PersistentVolume (PV) and back-end storage volume until a Pod actually references the PersistentVolumeClaim (PVC). By waiting for a Pod to claim the storage, Kubernetes can determine the Pod's scheduling location (node/availability zone) and then provision the storage in the same topology, ensuring optimal performance and data locality. This prevents the scenario where storage might be provisioned in one availability zone while the Pod gets scheduled to a different zone, which could cause performance issues or additional costs.

### ❓ Why didn't the PVC immediately create a volume when it was first created and what triggered the actual creation of the persistent volume and external storage?
The PVC didn't immediately create a volume because the volume binding mode is set to `WaitForFirstConsumer` (delayed binding). The actual creation of the persistent volume and external storage was triggered when you deployed a Pod that used the PVC to request a new 20GB volume. The SC controller observed this and dynamically created PV and the external volume on the Linode cloud.

### ❓ What are the benefits of topology-aware provisioning implemented by this StorageClass?
The topology-aware provisioning implemented by this StorageClass ensures Kubernetes creates volumes in the same region and zone as the Pods by delaying PV creation until a Pod references the PVC.

