# Ingress Classes

## Content

### ❓ How do Ingress classes enable running multiple Ingress controllers on a single cluster and what is the relationship between Ingress controllers and Ingress classes according to the mapping process described?
Ingress classes enable running multiple Ingress controllers on a single cluster through a simple mapping process:

"Ingress classes allow you to run multiple Ingress controllers on a single cluster. The process is simple:

You map each Ingress controller to its own Ingress class

When you create Ingress objects, you assign them to an Ingress class"

The relationship between Ingress controllers and Ingress classes is that each Ingress controller is mapped to its own specific Ingress class. When you create Ingress objects, you assign them to a particular Ingress class, which determines which controller will manage those routing rules.

### ❓ What does the output of the kubectl get ingressclass command indicate about the available Ingress classes in the cluster?
The `kubectl get ingressclass` command output shows the available Ingress classes in the cluster:

```bash
$ kubectl get ingressclass
NAME    CONTROLLER             PARAMETERS   AGE
nginx   k8s.io/ingress-nginx   <none>       2m25s
```

This output indicates:
- There is one Ingress class called "nginx" available
- The controller for this class is "k8s.io/ingress-nginx" (the NGINX Ingress controller)
- There are no additional parameters configured (\<none\>)
- The Ingress class has been running for 2 minutes and 25 seconds
- This class was automatically created when the NGINX controller was installed

### ❓ What does the kubectl describe command output reveal about the nginx Ingress class configuration, and what are the two types of applications and their corresponding ClusterIP Services that need to be deployed for the Ingress routing configuration?
The `kubectl describe ingressclass nginx` command output reveals the following configuration details about the nginx Ingress class:

```bash
$ kubectl describe ingressclass nginx
Name:         nginx
Labels:       app.kubernetes.io/component=controller
              app.kubernetes.io/instance=ingress-nginx
              app.kubernetes.io/name=ingress-nginx
              app.kubernetes.io/part-of=ingress-nginx
              app.kubernetes.io/version=1.9.4
Annotations:  <none>
Controller:   k8s.io/ingress-nginx
Events:       <none>
```

This shows the Ingress class is named "nginx" and is managed by the "k8s.io/ingress-nginx" controller with specific Kubernetes application labels and version 1.9.4.

For the Ingress routing configuration, two applications and their corresponding ClusterIP Services need to be deployed:

1. An app called "shield" listening on port 8080, fronted by a ClusterIP Service called "svc-shield"
2. Another app called "hydra" also listening on port 8080, fronted by a ClusterIP Service called "svc-hydra"

### ❓ What information does the CLASS field provide about Ingress rules and what might it show if only one controller exists?
The CLASS field shows which Ingress class is handling this set of rules. It may show as default if you only have one Ingress controller and didn't configure classes.

