# Resource Definition Hub

## Content

### ❓ What command is used to examine how Kubernetes stores ConfigMap entries and what does the output reveal about testmap1?
To examine how Kubernetes stores ConfigMap entries, run the following command:

```bash
$ kubectl describe cm testmap1
```

The output reveals that testmap1 stores its data as simple key-value pairs in the Data section:

```bash
$ kubectl describe cm testmap1
Name:         testmap1
Namespace:    default
Labels:       <none>
Annotations:  <none>
Data
====
shortname:
----
SAFC
longname:
----
Sunderland Association Football Club
BinaryData
====
Events:  <none>
```

This shows that ConfigMaps are essentially just maps of key-value pairs dressed up as Kubernetes objects.

### ❓ What does it mean that ConfigMaps are 'first-class API objects' and how can you list and inspect them?
ConfigMaps being 'first-class API objects' means you can inspect and query them like any other Kubernetes API object. You can list all ConfigMaps in your current Namespace with:

```bash
$ kubectl get cm
AME       DATA   AGE
testmap1   2      11m
testmap2   1      2m23s
```

When inspecting a ConfigMap created from a file, three key pieces of information are revealed:
1. The operation created a single map entry
2. The name of the key matches the name of the input file
3. The value stores the contents of the file

For example, inspecting testmap2 shows:

```bash
$ kubectl describe cm testmap2
Name:         testmap2
Namespace:    default
Labels:       <none>
Annotations:  <none>
Data
====
cmfile.txt:                   <<---- key
----
Kubernetes FTW!               <<---- value
BinaryData
====
Events:  <none>
```

### ❓ How can you view the complete YAML representation of a ConfigMap and what does it reveal about missing subresources?
You can view the complete YAML representation of a ConfigMap by running a `kubectl get` command with the `-o yaml` flag:

```bash
$ kubectl get cm testmap2 -o yaml
apiVersion: v1
data:
  cmfile.txt: |
    Kubernetes FTW!
kind: ConfigMap
metadata:
  creationTimestamp: "2025-02-04T14:19:21Z"
  name: testmap2
  namespace: default
  resourceVersion: "18128"
  uid: 146da79c-aa09-4b10-8992-4dffa087dbfb
```

If you look closely, you'll notice certain subresources are missing. This is because ConfigMaps don't have the concept of desired state and observed state. They have a different subresource structure instead.

### ❓ Analyze the YAML output from the kubectl get secret command. What does this reveal about how Kubernetes stores secret data?
The YAML output reveals that Kubernetes stores secret data as base64-encoded values in the `data` field:

```bash
$ kubectl get secret creds -o yaml
apiVersion: v1
kind: Secret
data:
  pwd: UGFzc3dvcmQxMjM=
  user: bmlnZWxwb3VsdG9u
<Snip>
```

This shows that both the password and username values are base64 encoded (pwd: UGFzc3dvcmQxMjM= and user: bmlnZWxwb3VsdG9u), demonstrating that Kubernetes obscures Secrets by encoding them as base64 values rather than encrypting them.

### ❓ Analyze the full YAML structure of the tkb-secret. What field would need to be changed to use plain text entries instead of base64-encoded ones?
The full YAML structure of the tkb-secret is:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tkb-secret
  labels:
    chapter: configmaps
type: Opaque
data:                          <<---- Change to "stringData" for plain text entries
  username: bmlnZWxwb3VsdG9u
  password: UGFzc3dvcmQxMjM=
```

To use plain text entries instead of base64-encoded ones, you would need to change the `data` block to `stringData`. This allows you to add plain text entries, but you should never store either type on places like GitHub, as anyone with access can read them.

### ❓ What is the primary purpose of ConfigMaps and Secrets in Kubernetes architecture and what does it mean that they are 'first-class objects' in the Kubernetes API?
The primary purpose of ConfigMaps and Secrets in Kubernetes architecture is to "decouple applications from their configuration data." This means separating application configuration and sensitive data from the application code itself, making applications more portable and easier to manage across different environments.

ConfigMaps and Secrets being "first-class objects in the Kubernetes API" means they are native resource types that are fully integrated into the Kubernetes API server. This status provides several benefits:
- They can be created, read, updated, and deleted using standard Kubernetes API operations
- They benefit from all the standard Kubernetes features like RBAC, validation, and watch operations
- They can be managed using familiar kubectl commands and YAML manifests
- They integrate seamlessly with other Kubernetes resources like Pods and Deployments

### ❓ What is the fundamental requirement for any object to be deployable in Kubernetes and what types of properties can be configured on API resources like Pods?
The fundamental requirement for any object to be deployable in Kubernetes is that "if an object doesn't exist in the API, you can't deploy it." This is similar to Amazon where you can only buy stuff listed on the website.

API resources have properties you can inspect and configure. For Pods specifically, you can configure properties including:
- metadata (name, labels, Namespace, annotations...)
- restart policy
- service account name
- runtime class
- containers
- volumes

### ❓ Explain the distinction between API resource definitions and running objects in a Kubernetes cluster with an example.
Most of the resources in the Kubernetes API are objects, so we sometimes use the terms resources and objects to mean the same thing. It's common to refer to their API definitions as resources or resource definitions, whereas running instances on a cluster are often referred to as objects. For example, "The Pod resource exists in the core API group, and there are five Pod objects running in the default Namespace."

