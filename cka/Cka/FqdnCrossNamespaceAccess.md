# FQDN Cross-Namespace Access

## Content

### ❓ What method is used to connect to a service in a different namespace, and how does the DNS resolution differ from connecting to a service in the same namespace?
To connect to a service in a different namespace, you append the domain name of the target namespace to the service name. For example, to connect to the ent Service in the prod namespace from the dev namespace, you would use the fully qualified domain name format.

This differs from connecting to a service in the same namespace because:
- Same namespace: You can use the short name (e.g., "ent:8080") and the search domains in resolv.conf automatically append the current namespace domain
- Different namespace: You must use the fully qualified domain name (e.g., "ent.prod.svc.cluster.local:8080") or the cluster DNS won't be able to resolve the service in the remote namespace

This will cause the cluster DNS to return the ClusterIP of the Service in the prod Namespace instead of the local dev namespace.

