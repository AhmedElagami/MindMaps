# Ingress Components

## Content

### ❓ What are the two required constructs for implementing Ingress in Kubernetes and how does Ingress differ from most other Kubernetes resources in terms of built-in controllers?
Ingress requires two constructs: a resource and a controller. The resource defines the routing rules, and the controller implements them. However, Kubernetes doesn't have a built-in Ingress controller, meaning you need to install one. This differs from Deployments, ReplicaSets, Services, and most other resources that have built-in pre-configured controllers. However, some cloud platforms simplify this by allowing you to install one when you build the cluster.

### ❓ What are the seven main steps involved in the hands-on Ingress configuration process?
The seven main steps involved in the hands-on Ingress configuration process are:
1. Install the NGINX Ingress controller
2. Configure an Ingress class
3. Deploy a sample app
4. Configure an Ingress object
5. Inspect the Ingress object
6. Configure DNS name resolution
7. Test the Ingress

### ❓ What are the three essential components required to have a functioning Ingress setup according to the deployment checklist?
The three essential components required for a functioning Ingress setup are:
1. Two apps and associated ClusterIP Services
2. Load balancer (cloud-based or local implementation)
3. Ingress (controller and resource) configured to route traffic

### ❓ What cleanup commands are used to remove the Ingress resource, application components, Ingress controller, and DNS entries from the cluster?
The cleanup process involves several commands:

1. **Delete the Ingress resource:**
```bash
$ kubectl delete -f ig-all.yml
ingress.networking.k8s.io "mcu-all" deleted
```

2. **Delete the Pods and ClusterIP Services:**
```bash
$ kubectl delete -f app.yml
service "svc-shield" deleted
service "svc-hydra" deleted
pod "shield" deleted
pod "hydra" deleted
```
It may take a few seconds for the Pods to terminate gracefully.

3. **Delete the NGINX Ingress controller:**
```bash
$ kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/
controller-v1.12.0/deploy/static/provider/cloud/deploy.yaml

namespace "ingress-nginx" deleted
serviceaccount "ingress-nginx" deleted
<Snip>
```
This command can take about a minute to complete.

4. **Clean up DNS entries from /etc/hosts:**
```bash
$ sudo vi /etc/hosts

# Host Database
<Snip>
212.2.246.150 shield.mcu.com       <<---- Delete this entry
212.2.246.150 hydra.mcu.com        <<---- Delete this entry
212.2.246.150 mcu.com              <<---- Delete this entry
```
This removes the manual DNS entries added for the Ingress setup.

