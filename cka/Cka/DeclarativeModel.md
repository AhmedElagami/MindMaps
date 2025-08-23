# Declarative Model

## Content

### ❓ What is the declarative method used to define and deploy Pods to the Kubernetes cluster?
You define Pods in declarative YAML files that you post to the API server and the control plane schedules them to the cluster.

### ❓ Analyze the YAML structure below that defines a Namespace called shield with specific labels.
The following YAML defines a Namespace called "shield" with a label `env: marvel`:

```bash
kind: Namespace
apiVersion: v1
metadata:
  name: shield
  labels:
    env: marvel
```

This YAML structure specifies:
- `kind: Namespace` - defines the resource type as a Namespace
- `apiVersion: v1` - uses the core v1 API version
- `metadata.name: shield` - sets the Namespace name to "shield"
- `metadata.labels.env: marvel` - adds a label with key "env" and value "marvel" for organizational purposes

### ❓ What does the following command sequence do to apply the shield Namespace configuration?
The following command applies the shield Namespace configuration defined in the shield-ns.yml file:

```bash
$ kubectl apply -f shield-ns.yml
namespace/shield created
```

This command uses the declarative approach to create the Namespace by reading the YAML configuration from the specified file. The output confirms that the shield Namespace was successfully created.

### ❓ Analyze the following YAML configuration that defines three objects targeted at the shield Namespace.
The following YAML defines three objects (ServiceAccount, Service, and Pod) all targeted at the shield Namespace:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  namespace: shield     <<---- Namespace
  name: default
---
apiVersion: v1
kind: Service
metadata:
  namespace: shield     <<---- Namespace
  name: the-bus
spec:
  type: LoadBalancer
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    env: marvel
---
apiVersion: v1
kind: Pod
metadata:
  namespace: shield     <<---- Namespace
  name: triskelion
<Snip>
```

This configuration includes:
1. A ServiceAccount named "default" in the shield Namespace
2. A Service named "the-bus" of type LoadBalancer in the shield Namespace, exposing port 8080 and selecting pods with label `env: marvel`
3. A Pod named "triskelion" in the shield Namespace (details snipped)

Each object explicitly specifies `namespace: shield` in its metadata, ensuring they are all deployed to the shield Namespace.

### ❓ How does the declarative model in Kubernetes differ from providing implementation details to the system?
The declarative model is how we declare a desired state to Kubernetes without telling Kubernetes how to implement it. You leave the how up to Kubernetes.

### ❓ Using the chocolate cake analogy, explain the fundamental difference between declarative and imperative models in Kubernetes.
Declarative: Give me a chocolate cake to feed ten people.

Imperative: Drive to store. Buy eggs, milk, flour, cocoa powder... Drive home. Preheat the oven. Mix the ingredients. Place in a cake tin. If a fan-assisted oven, place the cake in the oven for 30 minutes. If not a fan-assisted oven, place the cake in the oven for 40 minutes. Set a timer. Remove from the oven when the timer expires and turn the oven off. Leave to stand until cool. Add frosting.

### ❓ What are the key advantages of the declarative model over the imperative model in Kubernetes operations?
The declarative model is simpler and leaves the how up to Kubernetes. The imperative model is much more complex as you need to provide all the steps and commands that will hopefully achieve an end goal.

### ❓ What is Kubernetes' preferred model for managing state, and what does it support as an alternative?
Kubernetes strongly prefers the declarative model for managing state, but it also supports the imperative model as an alternative.

### ❓ What is Kubernetes' official stance regarding support for both declarative and imperative models?
Kubernetes supports both models but strongly prefers the declarative model.

### ❓ What does the following full command sequence do to prepare the Git repository for working with Kubernetes Deployments?
The command sequence prepares the Git repository for working with Kubernetes Deployments by:

1. Cloning the book's GitHub repository
2. Changing into the TKB directory
3. Fetching the latest changes from the origin remote
4. Creating and switching to a new local branch called "2025" that tracks the remote "2025" branch

```bash
$ git clone https://github.com/nigelpoulton/TKB.git
Cloning into 'TKB'...

$ cd TKB

$ git fetch origin

$ git checkout -b 2025 origin/2025
```

### ❓ How should you work with Deployments in relation to the Kubernetes API, and what approach is recommended?
Like Pods, Deployments are objects in the Kubernetes API, and you should work with them declaratively.

### ❓ What is the proper way to configure ReplicaSets according to Kubernetes best practices?
You shouldn't directly create or edit ReplicaSets, you should always configure them via a Deployment.

