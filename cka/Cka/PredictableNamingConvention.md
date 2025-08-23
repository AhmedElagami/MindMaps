# Predictable Naming Convention

## Content

### ❓ How does Kubernetes use the StatefulSet name 'tkb-sts' as the base for naming replicas and volumes, and what is the process for creating the 3 replicas including order and readiness requirements?
Kubernetes uses the StatefulSet name "tkb-sts" as the base for naming both replicas and volumes. The StatefulSet name becomes the foundation for every replica and volume name that gets created.

For the replica creation process with 3 replicas specified in the replicas field, Kubernetes will create them in sequential order as tkb-sts-0, tkb-sts-1, and tkb-sts-2. The creation follows a strict ordered process where Kubernetes waits for each replica to be running and ready before starting the next one. This ensures predictable deployment order and proper initialization of stateful applications.

