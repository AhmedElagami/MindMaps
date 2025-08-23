# CRUD Operations Processing

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

