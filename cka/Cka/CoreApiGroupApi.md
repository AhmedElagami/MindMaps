# Core API Group (/api)

## Content

### ❓ List five example resources that belong to the core API group, explain their REST path structure, and how does it differ between namespaced and non-namespaced objects?
Some of the resources in the core group include Pods, Nodes, Services, Secrets, and ServiceAccounts. You can find them in the API below the /api/v1 REST path with the following specific paths:

- Resource: Pods, REST Path: /api/v1/namespaces/{namespace}/pods/
- Resource: Services, REST Path: /api/v1/namespaces/{namespace}/services/
- Resource: Nodes, REST Path: /api/v1/nodes/
- Resource: Namespaces, REST Path: /api/v1/namespaces/

Notice how some objects are namespaced and some aren't. Namespaced objects have longer REST paths as you have to include two additional segments --- /namespaces/{namespace}. Objects, such as nodes, that aren't namespaced have much shorter REST paths.

### ❓ What are the exact REST paths required to list all Pods in a specific namespace and to access non-namespaced resources like Nodes in the core API group?
For example, listing all Pods in the shield Namespace requires the following path:

```bash
GET /api/v1/namespaces/shield/pods/
```

Objects, such as nodes, that aren't namespaced have much shorter REST paths:

```bash
GET /api/v1/nodes/
```

Expected HTTP response codes for read requests are 200: OK or 401: Unauthorized.

### ❓ What is a ConfigMap and what role does it play in the Kubernetes ecosystem according to the theory section, and what does the fact that ConfigMaps are first-class objects in the core API group indicate about their stability and maturity?
Kubernetes has an API resource called a ConfigMap (CM) that lets you store configuration data outside of Pods and inject it at run time.

ConfigMaps are first-class objects in the core API group. This tells us a few things:

- They're stable ()
- They've been around for a while (new stuff never goes in the core API group)
- You can define and deploy them in YAML files
- You can manage them with

