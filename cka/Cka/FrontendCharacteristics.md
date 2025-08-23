# Frontend Characteristics

## Content

### ❓ Explain what the following curl command output indicates about the application running in the shield Namespace.
The curl command output indicates that the application running in the shield Namespace is a web application that returns HTML content. The output shows:

```bash
$ curl localhost:8080

<!DOCTYPE html>
<html>
<head>
    <title>AOS</title>
    <Snip>
```

This demonstrates that:
1. The service is accessible and responding to HTTP requests
2. The application serves HTML content with a proper DOCTYPE declaration
3. The page title is "AOS" (likely an acronym or application name)
4. The application is functioning correctly and serving web content

### ❓ How does Kubernetes handle service name uniqueness and IP assignment when a developer deploys an app behind a Service?
Kubernetes ensures the Service name is unique and automatically assigns it an IP address (ClusterIP). It also registers the name and IP in the cluster DNS.

