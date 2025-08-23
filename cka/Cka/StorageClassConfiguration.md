# Storage Class Configuration

## Content

### ❓ List the various storage configuration options that are configured via the backend's CSI plugin.
The following storage configuration options are configured via the backend's CSI plugin:
- volume size
- protection level
- snapshot schedule
- replication settings
- encryption configuration
- and more

### ❓ Analyze the mapping between external storage tiers and Kubernetes StorageClasses as shown in the content.
The mapping between external storage tiers and Kubernetes StorageClasses is as follows:

| External tier     | Kubernetes StorageClass name | CSI plugin     |
|-------------------|------------------------------|----------------|
| Flash             | sc-fast-block                | csi.xyz.block  |
| Flash encrypted   | sc-fast-encrpted             | csi.xyz.block  |
| Mechanical disk   | sc-slow                      | csi.xyz.block  |
| NFS Filestore     | sc-file                      | csi.xyz.file   |

### ❓ Interpret the full StorageClass YAML definition below and explain how each component contributes to provisioning encrypted SSD volumes in the Ireland AWS region.
The following YAML defines a StorageClass called `fast-local` that provisions encrypted SSD volumes capable of 10 IOPs per gigabyte from the Ireland AWS region:

```bash
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: fast-local
provisioner: ebs.csi.aws.com         <<---- AWS Elastic Block Store CSI plugin
parameters:
  encrypted: true                    <<---- Create encrypted volumes
  type: io1                          <<---- AWS SSD drives
  iopsPerGB: "10"                    <<---- Performance requirement
allowedTopologies:                   <<---- Where to provision volumes and replicas
- matchLabelExpressions:             
  - key: topology.ebs.csi.aws.com/zone
    values:
    - eu-west-1a                     <<---- Ireland AWS region
```

Each component contributes as follows:
- **`kind` and `apiVersion`**: Tell Kubernetes the type and version of the object being defined
- **`metadata.name`**: Gives the object a friendly name (`fast-local`) that is used to reference the StorageClass
- **`provisioner`**: Specifies which CSI plugin to use (AWS Elastic Block Store CSI plugin)
- **`parameters` block**: Defines the type of storage to provision with plugin-specific values:
  - `encrypted: true`: Creates encrypted volumes
  - `type: io1`: Specifies AWS SSD drives
  - `iopsPerGB: "10"`: Sets performance requirement of 10 IOPs per gigabyte
- **`allowedTopologies`**: Specifies where to provision volumes and replicas, targeting the Ireland AWS region (`eu-west-1a`)

### ❓ How does the 'metadata.name' field function in a StorageClass object and why is it important?
The `metadata.name` is an arbitrary string that gives the object a friendly name. It should be meaningful, as it's how you and other objects refer to the class.

### ❓ What does the 'parameters' block define in a StorageClass configuration?
The `parameters` block defines the type of storage to provision. It contains plugin-specific values and is different for every plugin.

### ❓ How does the 'allowedTopologies' property function in storage provisioning?
The `allowedTopologies` property lets you specify where replicas should go, controlling the geographic or topological placement of storage volumes.

