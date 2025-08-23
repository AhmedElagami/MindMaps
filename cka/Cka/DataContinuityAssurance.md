# Data Continuity Assurance

## Content

### ❓ What is the operational advantage of PVCs persisting after StatefulSet scale-down operations?
The fact all three PVCs still exist means that scaling back up to 3 replicas will only require a new Pod. The StatefulSet controller will create the new Pod and connect it to the existing PVC, preserving the data and avoiding the need to create new storage volumes.

