# Scoped Resource Types

## Content

### ❓ Analyze the following kubectl api-resources output to identify which objects are namespaced and which are cluster-scoped, and explain the implications of this distinction.
```bash
$ kubectl api-resources
NAME                     SHORTNAMES   ...    NAMESPACED   KIND
nodes                    no                  false        Node
persistentvolumeclaims   pvc                 true         PersistentVolumeClaim
persistentvolumes        pv                  false        PersistentVolume
pods                     po                  true         Pod
podtemplates                                 true         PodTemplate
replicationcontrollers   rc                  true         ReplicationController
resourcequotas           quota               true         ResourceQuota
secrets                                      true         Secret
serviceaccounts          sa                  true         ServiceAccount
services                 svc                 true         Service
<Snip>
```

Most objects are namespaced, meaning you can deploy them to specific namespaces with custom policies and quotas. Objects that aren't namespaced, such as Nodes and PersistentVolumes, are cluster-scoped and you cannot deploy them to specific Namespaces.

### ❓ Based on the table, which resources are cluster-scoped versus namespaced, and what does this distinction mean?
The NAMESPACED column in the kubectl api-resources output indicates whether each resource is cluster-scoped (false) or namespaced (true).

**Cluster-scoped resources** (false):
- namespaces (ns)
- nodes (no) 
- storageclasses (sc)

**Namespaced resources** (true):
- pods (po)
- deployments (deploy)
- replicasets (rs)
- statefulsets (sts)
- cronjobs (cj)
- jobs
- horizontalpodautoscalers (hpa)
- ingresses (ing)
- networkpolicies (netpol)

This distinction means that cluster-scoped resources exist at the cluster level and are not confined to any specific namespace, while namespaced resources are created within and limited to a specific namespace.

```bash
$ kubectl api-resources
NAME                       SHORT    APIVERSION            NAMESPACED   KIND
namespaces                 ns       v1                    false        Namespace
nodes                      no       v1                    false        Node
pods                       po       v1                    true         Pod
deployments                deploy   apps/v1               true         Deployment
replicasets                rs       apps/v1               true         ReplicaSet
statefulsets               sts      apps/v1               true         StatefulSet
cronjobs                   cj       batch/v1              true         CronJob
jobs                                batch/v1              true         Job
horizontalpodautoscalers   hpa      autoscaling/v2        true         HorizontalPodAutoscaler
ingresses                  ing      networking.k8s.io/v1  true         Ingress
networkpolicies            netpol   networking.k8s.io/v1  true         NetworkPolicy
storageclasses             sc       storage.k8s.io/v1     false        StorageClass
```

