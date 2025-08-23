# CoreDNS Architecture

## Content

### ❓ What is the primary function of a service registry in Kubernetes and what component serves this role?
The primary function of a service registry in Kubernetes is to maintain a list of Service names and their associated IP addresses. Every Kubernetes cluster has a built-in cluster DNS that serves this role as its service registry.

### ❓ Describe the architecture of the cluster DNS service registry, including its components and how they're organized.
The cluster DNS service registry is a Kubernetes-native application running on the control plane of every Kubernetes cluster as two or more Pods managed by a Deployment and fronted by its own Service. The Deployment is usually called coredns or kube-dns, and the Service is always called kube-dns. It maps every Service's name and IP, and runs on the control plane as a set of Pods managed by a Deployment and fronted by a Service.

### ❓ Explain what the following command sequence does to list the cluster DNS Pods and interpret the output, and explain what the following command sequence does to show the cluster DNS Deployment and interpret the output.
The first command lists the Pods running the cluster DNS:

```bash
$ kubectl get pods -n kube-system -l k8s-app=kube-dns
NAME                       READY   STATUS    RESTARTS   AGE
coredns-76f75df574-d6nn5   1/1     Running   0          13d
coredns-76f75df574-n7qzk   1/1     Running   0          13d
```

This output shows two coredns Pods that are both running and ready (1/1), with no recent restarts, and have been running for 13 days. They normally use the coredns image, but some clusters may use a different image and call the Pods and Deployment kube-dns instead of coredns.

The second command shows the Deployment that manages the cluster DNS Pods:

```bash
$ kubectl get deploy -n kube-system -l k8s-app=kube-dns
NAME        READY     UP-TO-DATE     AVAILABLE     AGE
coredns     2/2       2              2             13d
```

This output shows that the coredns Deployment has 2 out of 2 desired Pods ready, 2 Pods are up-to-date with the current configuration, 2 Pods are available, and the Deployment has been running for 13 days. The Deployment ensures there is always the correct number of cluster DNS Pods running.

### ❓ What is the primary function of the cluster DNS Pods as described in the paragraph?
The cluster DNS Pods ensure there is always the correct number of cluster DNS Pods running in the Kubernetes cluster.

### ❓ What does the final command mentioned in the paragraph show about the cluster DNS architecture?
The final command shows the Service in front of the cluster DNS Pods, which provides a stable endpoint for the cluster DNS service.

### ❓ What is consistent and what varies about the kube-dns service across different Kubernetes clusters?
The kube-dns service is always called kube-dns, but it gets a different IP address on each cluster.

### ❓ What is the fundamental role of the internal cluster DNS service in Kubernetes service discovery?
Every Kubernetes cluster runs an internal cluster DNS service that it uses as its service registry.

### ❓ Describe the three components involved in running the cluster DNS service on the control plane.
The cluster DNS service runs on the control plane as a set of Pods managed by a Deployment and fronted by a Service.

### ❓ Analyze the following full kubectl command output to verify the CoreDNS deployment status and pod availability.
The kubectl commands verify that the CoreDNS deployment and its Pods are running properly:

```bash
$ kubectl get deploy -n kube-system -l k8s-app=kube-dns
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
coredns   2/2     2            2           14d

$ kubectl get pods -n kube-system -l k8s-app=kube-dns
NAME                       READY   STATUS    RESTARTS   AGE
coredns-76f75df574-6q7k7   1/1     Running   0          14d
coredns-76f75df574-krnr7   1/1     Running   0          14d
```

This output shows:
- The coredns Deployment has 2/2 replicas ready
- Both replicas are up-to-date and available
- Two coredns Pods are running with no recent restarts
- The Pods have been running for 14 days without issues

### ❓ What should you look for in the logs to confirm a CoreDNS pod is functioning properly? Explain the output of the following kubectl logs command and what it indicates about CoreDNS operation.
To confirm a CoreDNS pod is functioning properly, you should look for typical working DNS Pod output in the logs. The following output is typical of a working DNS Pod:

```bash
$ kubectl logs coredns-76f75df574-n7qzk -n kube-system
.:53
[INFO] plugin/reload: Running configuration SHA512 = 591cf328cccc12b...
CoreDNS-1.11.1
linux/arm64, go1.20.7, ae2bbc2
```

This indicates:
- The CoreDNS server is listening on port 53
- The reload plugin is running with a valid configuration SHA
- CoreDNS version 1.11.1 is operational
- The platform is linux/arm64 with Go version 1.20.7
- No error messages are present, indicating proper functionality

### ❓ What is the recommended approach for troubleshooting after verifying the fundamental DNS components are working?
Once you've verified the fundamental DNS components are up and working, you can perform more detailed and in-depth troubleshooting.

### ❓ What type of tools should be included in a troubleshooting Pod for DNS diagnostics?
Start a troubleshooting Pod with your favorite networking tools installed (ping, traceroute, curl, dig, nslookup, etc.).

### ❓ Explain the purpose and structure of the following kubectl command for starting a DNS troubleshooting Pod.
The following command starts a new standalone Pod called dnsutils and will connect your terminal. It's based on the image just mentioned and may take a few seconds to start:

```bash
$ kubectl run -it dnsutils \
  --image registry.k8s.io/e2e-test-images/jessie-dnsutils:1.7
```

This command creates an interactive Pod with DNS troubleshooting tools installed, using the official Kubernetes test image that contains utilities like dig and nslookup.

### ❓ What is the common method described for testing cluster DNS functionality and what service runs on every cluster to expose the API server to all Pods?
A common way to test if the cluster DNS is working is to use nslookup to resolve the kubernetes Service. This service runs on every cluster and exposes the API server to all Pods.

### ❓ Analyze the nslookup output below to identify the DNS server IP, service FQDN, and ClusterIP address and explain what information should appear in the first two lines versus the last two lines of a successful nslookup output.
The nslookup output provides detailed DNS resolution information:

```bash
# nslookup kubernetes
Server:    10.96.0.10
Address:   10.96.0.10#53
Name:      kubernetes.default.svc.cluster.local
Address:   10.96.0.1
```

Analysis:
- **First two lines**: Should show the IP address of your cluster DNS (Server: 10.96.0.10)
- **Last two lines**: Should show the FQDN of the kubernetes Service (kubernetes.default.svc.cluster.local) and its ClusterIP (10.96.0.1)

The query returns the name and its IP address, with the first two lines showing the DNS server IP and the last two showing the service FQDN and ClusterIP.

### ❓ What type of error indicates that DNS resolution is not working properly and what is one possible solution?
Errors such as "nslookup: can't resolve kubernetes" are indicators that DNS isn't working. A possible solution is to delete the coredns Pods, which will cause the coredns Deployment to recreate them.

### ❓ Explain what the following kubectl delete command accomplishes and what happens as a result.
The following command deletes the DNS Pods:

```bash
$ kubectl delete pod -n kube-system -l k8s-app=kube-dns
pod "coredns-76f75df574-d6nn5" deleted
pod "coredns-76f75df574-n7qzk" deleted
```

This command accomplishes the deletion of all coredns Pods (identified by the label k8s-app=kube-dns) in the kube-system namespace. As a result, the coredns Deployment will automatically recreate new Pods, which can help resolve DNS issues when the existing Pods are malfunctioning.

### ❓ What cleanup commands are provided to remove the DNS troubleshooting resources?
Run the following commands to clean up:

```bash
$ kubectl delete pod dnsutils

$ kubectl delete -f sd-example.yml
```

These commands remove the dnsutils troubleshooting Pod and any resources created from the sd-example.yml manifest file.

### ❓ What is the primary function of Kubernetes' internal cluster DNS according to the chapter summary and how does it interact with the API server?
Kubernetes uses the internal cluster DNS for service registration and service discovery. It's a Kubernetes-native application that watches the API server for newly created Service objects and automatically registers their names and IPs.

