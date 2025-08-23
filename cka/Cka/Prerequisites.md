# Prerequisites

## Content

### ❓ What prerequisite does the StorageClass workflow assume about your Kubernetes cluster infrastructure?
The list assumes you have an external storage system connected to your Kubernetes cluster, meaning the storage infrastructure must already be available and accessible to the cluster.

### ❓ What are the two specific requirements mentioned in the paragraph that must be met for the PVC creation command to work successfully?
The two specific requirements for the PVC creation command to work successfully are:

1. You must run the command from the storage folder (where the YAML file is located)
2. You must have an LKE (Linode Kubernetes Engine) cluster with a linode-block-storage storage class available

### ❓ What specific conditions must be met for the command to create the PVC to work successfully?
The command to create the PVC will only work successfully if you meet two specific conditions: 1) You must run the command from the storage folder, and 2) You must have an LKE cluster with a linode-block-storage storage class.

