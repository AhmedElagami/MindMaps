# PVC Naming Convention

## Content

### ❓ What naming convention pattern is used for PVCs created by a StatefulSet volume claim template, and what components are included in each name?
The PVC naming convention includes three components in each name: the volume claim template name, the StatefulSet name, and the associated Pod replica number. The pattern is:

volumeClaimTemplate name + StatefulSet name + Pod replica number

For example:
- webroot (volumeClaimTemplate) + tkb-sts (StatefulSet) + 0 (Pod replica) = webroot-tkb-sts-0
- webroot (volumeClaimTemplate) + tkb-sts (StatefulSet) + 1 (Pod replica) = webroot-tkb-sts-1  
- webroot (volumeClaimTemplate) + tkb-sts (StatefulSet) + 2 (Pod replica) = webroot-tkb-sts-2

