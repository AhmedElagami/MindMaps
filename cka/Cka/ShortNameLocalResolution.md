# Short Name Local Resolution

## Content

### ❓ What is an unqualified name and how does Kubernetes handle it using search domains?
An unqualified name is a short name such as "ent" or "cer" that is not a fully qualified domain name (FQDN). Kubernetes handles unqualified names by adding search domains to the container's resolv.conf file that get appended to these short names. This converts them into fully qualified domain names (FQDNs) that can be properly resolved by the cluster DNS. For example, the short name "cer" might be converted to "cer.default.svc.cluster.local" by appending the search domain.

