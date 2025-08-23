# Cloud Provider Provisioners

## Content

### ❓ Based on the paragraph, what are the three types of storage provisioners found in the Autopilot regional cluster on Google Kubernetes Engine, and what storage technologies do they provide access to?
The three types of storage provisioners found in the Autopilot regional cluster on Google Kubernetes Engine are:

1. **filestore.csi.storage.gke.io** - Provides access to Google Cloud's NFS-based Filestore storage
2. **pd.csi.storage.gke.io** - Provides access to Google Cloud's block storage
3. **kubernetes.io/gce-pd** - A legacy in-tree plugin (non-CSI) that also provides access to Google Cloud storage

