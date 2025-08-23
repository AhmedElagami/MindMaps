# Controller PVC Monitoring

## Content

### ❓ Describe how the StorageClass controller operates and what it monitors on the API server.
The StorageClass controller operates as a background reconciliation loop on the control plane, watching the API server for new PVCs.

