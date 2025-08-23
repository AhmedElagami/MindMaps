# Programming Models

## Content

### ❓ What are the key differences between the imperative and declarative models in Kubernetes and why does Kubernetes prefer the declarative model with what capabilities it enables?
The key differences are:
- The imperative model requires complex scripts of platform-specific commands to achieve an end-state
- The declarative model is a simple platform-agnostic way of describing an end state

Kubernetes supports both but prefers the declarative model because it integrates with version control systems and enables:
- self-healing
- autoscaling
- rolling updates

### ❓ What are the two methods available for working with Namespaces and how do they differ?
There are two methods for working with Namespaces: imperatively (via the CLI) and declaratively (via config files). The imperative method involves using direct kubectl commands with flags, while the declarative method uses YAML configuration files to define the desired state of Namespaces and other objects.

### ❓ What are the two methods for deploying objects to specific Namespaces and how do you use each method?
There are two ways to deploy objects to specific Namespaces: imperatively and declaratively. 

To do it imperatively, add the --namespace or -n flag to commands. 

To do it declaratively, you specify the Namespace in the object's YAML manifest by including the `namespace: [name]` field in the metadata section of each object definition.

### ❓ Compare and contrast the imperative and declarative methods for scaling Kubernetes Deployments, including the potential risks of each approach and what problem arises when using imperative scaling that creates a mismatch between the live environment and declarative manifests.
The imperative method uses the `kubectl scale` command to directly change replica counts, while the declarative method requires updating the Deployment YAML file and re-posting it to the cluster. Kubernetes prefers the declarative method.

The potential problem with imperative scaling is that it creates a mismatch between the live environment state and the declarative manifest. When you scale imperatively (e.g., down to 5 replicas), the cluster has 5 replicas but your YAML file still defines 10. This can cause issues during future updates - for example, if you update the image version in the YAML and re-apply it, the replica count will also revert back to 10, which might not be desired.

For this reason, you should always keep YAML manifests in sync with your live environment by making all changes declaratively via YAML manifests.

### ❓ What problem arises when you manually scale a deployment without updating the declarative YAML manifest and what specific issue can occur when updating the image version if the replica count is out of sync?
When you manually scale a deployment without updating the declarative YAML manifest, a significant problem arises: the current state of your environment no longer matches your declarative manifest. There are five replicas on your cluster, but your Deployment YAML still defines 10.

This mismatch can cause serious issues when using the YAML file to perform future updates. Specifically, if you update the image version in the YAML file and re-post it to the cluster, it will also change the number of replicas back to 10, which you might not want. This means that an intended image update would unintentionally also scale your deployment back to the original replica count specified in the YAML file.

### ❓ What is the recommended approach to maintain synchronization between YAML manifests and the live Kubernetes environment and explain what happens when you apply a YAML file using 'kubectl apply -f' to resynchronize the replica count?
The recommended approach to maintain synchronization between YAML manifests and the live Kubernetes environment is to always keep your YAML manifests in sync with your live environment. The easiest way to do this is by making all changes declaratively via your YAML manifests rather than using imperative commands.

When you apply a YAML file using `kubectl apply -f` to resynchronize the replica count, Kubernetes reconciles the live state with the desired state defined in the YAML file. The command output shows:

```bash
$ kubectl apply -f deploy.yml
deployment.apps/hello-deploy configured

$ kubectl get deploy hello-deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
hello-deploy   10/10   10           10          38m
```

The first command returns "deployment.apps/hello-deploy configured" indicating that the deployment has been updated to match the YAML specification. The verification command then shows that the deployment now has 10 out of 10 replicas ready, up-to-date, and available, matching the replica count specified in the YAML file.

### ❓ What are the two methods available for creating ConfigMaps and Secrets in Kubernetes?
ConfigMaps and Secrets can be created using two methods: "imperatively and declaratively." This means:

1. **Imperative method**: Using kubectl commands directly to create resources on the fly without writing manifest files
2. **Declarative method**: Creating YAML manifest files that define the desired state and applying them to the cluster using kubectl apply

Both methods are supported because ConfigMaps and Secrets are first-class objects in the Kubernetes API, providing flexibility in how you manage your configuration and sensitive data.

