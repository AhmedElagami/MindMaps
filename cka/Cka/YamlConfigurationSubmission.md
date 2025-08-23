# YAML Configuration Submission

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

