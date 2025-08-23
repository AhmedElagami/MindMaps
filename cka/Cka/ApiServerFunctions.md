# API Server Functions

## Central Communication Hub

## Content

### ❓ What does the diagram in Figure 15.1 illustrate about the central role of the API server in Kubernetes operations and what does Figure 15.2 demonstrate about the parallel between e-commerce systems and Kubernetes API operations?
Figure 15.1 shows the high-level process and highlights the central nature of the API and API server in Kubernetes operations. It illustrates that all resources are defined in the API, and all configuration and querying goes through the API server. Administrators and clients send requests to the API server to create, read, update, and delete objects. If it's a request to create an object, Kubernetes persists the object definition in the cluster store in its serialized state and schedules it to the cluster.

![Figure 15.1](media/figure15-1.png)

Figure 15.2 demonstrates the comparison between e-commerce systems and Kubernetes API operations through an analogy with Amazon. It shows a feature-for-feature comparison where:

- Amazon stuff corresponds to Kubernetes resources/objects
- Amazon warehouse corresponds to the Kubernetes API
- Browser corresponds to kubectl
- Amazon website corresponds to the API server

![Figure 15.2](media/figure15-2.png)

The analogy reveals that just as Amazon sells stuff stored in warehouses and exposed online via their website, Kubernetes has resources defined in the API and exposed through the API server. Users employ tools like kubectl to talk to the API server and request resources, similar to using browsers to access the Amazon website. When resources are requested through the API server, they get created on the cluster, and users can track their creation and deployment through the API server, much like tracking Amazon orders through their website.

### ❓ How does the Amazon analogy help explain the relationship between Kubernetes resources, the API, and the API server, and what does the comparison table between Amazon and Kubernetes components reveal about the fundamental architecture of the Kubernetes API system?
The Amazon analogy helps conceptualize the Kubernetes API by comparing it to a familiar e-commerce system. Here's how the analogy works:

Amazon sells lots of stuff that is stored in warehouses and exposed online via the Amazon website. You use tools like browsers to search the website and buy stuff, and third parties can sell their own stuff through the same platform.

Similarly, Kubernetes has resources (stuff) such as Pods, Services, and Ingresses that are defined in the API (warehouse) and exposed through the API server (Amazon website). You use tools like kubectl (browser) to talk to the API server and request resources, and third parties can define their own Kubernetes resources that you access through the same API server.

The comparison table reveals the fundamental architecture:

| Amazon Component | Kubernetes Component |
|------------------|---------------------|
| Stuff | Resources/objects |
| Warehouse | API |
| Browser | kubectl |
| Amazon website | API server |

This comparison shows that:
1. All deployable objects are defined as resources in the API (just as Amazon only sells stuff listed on their website)
2. The API server acts as the front-end gateway to access these resources
3. Standard tools like kubectl are used to interact with the API server
4. The system supports third-party extensions through the same interface

The analogy demonstrates that Kubernetes, like Amazon, provides a centralized catalog of available resources (API) with a standardized interface (API server) for discovery, configuration, and management using common tools.

### ❓ What does the analogy caution about comparing Amazon to Kubernetes, what does Figure 15.2 illustrate about the Kubernetes API analogy, and what are the component mappings between Amazon and Kubernetes?
The analogy cautions that "this is just an analogy, so not everything matches perfectly" when comparing Amazon to Kubernetes.

![Figure 15.2](media/figure15-2.png)

The Amazon-Kubernetes analogy maps components as follows:

- **Amazon → Kubernetes**
- **Stuff → Resources/objects**
- **Warehouse → API**
- **Browser → kubectl**
- **Amazon website → API server**

This means that in Kubernetes, all deployable objects such as Pods, Services, and Ingresses are defined as resources in the API, similar to how Amazon only sells items listed on their website. If an object doesn't exist in the API, you can't deploy it.

### ❓ What role does the API server play in Kubernetes architecture and what are the main components that interact with it?
The API server acts as the front-end to the API and is "a bit like Grand Central station for Kubernetes --- everything talks to everything else via REST API calls to the API server."

The three main components that interact with the API server are:

1. **All kubectl commands** - creating, retrieving, updating, and deleting objects
2. **All kubelets** - watch the API server for new tasks and report status to the API server
3. **All control plane services** - share data and status info via the API server

### ❓ What are the primary functions of the Kubernetes API server and where does it run with what performance characteristics?
The API server exposes the API over a secure RESTful interface that lets you manipulate and query the state of objects on the cluster. It runs on the control plane, which needs to be highly available and have enough performance to service requests quickly.

### ❓ What is the fundamental purpose of the Kubernetes API and what three characteristics define its architecture?
The API is where all Kubernetes resources are defined. It's large, modular, and RESTful.

### ❓ Why must the Kubernetes control plane be highly available and high-performance according to the API server's role?
The API server runs as a control plane service, and all internal and external clients interact with the API via the API server. This means your control plane needs to be highly available and high-performance. If it's not, you risk slow API responses or entirely losing access to the API.

### ❓ What are the critical implications for the control plane's availability and performance based on the API server's role?
The critical implications are that "your control plane needs to be highly available and high-performance. If it's not, you risk slow API responses or entirely losing access to the API." Since all internal and external clients interact with the API via the API server, the control plane's availability and performance directly affect API responsiveness and accessibility. A poorly performing or unavailable control plane can result in slow API responses or complete loss of API access.

### ❓ What is the fundamental purpose of Denial of Service (DoS) attacks in the context of Kubernetes infrastructure and how does overloading the API server represent a particularly effective DoS attack vector?
Denial of Service (DoS) is about making something unavailable. In the Kubernetes world, a potential attack might be overloading the API server so that cluster operations grind to a halt. This is particularly effective because even internal systems use the API server to communicate, making it a critical choke point for cluster operations.


## RESTful HTTPS Interface

## Content

### ❓ What protocol and interface does the API server use to expose the Kubernetes API, what are the common ports, and what does the kubectl cluster-info command reveal?
The API server exposes the API over a RESTful HTTPS interface. It's common for the API server to be exposed on port 6443 or 443, though you can configure it to operate on whatever port you require.

To see the address and port your Kubernetes cluster is exposed on, run:

```bash
$ kubectl cluster-info
Kubernetes control plane is running at https://kubernetes.docker.internal:6443
```

This command reveals the specific address and port where your Kubernetes control plane (including the API server) is running.

### ❓ What does the following command sequence do to prepare the Git repository for the hands-on exercises and explain what happens when you run the kubectl proxy command with the --port 9000 flag?
The command sequence prepares the Git repository for the hands-on exercises by cloning the book's GitHub repository and checking out the 2025 branch:

```bash
$ git clone https://github.com/nigelpoulton/TKB.git
<Snip>

$ cd TKB

$ git fetch origin

$ git checkout -b 2025 origin/2025
```

When you run the kubectl proxy command with the --port 9000 flag, it starts a proxy session that exposes the Kubernetes API on your localhost adapter and handles all authentication:

```bash
$ kubectl proxy --port 9000 &
[1] 27533
Starting to serve on 127.0.0.1:9000
```

This command exposes the API on port 9000 of your localhost adapter (127.0.0.1:9000) and automatically handles all authentication requirements, allowing you to use tools like curl to form API requests without dealing with authentication complexities.

### ❓ How does the kubectl proxy command simplify authentication when making direct API requests to the Kubernetes API server and what role does JSON serialization play in Kubernetes API communications?
The kubectl proxy command simplifies authentication by exposing the API on your localhost adapter and handling all authentication automatically. This means you don't need to manage authentication tokens, certificates, or other credentials when making API requests through the proxy.

JSON serialization plays a fundamental role in Kubernetes API communications as it's Kubernetes' preferred serialization schema. As mentioned in the content, "Kubernetes uses JSON as its preferred serialization schema. This means a command such as kubectl get pods will generate a request with the content type set to application/json." 

When a request is properly authenticated and authorized, it results in an HTTP 200 (OK) response code, and Kubernetes responds with a serialized JSON representation of the requested resources. The API server communicates this preference through Content-Type headers, ensuring that both requests and responses use JSON format for data exchange.

### ❓ How is the Kubernetes API described in terms of its fundamental nature and exposure mechanism?
The Kubernetes API is described as "an API-driven platform, and the API is exposed internally and externally via the API server." This means Kubernetes fundamentally operates through its API, and the API server is the component that exposes this API both internally within the cluster and externally to clients.

### ❓ What are the advantages and disadvantages of Kubernetes-native applications accessing ConfigMaps directly through the API server?
Kubernetes-native applications know they're running on Kubernetes and can talk to the Kubernetes API server. This has a lot of benefits, including the ability to read ConfigMaps directly through the API without having to mount them as volumes or via environment variables. It also means live containers can see updates to the ConfigMap. However, apps like these can only run on Kubernetes and create Kubernetes lock-in where your apps only work on Kubernetes.


## Resource Definition Hub

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


## CRUD Operations Processing

## Content

### ❓ What two commands should be run to delete Kubernetes resources while keeping the cluster intact?
To delete Kubernetes resources while keeping the cluster intact, you should run the following two commands:

```bash
$ kubectl delete -f app.yml
$ kubectl delete runtimeclass rc-spin
```

The first command deletes the application deployment, service, and ingress defined in app.yml, while the second command deletes the runtimeclass.

### ❓ Explain what happens when running the following full command sequence to delete Kubernetes resources and the runtime class.
When running the full command sequence to delete Kubernetes resources and the runtime class, the following actions occur:

```bash
$ kubectl delete -f app.yml
deployment.apps "wasm-spin" deleted
service "svc-wasm" deleted
ingress.networking.k8s.io "ing-wasm" deleted

$ kubectl delete runtimeclass rc-spin
runtimeclass.node.k8s.io "rc-spin" deleted
```

The first command `kubectl delete -f app.yml` deletes three resources: the "wasm-spin" deployment, the "svc-wasm" service, and the "ing-wasm" ingress. The second command `kubectl delete runtimeclass rc-spin` deletes the "rc-spin" runtimeclass that was used for scheduling Wasm applications.

### ❓ What are the two methods described for setting the Kubernetes context back to the original cluster after working with Wasm?
The two methods described for setting the Kubernetes context back to the original cluster are:

1. **Docker Desktop method**: Click the Docker whale and choose the context from the Kubernetes context option
2. **Command line method**: Use kubectl commands to list contexts and set the current context back to the appropriate one for your environment (e.g., docker-desktop)

### ❓ Analyze the following kubectl context output and explain what each command does to manage Kubernetes contexts.
The kubectl context commands manage Kubernetes contexts as follows:

```bash
$ kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
          docker-desktop   docker-desktop   docker-desktop
          lke349416-ctx    lke349416        lke349416-admin   default
*         k3d-wasm         k3d-wasm         admin@k3d-wasm

$ kubectl config current-context docker-desktop
docker-desktop
```

1. **`kubectl config get-contexts`**: Lists all available Kubernetes contexts, showing which one is currently active (marked with *), along with cluster, authinfo, and namespace details for each context.

2. **`kubectl config current-context docker-desktop`**: Sets the current context to "docker-desktop", switching from the "k3d-wasm" context that was previously active.

### ❓ What specific command is used to retrieve detailed information about the linode-block-storage storage class?
The command used to retrieve detailed information about the linode-block-storage storage class is:

```bash
$ kubectl describe sc linode-block-storage
```

### ❓ What is the purpose of listing existing PVs and PVCs before creating new ones, as mentioned in the paragraph?
The purpose of listing existing PVs and PVCs before creating new ones is to easily identify the storage resources you're about to create and distinguish them from any pre-existing storage resources in the cluster. This helps with monitoring and management of the newly created storage objects.

### ❓ What does the output of the 'kubectl get pv' and 'kubectl get pvc' commands indicate about the current state of the cluster's storage resources?
```bash
$ kubectl get pv
No resources found
$ kubectl get pvc
No resources found in default namespace.
```

The output indicates that there are currently no Persistent Volumes (PVs) and no Persistent Volume Claims (PVCs) in the default namespace of the cluster. The cluster's storage resources are completely empty with no existing storage allocations.

### ❓ What does the output 'No resources found' indicate about the current state of PersistentVolumes in the cluster?
The output 'No resources found' indicates that there are currently no PersistentVolumes existing in the cluster. This confirms that the PV and associated backend volume were successfully deleted when the PVC was removed.

```bash
$ kubectl get pv
No resources found
```

### ❓ What are the five required steps to completely clean up the storage environment?
You need to do five things to clean up your environment:
1. Delete the Pod using the PVC
2. Delete the PVC
3. Delete the PV
4. Manually delete the volume on the Linode cloud back end
5. Delete the SC

### ❓ Explain what the following kubectl delete pod command does and what output indicates successful completion.
The `kubectl delete pod volpod` command deletes the Pod named "volpod" from the Kubernetes cluster. The output `pod "volpod" deleted` indicates successful completion of the deletion operation.

```bash
$ kubectl delete pod volpod
pod "volpod" deleted
```

### ❓ Explain what the following kubectl delete pvc command does and what output confirms the PVC deletion.
The `kubectl delete pvc pvc-wait-keep` command deletes the PersistentVolumeClaim named "pvc-wait-keep" from the Kubernetes cluster. The output `persistentvolumeclaim "pvc-wait-keep" deleted` confirms the successful deletion of the PVC.

```bash
$ kubectl delete pvc pvc-wait-keep
persistentvolumeclaim "pvc-wait-keep" deleted
```

### ❓ Explain what the following kubectl delete pv command does and what output indicates successful PV deletion.
The `kubectl delete pv pvc-279f09e083254fa9` command deletes the PersistentVolume with the specified name from the Kubernetes cluster. The output `persistentvolume "pvc-279f09e083254fa9" deleted` indicates successful completion of the PV deletion. Note that the PV name will be different for each deployment.

```bash
$ kubectl delete pv pvc-279f09e083254fa9
persistentvolume "pvc-279f09e083254fa9" deleted
```

### ❓ What is the final step in the cleanup process according to the paragraph and what command is used to execute it?
The final step in the cleanup process is to delete the storage class. This is done using the following command:

```bash
$ kubectl delete sc block-wait-keep
storageclass.storage.k8s.io "block-wait-keep" deleted
```

### ❓ How can you edit a ConfigMap using kubectl, what editor does it typically open on Linux and Mac systems, and what commands demonstrate that changes to a ConfigMap are reflected in the live container?
To edit a ConfigMap using kubectl, use the following command:

```bash
$ kubectl edit cm multimap

# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving 
# this file will be reopened with the relevant failures.
#
apiVersion: v1
data:
  City: Macclesfield       <<---- changed
  Country: UK              <<---- changed
kind: ConfigMap
metadata:
<Snip>
```

The command typically opens the default editor on Linux and Mac systems (usually `vi` or `nano`). On Windows, it usually opens with a different editor.

After making changes to the ConfigMap, you can verify that the updates appear in the live container with these commands:

```bash
$ kubectl exec cmvol -- ls /etc/name
City
Country

$ kubectl exec cmvol -- cat /etc/name/Country
UK
```

It may take a minute or so for your changes to appear in the container. This demonstrates that ConfigMap volumes automatically update when the underlying ConfigMap changes.

### ❓ What command is used to list the Pods, ConfigMaps, and Secrets that have been deployed?
The command `kubectl get` is used to list the Pods, ConfigMaps, and Secrets that have been deployed. Specifically, you would use:

- `kubectl get pods` to list Pods
- `kubectl get cm` to list ConfigMaps  
- `kubectl get secrets` to list Secrets

Here's the full output example from the commands:

```bash
$ kubectl get pods
NAME           READY    STATUS       RESTARTS    AGE
cmvol          1/1      Running      0           27m
envpod         1/1      Running      0           22m
secret-pod     1/1      Running      0           16m

$ kubectl get cm
NAME                DATA    AGE
kube-root-ca.crt    1       71m    <<---- Don't delete this one
multimap            2       19m
test-config         1       17m
testmap1            2       24m
testmap2            1       22m

$ kubectl get secrets
NAME          TYPE      DATA    AGE
creds         Opaque    2       4m55s
tkb-secret    Opaque    2       3m35s
```

### ❓ Analyze the following full output from kubectl get commands. What do the results indicate about the current state of Pods, ConfigMaps, and Secrets in the cluster?
The output from the `kubectl get` commands indicates the following about the current state of the cluster:

```bash
$ kubectl get pods
NAME           READY    STATUS       RESTARTS    AGE
cmvol          1/1      Running      0           27m
envpod         1/1      Running      0           22m
secret-pod     1/1      Running      0           16m

$ kubectl get cm
NAME                DATA    AGE
kube-root-ca.crt    1       71m    <<---- Don't delete this one
multimap            2       19m
test-config         1       17m
testmap1            2       24m
testmap2            1       22m

$ kubectl get secrets
NAME          TYPE      DATA    AGE
creds         Opaque    2       4m55s
tkb-secret    Opaque    2       3m35s
```

**Analysis:**
- **Pods**: All three pods (cmvol, envpod, secret-pod) are in "Running" status with 1/1 containers ready, no restarts, and have been running for 16-27 minutes
- **ConfigMaps**: There are 5 ConfigMaps total, including the system-generated "kube-root-ca.crt" (which should not be deleted), plus 4 user-created ConfigMaps with 1-2 data items each, aged 17-24 minutes
- **Secrets**: Two Secrets exist (creds and tkb-secret), both of type "Opaque" with 2 data items each, created 3-5 minutes ago

The cluster appears to be in a healthy state with all resources properly deployed and functioning.

### ❓ What action should be taken after listing the deployed resources according to the clean-up instructions and what is the expected timing behavior when deleting Pods?
After listing the deployed resources using `kubectl get` commands, the action that should be taken is to delete them. The clean-up instructions state: "Now delete them" using the following commands:

```bash
$ kubectl delete pods cmvol envpod secret-pod

$ kubectl delete cm multimap test-config testmap1 testmap2

$ kubectl delete secrets creds tkb-secret
```

Regarding timing behavior, "It can take a few seconds for the Pods to delete" after executing the delete commands. This indicates that Pod deletion is not instantaneous and there may be a brief delay as Kubernetes processes the termination requests and gracefully shuts down the containers.

### ❓ Explain what the following full command sequence does when executed to clean up Kubernetes resources.
The following command sequence performs a complete cleanup of the deployed Kubernetes resources by deleting all Pods, ConfigMaps, and Secrets that were created during the exercise:

```bash
$ kubectl delete pods cmvol envpod secret-pod

$ kubectl delete cm multimap test-config testmap1 testmap2

$ kubectl delete secrets creds tkb-secret
```

**What each command does:**
1. `kubectl delete pods cmvol envpod secret-pod` - Deletes the three specific Pods (cmvol, envpod, and secret-pod)
2. `kubectl delete cm multimap test-config testmap1 testmap2` - Deletes the four user-created ConfigMaps (multimap, test-config, testmap1, testmap2) while preserving the system-generated "kube-root-ca.crt" ConfigMap
3. `kubectl delete secrets creds tkb-secret` - Deletes both Secrets (creds and tkb-secret)

This sequence ensures that all the resources created for the ConfigMaps and Secrets demonstration are properly removed from the cluster.

### ❓ Analyze the following Kubernetes verbs to HTTP methods mapping table and explain how it demonstrates the REST-based nature of the Kubernetes API.
The Kubernetes verbs to HTTP methods mapping table demonstrates the REST-based nature of the Kubernetes API:

| Kubernetes verb(s) | HTTP method | Common responses |
| --- | --- | --- |
| create | POST | 201 created, 403 Access Denied |
| get, list, watch | GET | 200 OK, 403 Access Denied |
| update | PUT | 200 OK, 403 Access Denied |
| patch | PATCH | 200 OK, 403 Access Denied |
| delete | DELETE | 200 OK, 403 Access Denied |

This table demonstrates the REST-based nature by mapping Kubernetes operations directly to standard HTTP methods:
- **create** maps to **POST** (creating new resources)
- **get, list, watch** map to **GET** (retrieving resources)
- **update** maps to **PUT** (replacing entire resources)
- **patch** maps to **PATCH** (partial updates to resources)
- **delete** maps to **DELETE** (removing resources)

The common HTTP response codes (201 Created, 200 OK, 403 Access Denied) further reinforce the RESTful design, showing that the Kubernetes API follows standard HTTP conventions for success and error responses, making it consistent with web-based REST APIs.

### ❓ What is the process for deploying a Pod to the Kubernetes cluster and what are the two conditions required for successful deployment?
To deploy a Pod, you send a Pod YAML file to the API server. The two conditions required for Kubernetes to successfully deploy it are:

1. The YAML must be valid
2. You must be authorized to create Pods

Assuming these conditions are met, Kubernetes deploys the Pod to the cluster. After deployment, you can query the API server to get its status, and when it's time to delete it, you send a delete request to the API server.

### ❓ What does it mean that the Kubernetes API is RESTful, what types of operations does it support, and how do CRUD operations map to HTTP methods?
The Kubernetes API is RESTful, which is "jargon for a modern web API that accepts CRUD-style requests via standard HTTP methods."

CRUD-style operations are "simple create, read, update, delete operations" that map to standard HTTP methods:

- **Create → POST**
- **Read → GET**
- **Update → PUT/PATCH**
- **Delete → DELETE**

These CRUD operations map to the Kubernetes API as follows:

| K8s CRUD verb     | HTTP method   | kubectl example               |
|-------------------|---------------|-------------------------------|
| create            | POST          | $ kubectl create -f           |
| get list, watch   | GET           | $ kubectl get pods            |
| update            | PUT/PATCH     | $ kubectl edit deployment     |
| delete            | DELETE        | $ kubectl delete ingress      |

As shown in the mapping, CRUD verb names, HTTP method names, and kubectl sub-command names don't always match exactly, but they correspond to the same underlying operations.

### ❓ Analyze the JSON response structure returned by the curl command to list Pods in the shield namespace and what does the empty items array indicate?
The JSON response structure for listing Pods in the shield namespace follows Kubernetes' standard API response format:

```bash
$ curl -X GET http://localhost:9000/api/v1/namespaces/shield/pods 
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "9524"
  },
  "items": []
}
```

The response includes:
- `"kind": "PodList"` - indicates this is a list of Pod objects
- `"apiVersion": "v1"` - specifies the API version used
- `"metadata"` with `"resourceVersion"` - shows the version of this resource list
- `"items": []` - an empty array

The empty `items` array indicates that there are no Pods currently existing in the shield Namespace. This is the expected response when querying for resources in a namespace that doesn't contain any instances of that particular resource type.

### ❓ What does the NamespaceList JSON response reveal about the structure and content of namespace objects in Kubernetes?
The NamespaceList JSON response reveals the detailed structure and metadata of namespace objects in Kubernetes:

```bash
$ curl -X GET http://localhost:9000/api/v1/namespaces
{
  "kind": "NamespaceList",
  "apiVersion": "v1",
  "metadata": {
    "resourceVersion": "9541"
  },
  "items": [
    {
      "metadata": {
        "name": "kube-system",
        "uid": "f5d39dd2-ccfe-4523-b634-f48ba3135663",
        "resourceVersion": "10",
<Snip>
```

The response shows:
- `"kind": "NamespaceList"` - indicates this is a list of Namespace objects
- `"apiVersion": "v1"` - uses the core API group version
- `"metadata"` with `"resourceVersion"` - version identifier for the entire list
- `"items"` array - contains individual namespace objects

Each namespace object within the items array includes comprehensive metadata such as:
- `"name"` - the namespace identifier
- `"uid"` - unique identifier for the namespace
- `"resourceVersion"` - version of this specific namespace object
- `"creationTimestamp"` - when the namespace was created
- `"labels"` - key-value pairs for categorization and organization

This structure demonstrates that namespaces are first-class objects in Kubernetes with rich metadata and management capabilities.

### ❓ Describe the relationship between HTTP methods, response codes, and successful API operations in the Kubernetes REST interface.
In the Kubernetes REST interface, there's a direct relationship between HTTP methods, response codes, and successful API operations. As explained in the content: "Assuming it's authenticated and authorized, it will result in HTTP 200 (OK) response code, and Kubernetes will respond with a serialized JSON list of all Pods in the shield Namespace."

The Kubernetes API implements CRUD-style operations via standard HTTP methods:
- GET for read operations (retrieving resources)
- POST for create operations (creating new resources)
- PUT/PATCH for update operations (modifying existing resources)
- DELETE for delete operations (removing resources)

For successful operations, the API typically returns HTTP 200 (OK) along with the requested data serialized in JSON format. The response includes the appropriate content type (application/json) and the serialized payload containing the resource data.

This RESTful approach ensures that API operations follow standard web conventions, making the Kubernetes API predictable and consistent with industry standards for web-based APIs.

### ❓ Explain the purpose and expected output of the curl command with the -v flag when querying Pods in the shield namespace and how do the header indicators (> and <) help in understanding the request and response flow?
The curl command with the -v flag provides verbose output that shows both the request headers sent and response headers received, offering detailed insight into the API communication:

```bash
$ curl -v -X GET http://localhost:9000/api/v1/namespaces/shield/pods
```

The purpose of the -v flag is to display the complete HTTP transaction, including:
- Request headers sent to the API server
- Response headers returned by the API server
- Additional diagnostic information

The header indicators (> and <) are crucial for understanding the request and response flow:

- **Lines starting with >** are header data sent by curl to the API server
- **Lines starting with <** are header data returned by the API server

For example, the > lines show curl sending a GET request to the `/api/v1/namespaces/shield/pods` REST path and telling the API server it can accept responses using any valid serialization schema (Accept: */*). The lines starting with < show the API server returning an HTTP response code and using JSON serialization.

This verbose output helps developers understand exactly what information is being exchanged between the client and the Kubernetes API server, including content types, acceptance criteria, and server response details.

### ❓ What do lines starting with '<' represent in API server communication and what information do they convey about the API server's response?
Lines starting with '<' represent header data returned by the API server. These lines show the API server returning an HTTP response code and using JSON serialization. Additionally, the X-Kubernetes-Pf-... lines are priority and fairness settings specific to Kubernetes.

### ❓ What does the '>' line indicate about the curl request to the Kubernetes API server?
The > lines show curl sending a GET request to the /api/v1/namespaces/shield/pods REST path and telling the API server it can accept responses using any valid serialization schema (Accept: */*).

### ❓ What is the purpose of X-Kubernetes-Pf-... lines in Kubernetes API responses?
The X-Kubernetes-Pf-... lines are priority and fairness settings specific to Kubernetes.

### ❓ What does CRUD stand for, what are its four basic functions in web APIs, and how does the Kubernetes API implement CRUD-style operations?
CRUD is an acronym for the four basic functions web APIs use to manipulate and persist objects: Create, Read, Update, Delete. The Kubernetes API exposes and implements CRUD-style operations via the common HTTP methods.

### ❓ What HTTP method would curl use to create a new object, what does this operation represent in CRUD terms, and why do people say they're 'POSTing' a configuration?
If you ran the curl command to create a namespace, curl would form a request to the API server using the HTTP POST method. The POST method creates a new object of the specified resource type, which represents the "Create" operation in CRUD terms. This is why you'll occasionally hear people say they're POSTing a configuration to the API server.

### ❓ Analyze the following HTTP request header structure for creating a Kubernetes resource.
The HTTP request header for creating a Kubernetes resource would look like this:

```bash
POST https://<api-server>/api/v1/namespaces
Content-Type: application/json
Accept: application/json
```

This header specifies:
- POST method for creating a new resource
- The API server endpoint for namespaces
- Content-Type header indicating the request contains JSON data
- Accept header specifying the client prefers JSON responses

### ❓ What components does a successful API server response include and interpret the structure of a successful HTTP response?
If the request is successful, the response will include a standard HTTP response code, content type, and payload. The structure of a successful HTTP response from the Kubernetes API server looks like:

```bash
HTTP/1.1 200 (OK)
Content-Type: application/json
{
    <payload>
}
```

This includes the HTTP status code (200 OK), the content type (application/json), and the JSON payload containing the response data.

### ❓ What are the prerequisites for running the curl command to create a namespace?
The prerequisites for running the curl command to create a namespace are:
1. You must still have the kubectl proxy process running from earlier (listening on localhost:9000)
2. You need to run the command from the ch15 directory where the ns.json file exists
3. If the shield Namespace already exists, you'll need to delete it before continuing
4. Windows users will need to replace the backslash with a backtick and place a backtick immediately before the $ symbol

### ❓ Explain what each component of the following full curl command does when creating a namespace.
The full curl command for creating a namespace is:

```bash
$ curl -X POST -H "Content-Type: application/json" \
  --data-binary @ns.json http://localhost:9000/api/v1/namespaces

{
  "kind": "Namespace",
  "apiVersion": "v1",
  "metadata": {
    "name": "shield",
<Snip>
```

Each component serves the following purpose:
- `-X POST`: Forces curl to use the HTTP POST method
- `-H "Content-Type: application/json"`: Tells the API server the request contains serialized JSON
- `--data-binary @ns.json`: Specifies the manifest file containing the namespace definition
- The URI includes the address the API server is exposed on by kubectl proxy and the REST path for the resource

### ❓ What does the -X POST argument accomplish in the curl command?
The -X POST argument forces curl to use the HTTP POST method.

### ❓ What information does the -H "Content-Type: application/json" header provide to the API server?
The -H "Content-Type: application/json" tells the API server the request contains serialized JSON.

### ❓ What is the function of the --data-binary @ns.json parameter in the curl command?
The --data-binary @ns.json specifies the manifest file.

### ❓ What two components does the URI in the curl command include?
The URI is the address the API server is exposed on by kubectl proxy and includes the REST path for the resource.

### ❓ Analyze the output of the kubectl get ns command to verify namespace creation.
The output of the `kubectl get ns` command shows the verification that the new shield Namespace was successfully created:

```bash
$ kubectl get ns
NAME              STATUS   AGE
kube-system       Active   47h
kube-public       Active   47h
kube-node-lease   Active   47h
default           Active   47h
shield            Active   14s
```

This output shows:
- The shield namespace exists with STATUS: Active
- The shield namespace is very new (only 14 seconds old)
- All other system namespaces are present and have been active for 47 hours
- The creation was successful as shield appears in the list of namespaces

### ❓ What HTTP method and content type header are required to delete a Namespace using the Kubernetes API, and what does the full curl command do when executed against the API server including the expected response structure?
To delete a Namespace using the Kubernetes API, you need to use the DELETE HTTP method and specify the "Content-Type: application/json" header. The full curl command performs a DELETE request to the API server to remove the specified Namespace.

```bash
$ curl -X DELETE \
  -H "Content-Type: application/json" http://localhost:9000/api/v1/namespaces/shield 
{
  "kind": "Namespace",
  "apiVersion": "v1",
  "metadata": {
    "name": "shield",
    <Snip>
  },
  "spec": {
    "finalizers": [
      "kubernetes"
    ]
  },
  "status": {
    "phase": "Terminating"
  }
}
```

The expected response structure shows that the Namespace enters a "Terminating" phase with Kubernetes finalizers in place to ensure proper cleanup.

### ❓ What HTTP method would be used to delete a Kubernetes resource via curl?
The DELETE HTTP method would be used to delete a Kubernetes resource via curl.

### ❓ Explain the purpose of the following Git command sequence for preparing the lab environment from the book's GitHub repository.
The following Git commands prepare the lab environment by cloning the repository, fetching the latest changes, checking out the 2025 branch, and navigating to the configmaps directory:

```bash
$ git clone https://github.com/nigelpoulton/TKB.git
Cloning into 'TKB'...

$ cd TKB

$ git fetch origin

$ git checkout -b 2025 origin/2025

$ cd configmaps
```

### ❓ What are the two data sources available when creating ConfigMaps imperatively with kubectl create configmap, and explain what the following kubectl create configmap command does including how it handles the multi-word literal value?
You create ConfigMaps imperatively with the kubectl create configmap command. However, you can shorten configmap to cm, and the command accepts two sources of data:

- Literal values on the command line (--from-literal)
- From a file (--from-file)

The following command creates a ConfigMap called testmap1 and populates it with two entries from command-line literals:

```bash
$ kubectl create configmap testmap1 \
  --from-literal shortname=SAFC \
  --from-literal longname="Sunderland Association Football Club"
```

The multi-word literal value "Sunderland Association Football Club" is handled by enclosing it in quotes to preserve it as a single value.


## YAML Configuration Submission

## Content

### ❓ What does the following kubectl apply command do when deploying a Pod and what two authentication and communication processes occur?
The `kubectl apply -f pod.yml` command deploys the Pod. The command sends the file to the API server defined in the current context of your kubeconfig file. It also authenticates the request using credentials from your kubeconfig file.

```bash
$ kubectl apply -f pod.yml
pod/hello-pod created
```

### ❓ What is the purpose of the 'kind' and 'apiVersion' fields in Kubernetes YAML files according to the StorageClass documentation?
As with all Kubernetes YAML files, `kind` and `apiVersion` tell Kubernetes the type and version of the object you're defining.

### ❓ What YAML syntax is used to define multiple Kubernetes objects in a single file?
You can define all three objects in the same YAML file by separating them with three dashes (---). The following YAML snippet defines a Pod, a PVC, and an SC. You can define all three objects in the same YAML file by separating them with three dashes (---).

### ❓ Explain what happens when you run the full command sequence to prepare the Git repository for the storage demos.
The command sequence prepares the Git repository for working with the storage demos by:

```bash
$ git clone https://github.com/nigelpoulton/TKB.git
<Snip>

$ cd TKB

$ git fetch origin

$ git checkout -b 2025 origin/2025

$ cd storage
```

This sequence:
1. Clones the book's GitHub repository to your local machine
2. Changes directory into the cloned repository
3. Fetches the latest updates from the origin remote
4. Creates and switches to a new local branch called "2025" that tracks the remote "origin/2025" branch
5. Changes directory into the "storage" folder where all storage-related demo files are located

This ensures you're working with the correct branch (2025) and in the proper directory structure for the storage demonstrations.

### ❓ What command should Windows users modify and how to create a ConfigMap called testmap1 with two literal entries?
Windows users should modify the command by replacing the backslashes with backticks at the end of the first two lines. To create a ConfigMap called testmap1 with two literal entries, run the following command:

```bash
$ kubectl create configmap testmap1 \
  --from-literal shortname=SAFC \
  --from-literal longname="Sunderland Association Football Club"
```

### ❓ What are the requirements for creating a ConfigMap from a file and what command is used?
To create a ConfigMap from a file, you need to use the `--from-file` flag and run the command from the folder containing the file. The file should contain the content you want to store. The command to create a ConfigMap called testmap2 from a file called cmfile.txt is:

```bash
$ kubectl create cm testmap2 --from-file cmfile.txt
configmap/testmap2 created
```

### ❓ What are the key differences in YAML structure for declarative ConfigMap creation and how are they deployed?
The key difference in YAML structure for declarative ConfigMap creation is that ConfigMaps have a `data` subresource instead of the usual `spec` subresource found in typical Kubernetes objects. Here's an example of a ConfigMap called multimap with multiple entries:

```bash
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: multimap 
data:
  firstname: Nigel
  lastname: Poulton
```

To deploy a ConfigMap from a YAML file, use the `kubectl apply` command:

```bash
$ kubectl apply -f fullname.yml
configmap/multimap created
```

A ConfigMap YAML with a single entry might appear more complex than one with multiple entries because the single value entry can contain an entire configuration file, making it look more complicated even though it's actually simpler in structure.

### ❓ Explain what the following YAML configuration does and how it creates a ConfigMap with a single complex entry, and what is the purpose of the pipe character (|) in the ConfigMap YAML and how does it affect how Kubernetes treats the subsequent content?
The YAML configuration creates a ConfigMap called "test-config" with a single complex entry. The key is "test.conf" and the value contains an entire configuration file with multiple lines. The pipe character (|) after the key property tells Kubernetes to treat everything after the pipe as a single value. This allows the ConfigMap to store multi-line configuration content as a single entry rather than multiple separate entries.

```bash
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: test-config
data:
  test.conf: |           <<---- Key
    env = plex-test      -----┐
    endpoint = 0.0.0.0:31001  |
    char = utf8               | Value
    vault = PLEX/test         |
    log-size = 512M      -----┘
```

When deployed, this creates a ConfigMap called "test-config" with a single complex map entry where the key is "test.conf" and the value contains the entire multi-line configuration content.

### ❓ What commands are used to deploy a Pod from a YAML file containing ConfigMap environment variable mappings and to create a Pod that executes a startup command using ConfigMap-derived environment variables?
To deploy a Pod from a YAML file containing ConfigMap environment variable mappings:

```bash
$ kubectl apply -f podenv.yml
pod/envpod created
```

To create a Pod that executes a startup command using ConfigMap-derived environment variables:

```bash
$ kubectl apply -f podstartup.yml
pod/startup-pod created
```

### ❓ What command is used to deploy a Pod from a YAML file, what output indicates successful creation, and what command deploys a Pod that uses a Secret volume?
The command used to deploy a Pod from a YAML file is:

```bash
$ kubectl apply -f podvol.yml
pod/cmvol created
```

The output "pod/cmvol created" indicates successful creation of the Pod.

For deploying a Pod that uses a Secret volume, the command is:

```bash
$ kubectl apply -f secretpod.yml
pod/secret-pod created
```

This command causes Kubernetes to transfer the unencrypted Secret over the network to the kubelet on the node running the Pod. From there, the container runtime mounts it into the container using a tmpfs mount.

### ❓ Explain what the following full command sequence does to create a Kubernetes Secret called 'creds'.
The following command creates a new Secret called 'creds' with two literal values - a username and password:

```bash
$ kubectl create secret generic creds \
  --from-literal user=nigelpoulton \
  --from-literal pwd=Password123
```

This creates a generic Secret resource named 'creds' containing the key-value pairs user=nigelpoulton and pwd=Password123.

### ❓ What does the kubectl apply command accomplish with the tkb-secret.yml file?
The kubectl apply command deploys the Secret defined in the tkb-secret.yml file to your cluster:

```bash
$ kubectl apply -f tkb-secret.yml 
secret/tkb-secret created
```

This command creates the tkb-secret Secret resource in the Kubernetes cluster based on the YAML configuration file.

### ❓ What does the kubectl apply command do when deploying the secret-pod?
The command deploys the Pod defined in the secretpod.yml file:

```bash
$ kubectl apply -f secretpod.yml
pod/secret-pod created
```

### ❓ What does the following full command sequence accomplish in preparing the Git repository for the StatefulSet deployment?
```bash
$ git clone https://github.com/nigelpoulton/TKB.git

$ cd TKB

$ git fetch origin

$ git checkout -b 2025 origin/2025
```

This command sequence prepares the Git repository for the StatefulSet deployment by:

1. Cloning the book's GitHub repository from `https://github.com/nigelpoulton/TKB.git`
2. Changing into the TKB directory
3. Fetching the latest changes from the origin remote
4. Creating and switching to a new local branch called "2025" that tracks the remote "origin/2025" branch

This ensures you have the correct version of the code and configuration files needed for the StatefulSet deployment exercises.

### ❓ Explain the concept of serialization as it relates to Kubernetes objects and why it's necessary for both network transmission and storage, and compare and contrast JSON and Protobuf serialization schemas in terms of their performance characteristics and use cases within Kubernetes.
Serialization is the process of converting an object into a string or stream of bytes so it can be sent over a network and persisted to a data store. The reverse process of converting a string or stream of bytes into an object is deserialization.

Kubernetes serializes objects, such as Pods and Services, as JSON strings and sends them over the network via HTTP. The process happens in both directions:

- Clients like kubectl serialize objects when posting them to the API server
- The API server serializes responses back to clients
- Kubernetes also serializes objects for storage in the cluster store

Serialization is necessary because:
1. For network transmission: Objects need to be converted to a format that can be transmitted over HTTP
2. For storage: Objects need to be converted to a format that can be persisted in the cluster store

Kubernetes supports JSON and Protobuf serialization schemas:

| Characteristic | JSON | Protobuf |
|----------------|------|----------|
| Speed | Slower | Faster |
| Efficiency | Less efficient | More efficient |
| Scalability | Scales worse | Scales better |
| Inspectability | Easy to inspect and troubleshoot | Harder to inspect and troubleshoot |
| Current Usage | Typically used with external clients | Typically used with internal cluster components |

At the time of writing, Kubernetes typically communicates with external clients via JSON but uses Protobuf when communicating with internal cluster components. When clients send requests to the API server, they use the Accept header to list the serialization schemas they support, and Kubernetes honors this preference.

### ❓ Explain the structure and purpose of the following JSON namespace definition.
The following JSON defines a new Namespace object called shield:

```json
{
  "kind": "Namespace",
  "apiVersion": "v1",
  "metadata": {
    "name": "shield",
    "labels": {
      "chapter": "api"
    }
  }
}
```

This JSON structure specifies that it's creating a Namespace resource using API version v1, with the name "shield" and a label "chapter: api".

### ❓ What types of configuration files beyond application code should be scanned for security vulnerabilities?
Beyond application code, you should scan your Dockerfiles, Kubernetes YAML files, Helm charts, and other configuration files for security vulnerabilities. Scanning app code for vulnerabilities is widely accepted as good production hygiene, but scanning configuration files is less widely adopted.

### ❓ What is the primary purpose of reviewing and scanning infrastructure-as-code configuration files in the context of Kubernetes security?
The content states that reviewing and scanning infrastructure-as-code configuration files is part of an image promotion workflow in the software delivery pipeline for Kubernetes security. However, it notes that "the list isn't supposed to represent an exact workflow" and there are "more things you can do," indicating this is one of multiple security practices rather than having a single primary purpose defined in the provided content.

### ❓ Explain the structure and content of the following ConfigMap YAML definition with three key-value pairs, and analyze another ConfigMap YAML that contains a complete configuration file as a value, explaining how the pipe character facilitates multi-line content.
Here's an example of a ConfigMap with three entries:

```bash
kind: ConfigMap
apiVersion: v1
metadata:
  name: epl
data:
  Competition: Premier League    -----┐
  Season: 2024-2025                   | 3 x map entries (key:value pairs)
  Champions: Liverpool           -----┘
```

Here's an example where the value is a complete configuration file:

```bash
kind: ConfigMap
apiVersion: v1
metadata:
  name: cm2
data:                       
  test.conf: |                 <<---- Key
    env = plex-test            -----┐
    endpoint = 0.0.0.0:31001        |
    char = utf8                     | Value
    vault = PLEX/test               |
    log-size = 512M            -----┘
```

The pipe character (`|`) after the key property tells Kubernetes to treat everything after the pipe as a single value, allowing for multi-line configuration files.

