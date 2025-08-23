# Label Selector Targeting

## Content

### ❓ Analyze the following full Service YAML definition and explain how the selector field determines which Pods will receive traffic.
The Service YAML defines a LoadBalancer Service that routes traffic to Pods based on their labels. The `selector` field with `app: hello-world` determines which Pods will receive traffic by matching Pods that have the label `app=hello-world`. This label-based selection ensures that the Service sends traffic only to Pods that are part of the specific application deployment.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lb-svc
  labels:
    app: hello-world
spec:
  type: LoadBalancer
  ports:
  - port: 8080
    protocol: TCP
  selector:
    app: hello-world            <<---- Send traffic to Pods with this label
```

### ❓ How does Kubernetes use labels to determine which pods to update during a deployment rollout, and how do the selector and template labels work together in the YAML configuration?
Kubernetes uses labels to determine which Pods to update during deployment rollouts. If you look closely at the YAML file, you'll see the Deployment spec has a selector block. This is a list of labels the Deployment controller looks for when finding Pods to update during rollouts. In this example, the controller will look for Pods with the `app: hello-world` label. If you look at the Pod template towards the bottom of the file, you'll notice it creates Pods with this same label. Net result: This deployment creates Pods with the `app: hello-world` label and selects Pods with the same label when performing updates, etc.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-deploy
spec:
  selector:                    <<---- The Deployment will manage all
    matchLabels:               <<---- replicas on the cluster with
      app: hello-world         <<---- this label
      <Snip>
  template:
    metadata:
      labels:
        app: hello-world       <<---- Matches the label selector above
<Snip>
```

The selector defines which Pods the Deployment will manage (all replicas with the `app: hello-world` label), and the template specifies that newly created Pods will have this same label, ensuring consistency in pod selection and management.

### ❓ Why can't you change the selector or labels after creating a Deployment, and what command is used to apply updated deployment manifests?
Pods and Deployments are both immutable, meaning you cannot change the selector or labels after you create the Deployment. This immutability ensures consistency in pod management and prevents configuration conflicts during updates.

To apply an updated deployment manifest and start the rollout process, use the following command:

```bash
$ kubectl apply -f deploy.yml
deployment.apps/hello-deploy configured
```

### ❓ Explain what the following full code block illustrates about Kubernetes rollback behavior, label management evolution, and the relationship between Deployments, ReplicaSets, and Pods. How do modern versions of Kubernetes prevent Deployments from seizing ownership of static Pods with matching labels, and what specific label is used for this protection?
```
- As with the rollout, the rollback replaces two Pods at a time and waits ten seconds after each
- Congratulations
  - You've performed a rolling update and a successful rollback
- #### Rollouts and labels
  - You've already seen that Deployments and ReplicaSets use labels and selectors to determine which Pods they own and manage
  - In earlier versions of Kubernetes, Deployments would seize ownership of static Pods if their labels matched the Deployment's label selector
    - However, recent versions of Kubernetes prevent this by adding a system-generated label to Pods created by controllers
  - Consider a quick example
    - Your cluster has five static Pods with the label
    - You add a new Deployment requesting ten Pods with the same label
    - Older versions of Kubernetes would see the existing five static Pods with the same label, seize ownership of them, and only create five new ones
    - The net result would be ten Pods with the label, all owned by the Deployment
    - However, the original five static Pods might be running a different app, and you might not want the Deployment managing them
  - Fortunately, modern versions of Kubernetes tag all Pods created by a Deployment (ReplicaSet) with the label
    - This stops higher-level controllers from seizing ownership of existing static Pods
  - Look closely at the following snipped output to see how the label connects Deployments to ReplicaSets, and ReplicaSets to Pods
  - ```bash
$ kubectl describe deploy hello-deploy
Name:      hello-deploy
<Snip>
NewReplicaSet:   hello-deploy-54f5d46964  

$ kubectl describe rs hello-deploy-54f5d46964     
Name:           hello-deploy-54f5d46964
<Snip>>
Selector:       app=hello-world,pod-template-hash=54f5d46964

$ kubectl get pods --show-labels
NAME                        READY   STATUS    LABELS
hello-deploy-54f5d46964..   1/1     Running   app=hello-world,pod-template-hash=54f5d46964
hello-deploy-54f5d46964..   1/1     Running   app=hello-world,pod-template-hash=54f5d46964
hello-deploy-54f5d46964..   1/1     Running   app=hello-world,pod-template-hash=54f5d46964
hello-deploy-54f5d46964..   1/1     Running   app=hello-world,pod-template-hash=54f5d46964
<Snip>
```

This code block illustrates several key concepts:

1. **Rollback Behavior**: Rollbacks follow the same pattern as rollouts - they replace Pods incrementally (two at a time) with waiting periods (ten seconds) between replacements

2. **Label Management Evolution**: Modern Kubernetes versions use the `pod-template-hash` label (e.g., `pod-template-hash=54f5d46964`) to prevent Deployments from seizing ownership of static Pods. This system-generated label is automatically added to all Pods created by controllers

3. **Protection Mechanism**: The `pod-template-hash` label stops higher-level controllers from seizing ownership of existing static Pods because ReplicaSets include this hash in their label selectors, while static Pods don't have this system-generated label

4. **Relationship Hierarchy**: The output shows how Deployments manage ReplicaSets (hello-deploy-54f5d46964), which in turn manage Pods through their selectors that include both the app label and the pod-template-hash label

### ❓ What is the key difference between how ReplicaSets and Deployments handle the pod-template-hash label in their selectors, and why is it acceptable that ReplicaSets include this label while Deployments do not?
The key difference is that **ReplicaSets include the pod-template-hash label in their label selectors, but Deployments don't**. This is acceptable because **it's actually ReplicaSets that manage Pods**. Deployments operate at a higher level by managing ReplicaSets, while ReplicaSets are responsible for the direct Pod management. The pod-template-hash label ensures that each ReplicaSet can uniquely identify and manage the specific set of Pods it created, preventing ownership conflicts with static Pods or Pods from other ReplicaSets.

### ❓ What technology do Services use to identify which Pods to send traffic to, and how does this relate to Deployments?
Services use labels and selectors to know which Pods to send traffic to. This is the same technology that tells Deployments which Pods they manage. Both Services and Deployments use label selectors to identify and manage their associated Pods, providing loose coupling between components.

### ❓ Interpret what this diagram shows about Service label selection and Pod matching criteria.
![Figure 7.2 - Services and labels](media/figure7-2.png)

This diagram illustrates how Services use label selectors to determine which Pods to send traffic to. It shows a Service selecting Pods with specific labels - Pod A, Pod B, and Pod D have all the required labels, so they receive traffic. Pod C doesn't have both required labels, so it doesn't receive traffic. The diagram demonstrates that additional labels on Pod D don't matter as long as all required labels are present.

### ❓ Explain the full YAML configuration below showing how a Deployment creates Pods and a Service routes traffic to them.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tkb-2024
spec:
  replicas: 10
  <Snip>
  template:
    metadata:
      labels:
        project: tkb        ----┐  Create Pods 
        zone: prod          ----┘  with these labels
    spec:
      containers:
  <Snip>
---
apiVersion: v1
kind: Service
metadata:
  name: tkb
spec:
  ports:
  - port: 8080
  selector:
    project: tkb            ----┐ Send to Pods 
    zone: prod              ----┘ with these labels
```

This YAML configuration defines two resources:

1. **Deployment:** Creates 10 Pod replicas with the labels `project: tkb` and `zone: prod`
2. **Service:** Routes traffic to Pods that match the label selector `project: tkb` and `zone: prod` on port 8080

The arrows show the relationship - the Deployment creates Pods with specific labels, and the Service sends traffic to Pods with those same labels.

