# kube-dns Service Configuration

## Content

### ❓ What does the following kubectl command output demonstrate about the relationship between the nameserver IP in resolv.conf and the cluster DNS service?
The kubectl command output demonstrates that the nameserver IP configured in the container's resolv.conf file (10.96.0.10) exactly matches the ClusterIP of the kube-dns Service, proving that containers are configured to use the cluster DNS for service discovery:

```bash
$ kubectl get svc -n kube-system -l k8s-app=kube-dns
NAME       TYPE        CLUSTER-IP      PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10      53/UDP,53/TCP,9153/TCP   13d
```

This confirms that:
1. The kube-dns Service has ClusterIP 10.96.0.10
2. Containers are configured to send DNS queries to this exact IP address
3. The cluster DNS runs on ports 53/UDP, 53/TCP, and 9153/TCP for DNS resolution

The nameserver IP in resolv.conf is not arbitrary - it's specifically the ClusterIP of the kube-dns Service that handles all internal service discovery within the cluster.

### ❓ Why is it critical that the kube-dns Service ClusterIP matches the IP address in container resolv.conf files? Analyze the following service output to verify the kube-dns Service configuration and listening ports.
It is critical that the kube-dns Service ClusterIP matches the IP address in the resolv.conf files of all containers on the cluster because if it doesn't, containers will send DNS requests to the wrong place.

The kube-dns Service configuration shows:

```bash
$ kubectl get svc kube-dns -n kube-system
NAME       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10    <none>        53/UDP,53/TCP,9153/TCP   14d
```

This output verifies:
- The service is a ClusterIP type with IP 10.96.0.10
- It's listening on ports 53/UDP and 53/TCP for DNS services
- Additional port 9153/TCP is also open
- The service has been running for 14 days
- This ClusterIP (10.96.0.10) must match what's in all container resolv.conf files

