# StorageClasses

## Content

### ❓ What are StorageClasses and in which API group are they resources, and how do they allow applications to define different classes of storage?
Storage classes are resources in the storage.k8s.io API group. As the name suggests, StorageClasses let you define different classes of storage that apps can request. How you define your classes is up to you and will depend on the types of storage you have available.

### ❓ What is the immutability characteristic of StorageClasses and what implication does this have for modifications?
Storage classes are immutable. Once you deploy them, you can't modify them, meaning you cannot change their configuration after creation.

### ❓ Why is the meaningful naming of StorageClasses particularly important in Kubernetes?
The name should be meaningful, as it's how you and other objects refer to the class, making it essential for proper identification and reference in PVCs and other Kubernetes objects.

### ❓ Analyze the output from the 'kubectl get sc' command and compare the Google Kubernetes Engine storage classes with the Linode storage classes. What are the key differences between the two pre-installed Linode storage classes in terms of reclaim policy and default status, and what patterns can you identify in terms of provisioner plugins and volume binding modes?
**Linode Storage Classes Output:**
```bash
$ kubectl get sc
                                                    RECLAIM   VOLUME       ALLOWVOLUME
NAME                          PROVISIONER               POLICY    BINDINGMODE   EXPANSION 
linode-blck-stg               linodebs.csi.linode.com   Delete    Immediate       true    
linode-blck-stg-retain (def)  linodebs.csi.linode.com   Retain    Immediate       true
```

**Key Differences in Linode Storage Classes:**
- Both use the same CSI plugin: `linodebs.csi.linode.com`
- Both use `Immediate` volume binding mode
- `linode-blck-stg` uses `Delete` reclaim policy
- `linode-blck-stg-retain` uses `Retain` reclaim policy and is marked as `(def)` default

**GKE Storage Classes Output:**
```bash
RECLAIM
NAME                 PROVISIONER                    POLICY    VOLUMEBINDINGMODE 
enterprise-multi..   filestore.csi.storage.gke.io   Delete    WaitForFirstConsumer
enterprise-rwx       filestore.csi.storage.gke.io   Delete    WaitForFirstConsumer
premium-rwo          pd.csi.storage.gke.io          Delete    WaitForFirstConsumer
premium-rwx          filestore.csi.storage.gke.io   Delete    WaitForFirstConsumer
standard             kubernetes.io/gce-pd           Delete    Immediate           
standard-rwo (def)   pd.csi.storage.gke.io          Delete    WaitForFirstConsumer
standard-rwx         filestore.csi.storage.gke.io   Delete    WaitForFirstConsumer
zonal-rwx            filestore.csi.storage.gke.io   Delete    WaitForFirstConsumer
```

**Patterns Identified:**
- GKE has multiple provisioner plugins: `filestore.csi.storage.gke.io` (NFS-based Filestore), `pd.csi.storage.gke.io` (block storage), and legacy `kubernetes.io/gce-pd`
- Most GKE classes use `WaitForFirstConsumer` volume binding mode (7 out of 8)
- Only the legacy `standard` class uses `Immediate` binding mode
- All GKE classes use `Delete` reclaim policy
- Linode uses only one provisioner plugin but offers different reclaim policies
- Both environments provide a default storage class marked with `(def)`

### ❓ Analyze the full storage class table output below and identify how many classes use each provisioner type, and which volume binding mode is used by the majority of classes versus the single exception.
```bash
RECLAIM
NAME                 PROVISIONER                    POLICY    VOLUMEBINDINGMODE 
enterprise-multi..   filestore.csi.storage.gke.io   Delete    WaitForFirstConsumer
enterprise-rwx       filestore.csi.storage.gke.io   Delete    WaitForFirstConsumer
premium-rwo          pd.csi.storage.gke.io          Delete    WaitForFirstConsumer
premium-rwx          filestore.csi.storage.gke.io   Delete    WaitForFirstConsumer
standard             kubernetes.io/gce-pd           Delete    Immediate           
standard-rwo (def)   pd.csi.storage.gke.io          Delete    WaitForFirstConsumer
standard-rwx         filestore.csi.storage.gke.io   Delete    WaitForFirstConsumer
zonal-rwx            filestore.csi.storage.gke.io   Delete    WaitForFirstConsumer
```

**Provisioner Distribution:**
- **filestore.csi.storage.gke.io**: 5 classes (enterprise-multi, enterprise-rwx, premium-rwx, standard-rwx, zonal-rwx)
- **pd.csi.storage.gke.io**: 2 classes (premium-rwo, standard-rwo)
- **kubernetes.io/gce-pd**: 1 class (standard)

**Volume Binding Mode:**
- **Majority**: 7 classes use `WaitForFirstConsumer`
- **Single Exception**: 1 class (standard) uses `Immediate`

### ❓ Explain what the full output of the 'kubectl describe sc linode-block-storage' command reveals about this storage class's configuration, including its provisioner, reclaim policy, volume binding mode, and expansion capabilities.
```bash
$ kubectl describe sc linode-block-storage
Name:                  linode-block-storage
IsDefaultClass:        No
Annotations:           lke.linode.com/caplke-version=v1.31.5-2025-02b
Provisioner:           linodebs.csi.linode.com
Parameters:            <none>
AllowVolumeExpansion:  True
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                <none>
```

The output reveals the following configuration details:

- **Provisioner**: `linodebs.csi.linode.com` (Linode's CSI driver for block storage)
- **Reclaim Policy**: `Delete` (PV and associated back-end volume will be automatically deleted when PVC is removed)
- **Volume Binding Mode**: `Immediate` (PV binding occurs immediately upon PVC creation)
- **Expansion Capabilities**: `AllowVolumeExpansion: True` (volumes can be expanded after creation)
- **Default Class**: `No` (this is not the default storage class)
- **Parameters**: `<none>` (no additional parameters configured)
- **Mount Options**: `<none>` (no special mount options specified)

### ❓ What are the three key properties that distinguish the block-wait-keep storage class from other storage classes?
The block-wait-keep storage class has three key distinguishing properties: 1) Block storage type, 2) Topology aware provisioning (WaitForFirstConsumer binding mode), and 3) Keeps volume and data when the PVC is deleted (Retain reclaim policy).

### ❓ Analyze the following full StorageClass YAML definition to explain the purpose of each annotated component: provisioner, volumeBindingMode, and reclaimPolicy.
The StorageClass YAML defines a custom storage class with three key annotated components:

1. **provisioner: linodebs.csi.linode.com** - Specifies the CSI Plugin that will handle the volume provisioning on the Linode cloud platform
2. **volumeBindingMode: WaitForFirstConsumer** - Implements topology-aware provisioning by delaying volume creation until a Pod requests it
3. **reclaimPolicy: Retain** - Keeps the volume and data when the PVC is deleted, preventing automatic cleanup

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: block-wait-keep
provisioner: linodebs.csi.linode.com       <<---- CSI Plugin
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer    <<---- Provision on demand
reclaimPolicy: Retain                      <<---- Keep volume when PVC released
```

### ❓ What is the purpose of the command sequence shown in the full code block to deploy and verify the StorageClass, and analyze the output table from the 'kubectl get sc' command to compare the provisioning and binding characteristics of the three StorageClasses shown?
The command sequence demonstrates how to deploy and verify a new StorageClass called `block-wait-keep`. The first command `kubectl apply -f lke-sc-wait-keep.yml` creates the StorageClass, and the second command `kubectl get sc` displays all available StorageClasses for comparison.

Here is the complete output table showing the three StorageClasses and their characteristics:

```bash
$ kubectl get sc
                                                                                   ALLOW
                                                    RECLAIM  VOLUME                VOLUME
NAME                          PROVISIONER               POLICY   BINDINGMODE           EXPANSION
block-wait-keep               linodebs.csi.linode.com   Retain   WaitForFirstConsumer  true
linode-blck-stg               linodebs.csi.linode.com   Delete   Immediate             true
linode-blck-stg-retain (def)  linodebs.csi.linode.com   Retain   Immediate             true
```

The comparison reveals:
- **block-wait-keep**: Uses `Retain` reclaim policy, `WaitForFirstConsumer` binding mode, and allows volume expansion
- **linode-blck-stg**: Uses `Delete` reclaim policy, `Immediate` binding mode, and allows volume expansion  
- **linode-blck-stg-retain (default)**: Uses `Retain` reclaim policy, `Immediate` binding mode, and allows volume expansion

The key difference is that `block-wait-keep` uses `WaitForFirstConsumer` binding mode while the other two use `Immediate` binding mode.

### ❓ What specific StorageClass was created and what cloud provider does it provision storage from?
You created a new StorageClass called `block-wait-keep` that provisions block storage from the Linode cloud.

### ❓ What action did the StorageClass controller take after the new StorageClass was created?
The StorageClass controller started watching the API server for new PVCs referencing your new block-wait-keep class.

### ❓ What is the relationship between StorageClasses and external storage systems?
Once you've installed the CSI plugin, you create StorageClasses that map to a type of storage on the external system.

### ❓ Explain the purpose and configuration of each field in the following StorageClass YAML definition for LKE block storage.
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: block                                <<---- The PVC will reference this name
provisioner: linodebs.csi.linode.com         <<---- LKE block storage CSI plugin
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```

This StorageClass YAML defines a storage class for Linode Kubernetes Engine (LKE) with the following configuration:

- **apiVersion**: `storage.k8s.io/v1` - The Kubernetes API version for StorageClass objects
- **kind**: `StorageClass` - Specifies this is a StorageClass resource
- **metadata.name**: `block` - The name of the StorageClass that PersistentVolumeClaims will reference
- **provisioner**: `linodebs.csi.linode.com` - The LKE block storage CSI driver that will dynamically provision block storage from the Linode Cloud
- **allowVolumeExpansion**: `true` - Allows volumes created with this StorageClass to be expanded later
- **volumeBindingMode**: `WaitForFirstConsumer` - Delays volume binding and provisioning until a Pod using the PersistentVolumeClaim is created
- **reclaimPolicy**: `Delete` - Specifies that the underlying storage should be deleted when the PersistentVolume is deleted

This StorageClass enables dynamic provisioning of block storage volumes that StatefulSets can use for persistent storage.

### ❓ What command is used to deploy the StorageClass, what output confirms successful creation, and what information does the 'kubectl get sc' command output provide?
The command used to deploy the StorageClass is:

```bash
$ kubectl apply -f lke-sc.yml
storageclass.storage.k8s.io/block created
```

The output `storageclass.storage.k8s.io/block created` confirms the successful creation of the StorageClass.

The `kubectl get sc` command provides the following information about deployed StorageClasses:

```bash
$ kubectl get sc
                                                                           ALLOWVOLUME
NAME     PROVISIONER               RECLAIMPOLICY    VOLUMEBINDINGMODE      EXPANSION
block    linodebs.csi.linode.com   Delete           WaitForFirstConsumer   true
```

This output shows:
- **NAME**: The name of the StorageClass ("block")
- **PROVISIONER**: The storage provisioner/driver ("linodebs.csi.linode.com")
- **RECLAIMPOLICY**: The reclaim policy ("Delete")
- **VOLUMEBINDINGMODE**: The volume binding mode ("WaitForFirstConsumer")
- **ALLOWVOLUMEEXPANSION**: Whether volume expansion is allowed ("true")

This verification confirms that the StorageClass is present and properly configured for the StatefulSet to use for dynamic volume creation.

### ❓ Analyze the following full StorageClass output table to identify the provisioner, reclamation policy, volume binding mode, and expansion capability for the 'block' StorageClass.
```bash
$ kubectl get sc
                                                                           ALLOWVOLUME
NAME     PROVISIONER               RECLAIMPOLICY    VOLUMEBINDINGMODE      EXPANSION
block    linodebs.csi.linode.com   Delete           WaitForFirstConsumer   true
```

The output shows the following properties for the 'block' StorageClass:
- **Provisioner**: linodebs.csi.linode.com
- **Reclaim Policy**: Delete
- **Volume Binding Mode**: WaitForFirstConsumer
- **Expansion Capability**: true

### ❓ What is the purpose of listing your cluster's StorageClasses before deploying a StatefulSet and what is the relationship between a StatefulSet and its StorageClass?
The purpose of listing your cluster's StorageClasses before deploying a StatefulSet is to verify that your required StorageClass is present and available. This verification step is important because the StatefulSet will use the StorageClass to create new volumes dynamically. The StatefulSet relies on the StorageClass to provision persistent volumes for each replica, ensuring that each Pod gets its own dedicated storage.

### ❓ What modification is required for the volume claim template when not using an LKE cluster with custom StorageClasses?
If you're not using an LKE cluster and you're using one of your cloud's built-in StorageClasses, you'll need to edit the YAML file and change the storageClassName field to a StorageClass that exists on your cluster. The configuration will work properly if you created your own StorageClass and called it 'block'.

### ❓ What type of Kubernetes resource is being targeted for deletion in this instruction, and what role does it play in StatefulSet storage management, and explain the command syntax and purpose for deleting the StorageClass named 'flash'?
The instruction targets a StorageClass named 'flash' for deletion. StorageClasses play a critical role in StatefulSet storage management by defining the storage provisioning behavior, including:

- Storage backend type (SSD, HDD, etc.)
- Provisioner type (cloud provider, CSI driver)
- Reclaim policy (Delete, Retain)
- Volume binding mode
- Other storage parameters

For StatefulSets, StorageClasses dynamically provision PersistentVolumes when PersistentVolumeClaims are created, enabling automated storage management for stateful applications.

The command `$ kubectl delete sc flash` uses the `delete` subcommand with `sc` (StorageClass) as the resource type and `flash` as the specific StorageClass name. This removes the StorageClass definition from the cluster, cleaning up the storage provisioning configuration that was used by the StatefulSet. However, note that deleting a StorageClass does not affect already provisioned volumes - those must be deleted through PVC deletion first.

### ❓ What command is used to delete a storage class named 'flash' in Kubernetes?
The command to delete a storage class named 'flash' in Kubernetes is:

```bash
$ kubectl delete sc flash
```

