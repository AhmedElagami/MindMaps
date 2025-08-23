# Deployment-ReplicaSet Relationship

## Content

### ❓ Describe the relationship between Deployments and ReplicaSets as shown in Figure 6.3 and explained in the accompanying text.
The relationship between Deployments and ReplicaSets is hierarchical:

![Deployment-ReplicaSet Architecture Diagram](media/figure6-3.png)

Deployments sit above ReplicaSets in the architecture:
- **ReplicaSets** provide the actual self-healing and scaling capabilities
- **Deployments** govern ReplicaSet configuration and add mechanisms for rollouts and rollbacks

When you post a Deployment YAML to the cluster, it creates:
1. A Deployment
2. A ReplicaSet 
3. The specified number of identical Pods running identical containers

The Pods are managed by the ReplicaSet, which in turn is managed by the Deployment. However, you perform all management via the Deployment and never directly manage the ReplicaSet or Pods.

### ❓ What two application design characteristics are essential for Deployments to perform optimal zero-downtime rolling updates, and how do they work?
Deployments work best if you design your apps to be: 1) Loosely coupled via APIs, and 2) Backward and forward compatible. Both are hallmarks of modern cloud-native microservices apps. Your microservices should always be loosely coupled and only communicate via well-defined APIs, which means you can update and patch any microservice without having to worry about impacting others. Ensuring releases are backward and forward-compatible means you can perform independent updates without caring which versions of clients are consuming your service.

### ❓ Using the car analogy, explain how maintaining a consistent 'API' (steering wheel and pedals) allows for internal component updates without requiring driver retraining.
Cars expose a standard driving "API" that includes a steering wheel and foot pedals. As long as you don't change this "API", you can re-map the engine, change the exhaust, and get bigger brakes, all without the driver having to learn any new skills.

### ❓ Describe the step-by-step process Kubernetes uses to perform a zero-downtime rollout of five stateless microservice replicas.
Assume you're running five replicas of a stateless microservice. Clients can connect to any of the five replicas as long as all clients connect via backward and forward-compatible APIs. To perform a rollout, Kubernetes creates a new replica running the new version and terminates one running the old version. At this point, you've got four replicas on the old version and one on the new. This process repeats until all five replicas are on the new version. As the app is stateless and multiple replicas are up and running, clients experience no downtime or interruption of service.

### ❓ What four key elements does a Kubernetes Deployment describe for managing microservices?
Each Deployment describes all the following:
- Number of Pod replicas
- Container images to use
- Network ports
- How to perform rolling updates

### ❓ How does the relationship between Deployments and ReplicaSets work, and what additional capabilities do Deployments provide?
A Deployment sits above the ReplicaSet, governing its configuration and adding mechanisms for rollouts and rollbacks. You post Deployment YAML files to the API Server, and the ReplicaSet controller ensures the correct number of Pods get scheduled. It also watches the cluster, ensuring observed state matches desired state.

### ❓ What happens to the observed state versus desired state when you update a Deployment YAML file with a new Pod specification, and how does the Deployment controller manage the transition between old and new ReplicaSets during a rolling update?
When you update a Deployment YAML file with the new Pod spec and re-post it to the API server, this updates the existing Deployment object with a new desired state requesting the same number of Pods, but all running the newer version. At this point, observed state no longer matches desired state - you've got five old Pods, but you want five new ones. To reconcile, the Deployment controller creates a new ReplicaSet defining the same number of Pods but running the newer version. You now have two ReplicaSets - the original one for the Pods with the old version and the new one for the Pods with the new version. The Deployment controller systematically increments the number of Pods in the new ReplicaSet as it decrements the number in the old ReplicaSet. The net result is a smooth, incremental rollout with zero downtime.

### ❓ What does Figure 6.4 illustrate about the state of ReplicaSets after a rolling update is complete and why is it important that the old ReplicaSet still exists with its configuration intact?
Figure 6.4 illustrates that after a rolling update is complete, the initial ReplicaSet (on the left) isn't managing any Pods, whereas the new ReplicaSet (on the right) is managing all three live Pods. This indicates the update is complete because the old ReplicaSet has been wound down while the new one is fully operational.

![Rolling Update Completion Diagram](media/figure6-4.png)

It's important that the old ReplicaSet still exists with its configuration intact because their configurations can be used to easily roll back to earlier versions. The rollback process is the opposite of a rollout - winding an old ReplicaSet up while the current one winds down.

### ❓ What does the Deployment controller use to determine how to update Pods during a rollout?
The Deployment controller uses the rollout strategy settings to determine how to update Pods when performing a rollout. These settings tell the Deployment controller how to update the Pods when performing a rollout, and they will be explained later in the chapter when you perform a rollout.

### ❓ Explain what happens when you run the following full command sequence to create a Deployment.
When you run the command `kubectl apply -f deploy.yml`, it creates the Deployment on your cluster. All kubectl commands include the necessary authentication tokens from your kubeconfig file.

```bash
$ kubectl apply -f deploy.yml
deployment.apps/hello-deploy created
```

### ❓ What happens to the Deployment configuration after it is applied to the cluster and what controllers are responsible for managing its state?
After the Deployment configuration is applied to the cluster, the Deployment configuration is persisted to the cluster store as a record of intent. The Deployment and ReplicaSet controllers are also running in the background, watching the state of play and eager to perform their reconciliation magic. Kubernetes has scheduled ten replicas to healthy worker nodes at this point.

### ❓ Analyze the following full output from kubectl get deploy and kubectl describe deploy commands to identify key Deployment status indicators.
The output from the kubectl get deploy and kubectl describe deploy commands shows detailed information about the Deployment status. Key indicators include:
- READY: 10/10 replicas are ready
- UP-TO-DATE: 10 replicas are updated to the desired state
- AVAILABLE: 10 replicas are available
- Replicas: 10 desired | 10 updated | 10 total | 10 available | 0 unavailable
- StrategyType: RollingUpdate
- MinReadySeconds: 10
- RollingUpdateStrategy: 1 max unavailable, 1 max surge

```bash
$ kubectl get deploy hello-deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
hello-deploy   10/10   10           10          105s

$ kubectl describe deploy hello-deploy
Name:                   hello-deploy
Namespace:              default
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=hello-world
Replicas:               10 desired | 10 updated | 10 total | 10 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        10
RollingUpdateStrategy:  1 max unavailable, 1 max surge
Pod Template:
  Labels:  app=hello-world
  Containers:
   hello-pod:
    Image:        nigelpoulton/k8sbook:1.0
    Port:         8080/TCP
<SNIP>
OldReplicaSets:  <none>
NewReplicaSet:   hello-deploy-54f5d46964 (10/10 replicas created)
<Snip>
```

### ❓ What is the relationship between Deployments and ReplicaSets in Kubernetes?
Deployments automatically create associated ReplicaSets. As mentioned earlier, Deployments automatically create associated ReplicaSets. The ReplicaSet is controlled by the Deployment, and the ReplicaSet's status (observed state) gets rolled up into the Deployment status.

### ❓ Interpret the following full ReplicaSet listing output and explain what it reveals about the current state of the Deployment.
The ReplicaSet listing output shows that there is one ReplicaSet associated with the Deployment, which has 10 desired, current, and ready replicas that are 3 minutes and 45 seconds old. This indicates that all 10 replicas are running successfully and the Deployment is in a stable state.

```bash
$ kubectl get rs
NAME                      DESIRED   CURRENT   READY   AGE
hello-deploy-54f5d46964   10        10        10      3m45s
```

