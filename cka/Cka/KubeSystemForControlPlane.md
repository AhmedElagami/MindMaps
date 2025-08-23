# kube-system for control plane

## Content

### ❓ What does the output of the kubectl get svc command with the --namespace flag reveal about the services running in the kube-system Namespace?
The `kubectl get svc --namespace kube-system` command reveals the Service objects running in the kube-system Namespace, which typically include critical control plane components:

```bash
$ kubectl get svc --namespace kube-system
NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                 
kube-dns             ClusterIP      10.43.0.10     <none>        53/UDP,53/TCP,9153...   
metrics-server       ClusterIP      10.43.4.203    <none>        443/TCP                 
traefik-prometheus   ClusterIP      10.43.49.213   <none>        9100/TCP                
traefik              LoadBalancer   10.43.222.75   <pending>     80:31716/TCP,443:31...
```

This output shows services like:
- **kube-dns**: Internal DNS service (ClusterIP)
- **metrics-server**: Metrics collection service (ClusterIP) 
- **traefik-prometheus**: Monitoring service (ClusterIP)
- **traefik**: Load balancer service (LoadBalancer)

Note: Your output might be different depending on your cluster configuration.

