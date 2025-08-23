# Storage Configuration

## Content

### ❓ What does the following YAML volume claim template define in terms of storage requirements and access, and explain each configuration element?
The YAML volume claim template defines a claim template called 'webroot' that requests 10GB volumes from the 'block' StorageClass with ReadWriteOnce access mode.

```bash
volumeClaimTemplates:
- metadata:
    name: webroot
  spec:
    accessModes: [ "ReadWriteOnce" ]
    storageClassName: "block"
    resources:
      requests:
        storage: 10Gi
```

Configuration elements:
- **metadata.name**: 'webroot' - the name of the volume claim template
- **spec.accessModes**: ["ReadWriteOnce"] - the volume can be mounted as read-write by a single node
- **spec.storageClassName**: "block" - specifies the StorageClass to use for provisioning
- **spec.resources.requests.storage**: 10Gi - requests 10 gibibytes of storage capacity

