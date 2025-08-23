# State Management

## Content

### ❓ Explain the concept of controllers 'reconciling observed state with desired state' through background watch loops and provide a specific example of how a controller ensures application requirements are met.
They all run as background watch loops, reconciling observed state with desired state. This means controllers ensure the cluster runs what you asked it to run. For example, if you ask for three replicas of an app, a controller will ensure you have three healthy replicas and take appropriate actions if you don't.

### ❓ What are the three fundamental principles that form the core of Kubernetes' declarative model, and define desired state, observed state, and reconciliation in this context?
They work on three basic principles:
- Desired state
- Observed state
- Reconciliation

Desired state is what you want. Observed state is what you have. Reconciliation is the process of keeping observed state in sync with desired state.

### ❓ Describe the complete workflow of the Kubernetes declarative model from YAML creation to continuous reconciliation.
In Kubernetes, the declarative model works like this:
- You describe the desired state of an application in a YAML manifest file
- You post the YAML file to the API server
- Kubernetes records this in the cluster store as a record of intent
- A controller notices the observed state of the cluster doesn't match the new desired state
- The controller makes the necessary changes to reconcile the differences
- The controller keeps running in the background, ensuring observed state always matches desired state

### ❓ What happens to the YAML configuration after it's authenticated and authorized by the API server?
Once authenticated and authorized, Kubernetes persists the configuration to the cluster store as a record of intent.

### ❓ What triggers the reconciliation process in Kubernetes and what types of changes does it typically involve, including the specific actions that controllers perform?
The reconciliation process is triggered when the observed state of the cluster doesn't match the desired state. A controller will notice this difference and begin reconciliation, which typically involves making all the changes described in the YAML file including:
- scheduling new Pods
- pulling images
- starting containers
- attaching them to networks
- starting application processes

### ❓ What happens once reconciliation completes and how do controllers continue to operate?
Once the reconciliation completes, observed state will match desired state, and everything will be OK. However, the controllers keep running in the background, ready to reconcile any future differences.

### ❓ What are the three fundamental state concepts that are vital to understand in Kubernetes deployment theory?
The three fundamental state concepts are: Desired state, Observed state (sometimes called actual state or current state), and Reconciliation.

### ❓ Define desired state and observed state in Kubernetes, and explain the relationship between them.
Desired state is what you want, observed state is what you have, and the goal is for them to always match.

