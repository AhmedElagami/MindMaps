# External Volume Creation

## Content

### ❓ What is the relationship between the PV's existence and its corresponding storage volume on the Linode cloud platform?
The PV's existence indicates that a corresponding storage volume should also exist on the Linode cloud. When a PV is bound to a PVC claim (like pvc-test), it means the underlying storage volume has been provisioned and is available on the cloud provider's infrastructure.

### ❓ What specific verification steps should be taken in the LKE dashboard to confirm the external volume was successfully created?
To verify the external volume was successfully created, you should open your LKE dashboard and navigate to the Volumes tab to confirm you have a 10GB volume with the same name as your PVC (pvc-test) in the same region as your LKE cluster.

