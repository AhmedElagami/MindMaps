# Ingress Controllers

## Content

### ❓ Why does the author recommend using a cloud-based cluster instead of Docker Desktop for the hands-on Ingress section and what are the two essential prerequisites needed to follow along with the hands-on Ingress exercises?
The author recommends using a cloud-based cluster instead of Docker Desktop because "This section doesn't work with the multi-node Kubernetes cluster that ships with Docker Desktop v4.38 or earlier. It may work with future releases. I recommend you work with a cloud-based cluster such as the LKE cluster we show you how to build in Chapter 3."

The two essential prerequisites needed to follow along with the hands-on Ingress exercises are:
1. A Kubernetes cluster
2. A clone of the book's GitHub repo

### ❓ What does the following full command sequence do to prepare the Git repository for the hands-on Ingress lab?
The command sequence clones the book's GitHub repository and sets up the correct branch for the hands-on Ingress exercises:

```bash
$ git clone https://github.com/nigelpoulton/TKB.git
Cloning from...

$ cd TKB

$ git fetch origin

$ git checkout -b 2025 origin/2025
```

This sequence:
1. Clones the repository from the specified URL
2. Changes into the TKB directory
3. Fetches the latest changes from the origin remote
4. Creates and switches to a new local branch called "2025" that tracks the remote "origin/2025" branch

### ❓ What Kubernetes constructs are installed when deploying the NGINX Ingress controller from the YAML file?
When deploying the NGINX Ingress controller from the YAML file, it installs "a bunch of Kubernetes constructs, including a Namespace, ServiceAccounts, ConfigMaps, Roles, RoleBindings, and more."

### ❓ Explain what happens when you run the full command to install the NGINX Ingress controller from the Kubernetes GitHub repository.
When you run the command to install the NGINX Ingress controller, it deploys multiple Kubernetes constructs from a YAML file hosted in the official Kubernetes GitHub repository:

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/
controller-v1.12.0/deploy/static/provider/cloud/deploy.yaml

namespace/ingress-nginx created
serviceaccount/ingress-nginx created
<Snip>
```

The installation creates various Kubernetes resources including:
- A Namespace called "ingress-nginx"
- ServiceAccounts for the controller
- ConfigMaps for configuration
- Roles and RoleBindings for permissions
- And other necessary components for the NGINX Ingress controller to function properly

### ❓ What information does the output of the kubectl get pods command reveal about the status of the NGINX Ingress controller components?
The `kubectl get pods` command output shows the status of all NGINX Ingress controller components in the ingress-nginx namespace:

```bash
$ kubectl get pods -n ingress-nginx \
  -l app.kubernetes.io/name=ingress-nginx

NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-789md        0/1     Completed   0          25s
ingress-nginx-admission-patch-tc4cl         0/1     Completed   0          25s
ingress-nginx-controller-7445ddc6c4-csf98   0/1     Running     0          26s
```

The output reveals:
- Two "Completed" Pods (ingress-nginx-admission-create and ingress-nginx-admission-patch) that were short-lived initialization Pods
- One "Running" Pod (ingress-nginx-controller) which is the main controller component
- The READY column shows 0/1 for all Pods, indicating they're either completed their task or still initializing
- The STATUS column shows the current state of each Pod
- The AGE column shows how long each Pod has been running

### ❓ What factors affect the time it takes to get an ADDRESS when listing Ingress objects on a cloud cluster?
If your cluster is on a cloud, it can take a minute or so to get an ADDRESS while the cloud provisions the load balancer.

### ❓ What does the Address line in the Ingress describe output represent, and what might be different about this value on local clusters?
The Address line in the Ingress describe output represents the IP or DNS name of the load balancer created by the Ingress. This might be different on local clusters where the load balancer implementation may vary or use different addressing schemes compared to cloud environments.

### ❓ What does Figure 8.3 illustrate about the relationship between the Ingress controller and cloud infrastructure?
Figure 8.3 illustrates how the cloud back-end load balancer configuration looks when your cluster is on Google Kubernetes Engine (GKE), showing the integration between the Ingress controller and the underlying cloud infrastructure.

![Figure 8.3 Cloud back-end load balancer configuration](media/figure8-3.png)

### ❓ What IP address should be used when configuring DNS entries for the Ingress hostnames, and where can this address be found?
You should use the IP address of the load balancer when configuring DNS entries. This IP can be found in the output of a `kubectl describe ing` command. If you're using a local cluster and the address shows as `localhost`, you should use the localhost IP address instead.

### ❓ What are the two different approaches to Ingress controller availability mentioned in the chapter summary?
The two approaches to Ingress controller availability are: 1) Lots of Kubernetes clusters require you to install an Ingress controller, and 2) Some hosted Kubernetes services make things easy by shipping with a built-in Ingress controller.

