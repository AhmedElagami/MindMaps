# Pod Health Tracking

## Content

### ❓ How do Services maintain an up-to-date list of healthy Pods despite scaling operations and failures?
Services are intelligent enough to maintain a list of healthy Pods with matching labels. This means you can scale up and down, perform rollouts and rollbacks, and Pods can even fail, but the Service will always have an up-to-date list of active healthy Pods. This is achieved through automatic EndpointSlice creation and management.

### ❓ What is the significance of the Endpoints field in the kubectl describe output, and where does this information originate from?
The Endpoints field in the `kubectl describe` output is the list of healthy matching Pods from the Service's EndpointSlice object. This information originates from Kubernetes' automatic monitoring and maintenance of Pod health status. When a Service has a label selector, Kubernetes continuously watches for Pods that match those labels and updates the corresponding EndpointSlice objects with the IP addresses of healthy Pods. The Endpoints field shows these actual backend Pod IP addresses (like 10.1.0.200:8080, 10.1.0.201:8080, etc.) that will receive traffic from the Service, ensuring that traffic is only routed to currently healthy and available application instances.

### ❓ What is the relationship between deleting a Service and the automatic cleanup of associated Endpoints and EndpointSlices in Kubernetes, and what does the full command sequence do when cleaning up Kubernetes resources?
When you delete a Service in Kubernetes, the system automatically deletes the associated Endpoints and EndpointSlice objects. This cleanup process is handled by Kubernetes controllers that manage the lifecycle of these dependent objects. The relationship is that Endpoints and EndpointSlices are tightly coupled with Services - they exist to support Service functionality and are automatically managed throughout the Service lifecycle.

The full cleanup command sequence:

```bash
$ kubectl delete -f deploy.yml -f lb.yml
deployment.apps "svc-lb" deleted
service "svc-lb" deleted
```

This command specifically removes:
1. The Deployment created from deploy.yml (which manages the Pods)
2. The Service created from lb.yml

When the Service is deleted, Kubernetes automatically deletes the associated Endpoints and EndpointSlices, completing the cleanup of all related networking resources.

### ❓ What role do EndpointSlice objects play in the service registration process?
Associated EndpointSlice objects are created to hold the list of healthy Pod IPs that match the Service's label selector.

