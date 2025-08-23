# Deployment Architecture

## Content

### ❓ Why are Pods typically managed through higher-level controllers rather than deployed directly, and what are the two essential concepts to understand about Kubernetes application packaging?
You'll almost always deploy and manage Pods via higher-level controllers. For example, you can wrap Pods inside of Deployments for scaling, self-healing, and rollouts. The two essential concepts are:
1. Apps need to be wrapped in Pods to run on Kubernetes
2. Pods get wrapped in higher-level controllers for advanced features

### ❓ What additional value do Kubernetes controllers provide, similar to courier services?
It implements controllers that add value, such as:
- Ensuring the health of apps
- Automatically scaling when demand increases
- And more

### ❓ Explain what the diagram illustrates about the nesting relationship between containers, Pods, and Deployments and what specific value each layer of wrapping adds to application deployment?
![Figure 2.7 - Object nesting](media/figure2-7.png)

The diagram shows a container wrapped in a Pod, which, in turn, is wrapped in a Deployment. Each layer of wrapping adds something:
- The container wraps the app and provides dependencies
- The Pod wraps the container so it can run on Kubernetes
- The Deployment wraps the Pod and adds self-healing, scaling, and more

### ❓ What higher-level controllers are typically used to deploy Pods in Kubernetes instead of deploying them directly, and how do these control plane services operate?
Even though Kubernetes works with Pods, you'll almost always deploy them via higher-level controllers such as: Deployments, StatefulSets, DaemonSets. These are all control plane services that operate as background watch loops, reconciling observed state with desired state.

### ❓ What four key capabilities do Deployments add to stateless applications in Kubernetes?
Deployments add to stateless apps: self-healing, scaling, rolling updates, versioned rollbacks.

### ❓ What four key capabilities do Deployments provide to augment Pods in Kubernetes?
Deployments augment Pods with four key capabilities: self-healing, scalability, rolling updates, and rollbacks.

### ❓ What are the four key capabilities that Deployments add to Pods according to the chapter summary, and what is the recommended approach for working with Deployments in Kubernetes?
According to the chapter summary, Deployments augment Pods with four key capabilities:

1. **Self-healing** - Automatic replacement of failed Pods
2. **Scalability** - Ability to scale the number of Pod replicas up or down
3. **Rolling updates** - Controlled, organized updates by gradually replacing old Pods with new ones
4. **Rollbacks** - Ability to revert to previous versions if updates fail

The recommended approach for working with Deployments is to **work with them declaratively**, just like Pods. This means defining them as objects in the Kubernetes API and letting the controller running as a reconciliation loop on the control plane manage the actual implementation. You should always configure ReplicaSets via a Deployment rather than creating or editing ReplicaSets directly.

### ❓ What application deployment is created by running the deploy.yml file, what are its characteristics, and how do you deploy and verify it?
Running the deploy.yml file creates a sample app called svc-test, which is a Deployment that creates ten Pods with the following characteristics:
- Runs a web app listening on port 8080
- Has the `chapter: services` label
- Creates 10 replicas of the Pod

To deploy the sample application:

```bash
$ kubectl apply -f deploy.yml
deployment.apps/svc-test created
```

To verify the successful deployment of Pods:

```bash
$ kubectl get deploy svc-test
NAME        READY    UP-TO-DATE    AVAILABLE    AGE
svc-test    10/10    10            10           24s
```

This shows that all 10 Pods are ready, up-to-date, and available.

