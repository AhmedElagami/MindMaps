# Locality Constraints

## Content

### ❓ What are the two main types of restrictions that apply when using external storage providers with Kubernetes clusters?
The two main types of restrictions that apply when using external storage providers with Kubernetes clusters are:

1. **Cloud provider compatibility restrictions** - For example, you can't provision and mount AWS EBS volumes if your cluster is on Microsoft Azure

2. **Locality restrictions** - Pods may have to be in the same region or zone as the storage they're accessing

### ❓ What type of restrictions may apply to Pods accessing storage according to the content, and provide a specific example?
Locality restrictions may apply to Pods accessing storage. For example, Pods may have to be in the same region or zone as the storage they're accessing.

