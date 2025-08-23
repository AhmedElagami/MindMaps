# Persistent Volume Claims

## Content

### ❓ Based on the paragraph, what specific PVC is being created, what storage class does it use, and what is the capacity of the volume being provisioned?
The PVC being created is called `pvc-test`. It uses the `linode-block-storage` storage class and provisions a volume with a capacity of 10GB.

### ❓ Analyze the full YAML specification below and explain the purpose of each field in defining a PersistentVolumeClaim that uses the linode-block-storage storage class with 10GB capacity.
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-test
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: linode-block-storage
  resources:
    requests:
      storage: 10Gi
```

**Field Analysis:**

- **apiVersion: v1**: Specifies the Kubernetes API version to use for this resource
- **kind: PersistentVolumeClaim**: Defines the type of resource being created (a PVC)
- **metadata.name: pvc-test**: Sets the name of the PVC to "pvc-test" for identification
- **spec.accessModes: ReadWriteOnce**: Specifies the volume can be mounted as read-write by a single node
- **spec.storageClassName: linode-block-storage**: Identifies which storage class to use for provisioning
- **spec.resources.requests.storage: 10Gi**: Requests 10 gibibytes of storage capacity for the volume

### ❓ Explain what the following full command sequence does to create the PVC and what output indicates success.
The command `kubectl apply -f lke-pvc-test.yml` creates a PersistentVolumeClaim (PVC) using the configuration defined in the lke-pvc-test.yml file. The output `persistentvolumeclaim/pvc-test created` indicates that the PVC was successfully created in the Kubernetes cluster.

```bash
$ kubectl apply -f lke-pvc-test.yml
persistentvolumeclaim/pvc-test created
```

### ❓ What does the output of the 'kubectl get pvc' command reveal about the status, volume binding, and storage class of the pvc-test PVC?
The output of `kubectl get pvc` reveals that the pvc-test PVC has a STATUS of 'Bound', indicating it has been successfully bound to a volume. It shows the PVC is associated with volume 'pvc-cc3dd8716c7d46c9' with a capacity of 10Gi, uses ReadWriteOnce (RWO) access mode, and is provisioned through the 'linode-block-storage' storage class.

```bash
$ kubectl get pvc
NAME       STATUS   VOLUME                 CAPACITY   ACCESS MODES   STORAGECLASS           
pvc-test   Bound    pvc-cc3dd8716c7d46c9   10Gi       RWO            linode-block-storage
```

### ❓ Explain what the following command does and what output confirms successful deletion of the PVC.
The command `kubectl delete pvc pvc-test` deletes the PersistentVolumeClaim named 'pvc-test' from the Kubernetes cluster. The output `persistentvolumeclaim "pvc-test" deleted` confirms that the PVC was successfully deleted.

```bash
$ kubectl delete pvc pvc-test
persistentvolumeclaim "pvc-test" deleted
```

### ❓ What specific PVC configuration is defined in the YAML block, including its name, storage request, and storage class reference?
The YAML block defines a PersistentVolumeClaim (PVC) with the following configuration:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-wait-keep
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: block-wait-keep
```

Specific configuration details:
- **Name**: `pvc-wait-keep`
- **Storage request**: `20Gi` (20 gigabytes)
- **Storage class reference**: `block-wait-keep`
- **Access mode**: `ReadWriteOnce`
- **API version**: `v1`
- **Kind**: `PersistentVolumeClaim`

### ❓ Analyze the output of the kubectl commands and what specific information can you gather about the PVC binding, PV creation, storage configuration, and their relationship?
The kubectl command output provides detailed information about the PVC binding, PV creation, and storage configuration:

```bash
$ kubectl apply -f lke-app.yml
pod/volpod created

$ kubectl get pvc
NAME            STATUS   VOLUME                 CAPACITY   ACCESS MODES   STORAGECLASS   
pvc-wait-keep   Bound    pvc-279f09e083254fa9   20Gi       RWO            block-wait-keep

$ kubectl get pv
                              ACCESS  RECLAIM
NAME                   CAPACITY   MODES   POLICY   STATUS   CLAIM           STORAGECLASS     
pvc-279f09e083254fa9   20Gi       RWO     Retain   Bound    pvc-wait-keep   block-wait-keep
```

**Key Information from the Output:**

1. **PVC Binding Status:** The PVC `pvc-wait-keep` shows status "Bound" and is associated with volume `pvc-279f09e083254fa9`

2. **PV Creation Details:** A PersistentVolume was automatically created with the name `pvc-279f09e083254fa9`

3. **Storage Configuration:**
   - **Capacity:** 20Gi
   - **Access Mode:** RWO (ReadWriteOnce)
   - **Reclaim Policy:** Retain
   - **Storage Class:** block-wait-keep

4. **Relationship:** The PV is explicitly bound to the PVC `pvc-wait-keep` as shown in the "CLAIM" column, confirming the direct relationship between the PVC and PV

5. **Storage Class:** Both the PVC and PV are using the same storage class `block-wait-keep`, ensuring consistency in storage provisioning

The output confirms that the storage provisioning workflow is working correctly: the PVC requested storage, the storage class provisioned a PV with the requested specifications, and the two are properly bound together.

### ❓ Based on the paragraph, what are the three key indicators that confirm the PVC and PV are properly configured and bound?
According to the paragraph content, the three key indicators that confirm the PVC and PV are properly configured and bound are:

1. **The PVC is bound to a volume** - This shows the PersistentVolumeClaim has successfully obtained storage resources
2. **A PV exists with the same name** - This indicates the PersistentVolume was created and is available
3. **All the settings are as expected** - This includes the capacity, reclaim policy, and storage class matching the requested configuration

These three indicators demonstrate that the storage provisioning process completed successfully and the PVC-PV binding is functioning correctly with all specified parameters in place.

