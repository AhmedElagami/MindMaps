# Endpoint Components

## Content

### ❓ What should the kube-dns EndpointSlice object contain according to the paragraph?
The associated kube-dns EndpointSlice object should be up and have the IP addresses of the coredns Pods listening on port.

### ❓ What information should the kube-dns EndpointSlice contain to indicate proper DNS functionality?
The associated kube-dns EndpointSlice object should also be up and have the IP addresses of the coredns Pods listening on port. This indicates that the EndpointSlice properly maps the kube-dns Service to the actual CoreDNS Pod endpoints.

### ❓ Explain what the following kubectl command output reveals about the kube-dns EndpointSlice configuration.
The kubectl command output shows the detailed configuration of the kube-dns EndpointSlice:

```bash
$ kubectl get endpointslice -n kube-system -l k8s-app=kube-dns
NAME             ADDRESSTYPE   PORTS        ENDPOINTS                AGE
kube-dns-jb72g   IPv4          9153,53,53   10.244.1.9,10.244.1.14   14d
```

This reveals:
- The EndpointSlice name: kube-dns-jb72g
- Address type: IPv4
- Ports exposed: 9153, 53, 53 (showing both TCP and UDP port 53)
- Endpoint IP addresses: 10.244.1.9 and 10.244.1.14 (the IPs of the coredns Pods)
- Age: 14 days old

