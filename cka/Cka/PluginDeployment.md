# Plugin Deployment

## Content

### ❓ What are the typical distribution methods for CSI plugins and where do they run once installed in a Kubernetes cluster?
The provider usually distributes the CSI plugin via a Helm chart or YAML installer. Once installed, the plugin runs as a set of Pods in the kube-system Namespace.

### ❓ What is the responsibility of the administrator regarding CSI plugin features and configuration?
Not all backends support the same features, and it's your responsibility to read the plugin's documentation and configure it properly.

### ❓ How do most cloud platforms handle CSI plugin installation for their native storage services, and describe the deployment method for third-party storage system CSI plugins?
Most cloud platforms pre-install CSI plugins for the cloud's native storage services. You'll have to install plugins for third-party storage systems manually, but as previously stated, they're usually available as Helm charts or YAML files from the provider.

### ❓ What role does the 'provisioner' field play in a StorageClass and what prerequisite does it require?
The `provisioner` field tells Kubernetes which CSI plugin to use. You'll need the plugin installed for it to work properly.

### ❓ How are CSI plugins typically deployed in most hosted Kubernetes clusters?
Most hosted Kubernetes clusters pre-install CSI plugins that run as Pods in the Namespace.

