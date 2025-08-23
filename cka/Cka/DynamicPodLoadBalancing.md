# Dynamic Pod Load Balancing

## Content

### ❓ Analyze the output of the curl command to the ent service on port 8080, and explain how the response confirms connection to the correct namespace instance.
The output of the curl command to the ent service is:

```bash
# curl ent:8080
Hello from the DEV Namespace!
Hostname: enterprise-76fc64bd9-lvzsn
```

The response confirms connection to the correct namespace instance because:
1. The message "Hello from the DEV Namespace!" explicitly identifies that the connection reached the instance in the dev Namespace
2. The hostname "enterprise-76fc64bd9-lvzsn" shows which specific pod handled the request
3. The version of the app in each Namespace returns a different message, so this specific response proves it reached the dev instance

### ❓ Explain what the following full curl command does and interpret its output in the context of Kubernetes service discovery.
The curl command `curl ent.prod.svc.cluster.local:8080` demonstrates service discovery across Namespaces. This command connects to the 'enterprise' service in the 'prod' Namespace using its fully qualified domain name (FQDN). The output shows:

```bash
# curl ent.prod.svc.cluster.local:8080
Hello from the PROD Namespace!
Hostname: enterprise-5cfcd578d7-nvzlp
```

This confirms successful DNS resolution and connection to a Pod in the prod Namespace, with the response coming from a specific Pod identified by its hostname.

