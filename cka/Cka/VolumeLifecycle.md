# Volume Lifecycle

## Content

### ❓ What are the two reclaim policies available for PVs and what triggers their execution, explain why the Delete reclaim policy is considered dangerous and what specific actions occur when it is triggered, and what are the advantages and disadvantages of using the Retain reclaim policy compared to Delete?
**Two Reclaim Policies:**
1. **Delete** - The most dangerous and is the default for PVs created dynamically via StorageClasses
2. **Retain** - Will keep the PV and external storage after you delete the PVC

**Delete Policy Risks:** Delete is considered dangerous because it deletes the PV and associated external storage when the PVC is released. This means deleting the PVC will delete the PV and the external storage. Use with caution as this results in permanent data loss.

**Retain Policy Analysis:** Retain will keep the PV and external storage after you delete the PVC. It's the safer option as it preserves data, but you have to reclaim resources manually, which requires additional administrative overhead compared to the automatic cleanup provided by the Delete policy.

Both policies are triggered when the PVC that was bound to the PV is released or deleted.

### ❓ What is the significance of the Delete reclaim policy mentioned in the paragraph, and what specific action does Kubernetes take when this policy is in effect?
The Delete reclaim policy is significant because it means Kubernetes will automatically delete the Persistent Volume (PV) and the associated back-end volume when you stop using the Persistent Volume Claim (PVC). This automatic cleanup prevents orphaned storage resources and helps manage storage costs efficiently.

### ❓ Why does deleting the PVC also delete the PV and associated backend volume, and what reclaim policy enables this behavior?
Deleting the PVC also deletes the PV and associated backend volume because the storage class created them with the reclaim policy set to Delete. This policy enables automatic cleanup of both the Kubernetes PV object and the underlying cloud storage volume when the PVC is released.

### ❓ Why are the PV and external volume not automatically deleted when the Pod and PVC are removed?
Even though you've deleted the Pod and the PVC, Kubernetes hasn't deleted the PV or external volume because the storage class created them with the `Retain` reclaim policy. This means you'll have to delete them manually.

### ❓ What specific indicator in the Linode cloud console helps identify which volume to delete and what is the consequence of not manually deleting the external volume from the cloud provider?
In the Linode cloud console, the volume will show as "unattached" in the "Attached to" column, which helps identify which volume to delete. Not deleting the volume will incur unwanted Linode costs.

### ❓ What is the financial consequence of not deleting the volume as mentioned in the text?
Not deleting the volume will incur unwanted Linode costs.

