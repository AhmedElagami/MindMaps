# Persistent Volume Subsystem

## Content

### ❓ What are the three core API objects in the Kubernetes persistent volume subsystem and what are their primary functions?
The three core API objects in the Kubernetes persistent volume subsystem are:

1. **PersistentVolumes (PV)** - These map to external volumes and make external volumes available on Kubernetes
2. **PersistentVolumeClaims (PVC)** - These grant access to PVs; if a Pod wants to use a PV, it needs a PVC granting it access
3. **StorageClasses (SC)** - These make the provisioning process automatic and dynamic by allowing applications to create PVs and backend volumes dynamically

As summarized in the content: "PVs map to external volumes, PVCs grant access to PVs, and SCs make it all automatic and dynamic."

### ❓ What are the three main API objects that comprise the Kubernetes Persistent Volume Subsystem?
The Kubernetes Persistent Volume Subsystem has the following three main API objects:
- PersistentVolumes (PV)
- PersistentVolumeClaims (PVC)
- StorageClasses (SC)

### ❓ Explain the relationship between PVs, PVCs, and Pods in the storage access process and what is the function of StorageClasses in the persistent volume subsystem?
PVs make external volumes available on Kubernetes. If a Pod wants to use a PV, it needs a PVC granting it access to the PV. SCs allow applications to create PVs and backend volumes dynamically.

### ❓ What three terms are used interchangeably when referring to storage provisioning components?
We sometimes use the terms provisioner, plugin, and driver interchangeably when referring to storage provisioning components.

