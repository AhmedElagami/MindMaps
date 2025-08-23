# API Group Evolution

## Content

### ❓ How has the Kubernetes API evolved from its original monolithic structure to its current organization and what does Figure 15.3 illustrate about this organization?
When Kubernetes was new, the API was monolithic and all resources existed in a single global namespace. However, as Kubernetes grew, we split the API into smaller, more manageable groups that we call named groups or sub-groups. Figure 15.3 shows a simplified view of the API with resources divided into groups.

![Figure 15.3 - Simplified view of Kubernetes API](media/figure15-3.png)

The image shows the API with four groups. There are lots more than four, but I'm keeping the picture simple.

### ❓ What is the primary way network and storage vendors expose advanced features through the Kubernetes API?
The primary way network and storage vendors expose advanced features through the Kubernetes API is by extending Kubernetes by adding their own resources and controllers. This is a popular way for network and storage vendors to expose advanced features, such as snapshot schedules or IP address management, via the Kubernetes API.

### ❓ Explain how the storage example demonstrates the division between built-in Kubernetes resources and custom API extensions for advanced features.
In the storage example, volumes are surfaced inside of Kubernetes via CSI drivers, Pods consume them via built-in Kubernetes resources such as StorageClasses and PersistentVolumeClaims, but advanced features such as snapshot scheduling can be managed via custom API resources and controllers. This demonstrates the division where basic functionality is handled by built-in Kubernetes resources, while advanced vendor-specific features are managed through custom API extensions.

### ❓ What are the two main components required to extend the Kubernetes API according to the high-level pattern?
The high-level pattern for extending the API involves two main things:

1. Create your custom resource
2. Write and deploy your custom controller

### ❓ What does the CustomResourceDefinition (CRD) object enable you to create within the Kubernetes API?
The CustomResourceDefinition (CRD) object enables you to create new API resources that look, smell, and feel like native Kubernetes resources. It lets you create new API resources that integrate seamlessly with the Kubernetes ecosystem.

### ❓ How do custom resources integrate with standard Kubernetes tooling and workflows?
Custom resources integrate with standard Kubernetes tooling and workflows by allowing you to create your custom resource as a CRD and then use kubectl to create instances and inspect them just like you do with native resources. This allows developers and Kubernetes operators to deploy and manage everything via well-understood API interfaces and standard tools such as kubectl and YAML files.

### ❓ What REST API path structure do custom resources receive within the Kubernetes API?
Custom resources receive their own REST paths in the API. For example, a custom resource called "books" in the API group "nigelpoulton.com/v1" would be served on the REST path: apis/nigelpoulton.com/v1/books/

### ❓ Analyze the following full CustomResourceDefinition YAML and explain the purpose of each annotated section including API group, scope, naming conventions, and version management.
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: books.nigelpoulton.com
spec:
  group: nigelpoulton.com      <<---- API sub-group (a.k.a. "named API group")
  scope: Cluster               <<---- Can be "Namespaced" or "Cluster"
  names:
    plural: books              <<---- All resources need a plural and singular name
    singular: book             <<---- Singular names are used on CLI and command outputs
    kind: Book                 <<---- kind property used in YAML files
    shortNames:
    - bk                       <<---- Short name that can be used by kubectl
  versions:                    <<---- Resources can be served by multiple API versions
    - name: v1                  
      served: true             <<---- If set to false, "v1" will not be served
      storage: true            <<---- Store instances of the object as this version
      schema:                  <<---- This block defines the resource's properties
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                <Snip>
```

**Purpose of each annotated section:**

- **API group (nigelpoulton.com)**: Defines the named API group/sub-group where the custom resource will be served
- **Scope (Cluster)**: Determines whether the resource is cluster-scoped or namespaced
- **Naming conventions**: 
  - plural: Used in REST API paths and CLI commands
  - singular: Used in CLI outputs and commands
  - kind: Used in YAML files to specify the resource type
  - shortNames: Short aliases that can be used with kubectl
- **Version management**:
  - Resources can be served by multiple API versions
  - served: Controls whether a version is actively served by the API
  - storage: Determines which version is used for persistent storage
  - schema: Defines the structure and validation rules for the resource's properties

### ❓ What is the purpose of running the full command sequence to clone the Git repository and switch to the correct branch, and what specific directory should you navigate to after cloning?
The purpose of running the full command sequence is to obtain the book's GitHub repository and ensure you're working with the correct branch (2025) that contains the necessary files for the custom controller implementation. After cloning, you should navigate to the `TKB/api` directory where the relevant files are located.

```bash
$ git clone https://github.com/nigelpoulton/TKB.git
<Snip>

$ cd TKB

$ git fetch origin

$ git checkout -b 2025 origin/2025
```

```bash
$ cd TKB/api
```

### ❓ What command is used to deploy the custom resource definition and what confirmation message indicates successful creation?
The command `kubectl apply -f crd.yml` is used to deploy the custom resource definition. The confirmation message `customresourcedefinition.apiextensions.k8s.io/books.nigelpoulton.com created` indicates successful creation.

```bash
$ kubectl apply -f crd.yml
customresourcedefinition.apiextensions.k8s.io/books.nigelpoulton.com created
```

### ❓ What is the REST API path where Kubernetes serves the newly created custom resource?
Kubernetes serves the newly created custom resource on the following REST API path:

```bash
apis/nigelpoulton.com/v1/books/
```

### ❓ Explain what information the two verification commands provide about the newly created Book custom resource, including its shortname, API group, and namespaced status.
The verification commands provide the following information about the Book custom resource:

1. `kubectl api-resources | grep books` shows:
   - Resource name: `books`
   - Shortname: `bk`
   - API group: `nigelpoulton.com/v1`
   - Namespaced status: `false` (meaning it's cluster-scoped)
   - Kind: `Book`

2. `kubectl explain book` confirms:
   - GROUP: `nigelpoulton.com`
   - KIND: `Book`
   - VERSION: `v1`

```bash
$ kubectl api-resources | grep books
NAME      SHORTNAMES     APIGROUP                NAMESPACED     KIND
books     bk             nigelpoulton.com/v1     false          Book

$ kubectl explain book
GROUP:      nigelpoulton.com
KIND:       Book
VERSION:    v1
DESCRIPTION:
    <empty>
FIELDS:
    <Snip>
```

### ❓ Analyze the YAML structure of the Book custom resource and explain how each field in the spec section corresponds to the expected data types and structure defined in the CRD.
The YAML structure defines a Book custom resource instance called "ai" with the following spec fields that match the expected data types and structure defined in the CRD:

- `bookTitle`: String value ("AI Explained")
- `subTitle`: String value ("Facts, Fiction, and Future")
- `topic`: String value ("Artificial Intelligence")
- `edition`: Integer value (1)
- `salesUrl`: String URL value (https://www.amazon.com/dp/1916585388)

Each field in the spec section corresponds to the expected data types defined in the custom resource definition, with string fields containing quoted text values and the edition field containing a numeric value.

```yaml
apiVersion: nigelpoulton.com/v1
kind: Book
metadata:
  name: ai
spec:
  bookTitle: "AI Explained"
  subTitle: "Facts, Fiction, and Future"
  topic: "Artificial Intelligence"
  edition: 1
  salesUrl: https://www.amazon.com/dp/1916585388
```

### ❓ What command deploys the Book custom resource instance and what confirmation message indicates successful creation?
The command `kubectl apply -f book.yml` is used to deploy the Book custom resource instance. The confirmation message `book.nigelpoulton.com/ai created` indicates successful creation.

```bash
$ kubectl apply -f book.yml
book.nigelpoulton.com/ai created
```

### ❓ Interpret the output format of the 'kubectl get bk' command and explain what information each column provides about the Book custom resource instance.
The `kubectl get bk` command output provides the following information about the Book custom resource instance in a tabular format:

- **NAME**: The name of the Book resource instance ("ai")
- **TITLE**: The book title ("AI Explained")
- **SUBTITLE**: The book subtitle ("Facts, Fiction, and Future")
- **EDITION**: The edition number (1)
- **URL**: The sales URL (www.amazon.com/dp/1916585388)

This output displays the actual data stored in the custom resource's spec fields, showing how the custom resource presents its information in a formatted table when queried with the shortname "bk".

```bash
$ kubectl get bk
NAME    TITLE           SUBTITLE                      EDITION    URL
ai      AI Explained    Facts, Fiction, and Future    1          www.amazon.com/dp/1916585388
```

### ❓ What does the following command sequence do to start a proxy and query the custom API group, and what are all the available operations (verbs) that can be performed on the Book resource based on the API response?
The command sequence starts a kubectl proxy on port 9000 and then queries the custom API group to list available resources. The first command starts the proxy in the background:

```bash
$ kubectl proxy --port 9000 &
[1] 14784
Starting to serve on 127.0.0.1:9000
```

The second command queries the custom API group at nigelpoulton.com/v1:

```bash
$ curl http://localhost:9000/apis/nigelpoulton.com/v1/
{
  "kind": "APIResourceList",
  "apiVersion": "v1",
  "groupVersion": "nigelpoulton.com/v1",
  "resources": [
    {
      "name": "books",
      "singularName": "book",
      "namespaced": false,
      "kind": "Book",
      "verbs": [
        "delete",
        "deletecollection",
        "get",
        "list",
        "patch",
        "create",
        "update",
        "watch"
      ],
      "shortNames": [
        "bk"
      ],
      "storageVersionHash": "F2QdXaP5vh4="
    }
  ]
}
```

Based on the API response, the available operations (verbs) that can be performed on the Book resource are: delete, deletecollection, get, list, patch, create, update, and watch.

### ❓ Why are custom controllers necessary for custom resources to be useful, according to the text?
Custom controllers are necessary because "custom resources don't do anything useful until you create a custom controller to do something with them." The text explains that while custom resources are interesting, they require custom controllers to actually perform useful functions.

### ❓ What are the four specific resources mentioned that need to be cleaned up after following the chapter exercises?
The four specific resources that need to be cleaned up are:
1. A process
2. An ai book resource
3. A custom resource (CRD)

### ❓ Analyze the following command output to identify the process ID and command for the kubectl proxy running on both Linux/Mac and Windows systems.
The commands demonstrate how to identify the kubectl proxy process on different operating systems:

```bash
// Linux and Mac command

$ ps | grep kubectl
PID     TTY       TIME      CMD
27533   ttys001   0:03.13   kubectl proxy --port 9000

// Windows command

> tasklist | Select-String -Pattern 'kubectl'
Image Name       PID  Session Name   Session#
=============  =====  ============   ========
kubectl.exe    19776  Console               1
```

On Linux/Mac systems:
- Process ID (PID): 27533
- Command: kubectl proxy --port 9000

On Windows systems:
- Process ID (PID): 19776
- Command: kubectl.exe (running as a process)

### ❓ What do the following kill commands demonstrate about terminating the kubectl proxy process on different operating systems?
The commands demonstrate the different methods for terminating the kubectl proxy process on various operating systems:

```bash
// Linux and Mac command

$ kill -9 27533
[1]  + 27533 killed     kubectl proxy --port 9000

// Windows command

> taskkill /F /PID 19776
SUCCESS: The process with PID 19776 has been terminated.
```

On Linux/Mac systems, the `kill -9` command is used with the process ID to forcefully terminate the process. On Windows systems, the `taskkill /F /PID` command is used to forcefully terminate the process by its process ID. Both commands show successful termination of the kubectl proxy process.

### ❓ Explain what the command 'kubectl delete book ai' accomplishes based on the output shown.
The command `kubectl delete book ai` deletes the "ai" book object from the cluster:

```bash
$ kubectl delete book ai
book.nigelpoulton.com "ai" deleted
```

The output confirms that the book resource named "ai" in the nigelpoulton.com API group has been successfully deleted from the cluster.

### ❓ What does the command 'kubectl delete crd books.nigelpoulton.com' remove from the cluster?
The command `kubectl delete crd books.nigelpoulton.com` removes the CustomResourceDefinition (CRD) from the cluster:

```bash
$ kubectl delete crd books.nigelpoulton.com
customresourcedefinition.apiextensions.k8s.io "books.nigelpoulton.com" deleted
```

The output confirms that the CustomResourceDefinition named "books.nigelpoulton.com" has been successfully deleted from the cluster, which removes the custom resource definition itself from the API.

### ❓ How does Kubernetes enable third-party technologies to extend its API and what mechanism makes external resources appear native?
Finally, the Kubernetes API is becoming the de facto cloud API, with many third-party technologies extending it so they can expose their own technologies through it. Kubernetes makes it easy to extend the API through CustomResourceDefinitions that make 3rd-party resources look and feel like native Kubernetes resources.

