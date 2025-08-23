# Resource Management

## Content

### ❓ Describe the organizational structure example provided for implementing Namespaces in a production environment, including the three departments and their corresponding deployment targets, and what four types of resources can each Namespace have configured independently?
You might divide a production cluster into three Namespaces to match your organizational structure: finance, hr, and corporate-ops. You'd deploy Finance apps to the finance Namespace, HR apps to the hr Namespace, and Corporate apps to the corporate-ops Namespace. Each Namespace can have its own: users, permissions, resource quotas, and policies.

### ❓ What four types of configurations can be customized per Namespace according to the chapter summary?
Each Namespace can have its own:
1. Users
2. RBAC rules
3. Resource quotas
4. Selectively applied policies

### ❓ What Kubernetes objects should be limited in number within a Namespace as a good practice for DoS protection and what does the example YAML manifest do to limit Pod objects?
Limiting Kubernetes objects can also be a good practice. This includes limiting things such as the number of ReplicaSets, Pods, Services, Secrets, and ConfigMaps in a particular Namespace. Here's an example manifest that limits the number of Pod objects in the Namespace to 100:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pod-quota
  namespace: skippy
spec:
  hard:
    pods: "100"
```

