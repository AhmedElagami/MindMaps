# Provisioning Process

## Content

### ❓ What does the diagram in Figure 11.1 illustrate about the high-level architecture of Kubernetes storage and what does the diagram in Figure 11.2 show about the volume provisioning workflow with AWS?
Figure 11.1 shows the high-level architecture of Kubernetes storage, which consists of three main components:

1. **Storage providers on the left** - These are the external systems providing advanced storage services, which can be on-premises systems such as EMC and NetApp or storage services provided by your cloud
2. **The plugin layer in the middle** - This is the interface between the external storage systems and Kubernetes, where modern plugins use the Container Storage Interface (CSI), which is an industry-standard storage interface for container orchestrators such as Kubernetes
3. **The Kubernetes persistent volume subsystem on the right** - This is a standardized set of API objects that make it easy for applications to consume storage

![Figure 11.1](media/figure11-1.png)

Figure 11.2 shows the volume provisioning workflow with AWS, which demonstrates how the components work together:

1. The Pod on the far right needs a 50GB volume and requests it via a PersistentVolumeClaim (PVC)
2. The PVC asks the StorageClass to create a new PV and associated volume on the AWS backend
3. The SC makes the call to the AWS backend via the AWS CSI plugin
4. The CSI plugin creates the 50GB EBS volume on AWS
5. The CSI plugin reports the creation of the external volume back to the SC
6. The SC creates the PV and maps it to the EBS volume on the AWS back end
7. The Pod mounts the PV and uses it

![Figure 11.2 - Volume provisioning workflow](media/figure11-2.png)

### ❓ Describe the complete workflow from PVC creation to Pod storage access as outlined in the content.
The complete workflow from PVC creation to Pod storage access is:
1. You create a YAML file defining a Pod that references a PVC requesting a volume from a specific storage class
2. You deploy the app by sending the YAML file to the API server
3. The SC controller observes the new PVC and instructs the CSI plugin to provision storage on the external storage system
4. The external system creates the volume and reports back to the CSI plugin, which then informs the SC controller that maps it to a new PV
5. The Pod uses the PVC to mount and use the PV

### ❓ What are the three main steps in the basic workflow for deploying and using a StorageClass?
The basic workflow for deploying and using a StorageClass is as follows:
1. Install and configure the CSI plugin
2. Create one or more StorageClasses
3. Deploy Pods with PVCs that request volumes via the StorageClasses

### ❓ Explain the purpose and relationship between the Pod, PVC, and SC objects defined in the following YAML file, and how they are separated within the same document, and analyze the complete workflow from Pod creation to storage provisioning, referencing each of the eight numbered annotations.
The YAML file defines three Kubernetes objects that work together to provide dynamic storage provisioning: a Pod, a PersistentVolumeClaim (PVC), and a StorageClass (SC). These objects are separated within the same document using three dashes (---) as YAML document separators.

**Complete Workflow with Numbered Annotations:**

```yaml
apiVersion: v1
kind: Pod                             <<---- 1. Pod
metadata:
  name: mypod
spec:
  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: mypvc              <<---- 2. Request volume via the "mypvc" PVC
  containers: ...
  <SNIP>
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc                         <<---- 3. This is the "mypvc" PVC
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi                   <<---- 4. Provision a 50Gi volume...
  storageClassName: fast              <<---- 5. ...based on the "fast" StorageClass
---
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast                          <<---- 6. This is the "fast" StorageClass
provisioner: pd.csi.storage.gke.io    <<---- 7. Use this CSI plugin
parameters:
  type: pd-ssd                        <<---- 8. Provision this type of storage
```

**Workflow Explanation:**
1. A normal Pod object is defined
2. The Pod requests a volume via the "mypvc" PVC
3. The file defines a PVC called "mypvc"
4. The PVC provisions a 50Gi volume
5. The volume will be provisioned via the "fast" StorageClass
6. The file defines the "fast" StorageClass
7. The StorageClass provisions volumes via the pd.csi.storage.gke.io CSI plugin
8. The CSI plugin will provision fast (pd-ssd) storage from the Google Cloud's storage backend

The relationship is hierarchical: the Pod references the PVC, the PVC references the StorageClass, and the StorageClass defines how and where to provision the actual storage.

### ❓ What does Figure 11.3 illustrate about the complete storage provisioning workflow and the missing component that would utilize the PVC?
Figure 11.3 illustrates how everything fits together in the storage provisioning workflow, showing the relationships between the StorageClass, PersistentVolumeClaim, PersistentVolume, and the backend storage. The missing component is a Pod on the right that would use the PVC to gain access to the PV.

![Figure 11.3 - How everything fits together](media/figure11-3.png)

### ❓ What two actions does the StorageClass controller perform when it detects a new PVC?
Whenever it sees a new PVC, it:

- creates the requested volume on the external system
- maps it to a new PV on Kubernetes

