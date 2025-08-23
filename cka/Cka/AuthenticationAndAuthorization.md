# Authentication and Authorization

## Content

### ❓ Where is the kubeconfig file located on different operating systems and what is its standard filename?
Behind the scenes, kubectl reads your kubeconfig file to know which cluster to send commands to and which credentials to authenticate with. Your kubeconfig file is called **config** and lives in the following hidden directories depending on your system (you'll need to configure your system to show hidden folders):

- **macOS**: ~/.kube/
- **Windows**: %USERPROFILE%.kube\

### ❓ What does the following command sequence do to merge multiple kubeconfig files?
The following command sequence merges configurations from multiple kubeconfig files and displays the merged configuration:

```bash
$ export KUBECONFIG=~/.kube/config-bkp:~/.kube/tkb-kubeconfig.yaml

$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://127.0.0.1:54225
<Snip>
```

This command sets the KUBECONFIG environment variable to include both the backup configuration file (config-bkp) and the LKE kubeconfig file (tkb-kubeconfig.yaml), then uses `kubectl config view` to display the merged configuration. If you look closely, you should see your LKE cluster's details listed in the clusters, contexts, and users sections.

### ❓ Explain the purpose and outcome of running the kubectl config view --flatten command in the context provided.
The `kubectl config view --flatten` command exports the merged configuration into a new kubeconfig file called config and then sets the KUBECONFIG environment variable to use the new file:

```bash
$ kubectl config view --flatten >  ~/.kube/config

$ export KUBECONFIG=~/.kube/config
```

This command takes the merged configuration from multiple kubeconfig files (previously set via KUBECONFIG environment variable) and flattens it into a single, clean kubeconfig file. The `--flatten` option removes any redundant or duplicate entries, creating a consolidated configuration file that contains all the necessary cluster, context, and user information in a single file.

### ❓ Explain the purpose and output of the following kubectl config get-contexts command sequence, including how to interpret the CURRENT and NAME columns.
```bash
$ kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO          NAMESPACE
*         docker-desktop   docker-desktop   docker-desktop
          lke349416-ctx    lke349416        lke349416-admin   default

$ kubectl config use-context lke349416-ctx
Switched to context "lke349416-ctx".
```

The `kubectl config get-contexts` command lists all available contexts in your kubeconfig file:
- **CURRENT column**: Shows which context is currently active (marked with *)
- **NAME column**: The friendly name of each context
- **CLUSTER column**: The cluster each context points to
- **AUTHINFO column**: The user credentials each context uses
- **NAMESPACE column**: The default namespace for each context

The second command `kubectl config use-context lke349416-ctx` switches the current context to the LKE cluster context, making it the active context for all subsequent kubectl commands.

### ❓ What are the three actions that kubectl performs every time a command is executed?
Every time you execute a kubectl command it does the following three things:

1. Converts the command into an HTTP REST request
2. Sends the request to the Kubernetes cluster defined in the current context of your kubeconfig file
3. Uses the credentials specified in the current context of your kubeconfig file

### ❓ What four key components does a kubeconfig file define, and where is this file typically located?
Your kubeconfig file is called config and lives in your home directory's hidden .kube folder. It defines:

- Clusters
- Users (credentials)
- Contexts
- Current context

### ❓ What are the three main components defined in the simple kubeconfig file described, and how are they related in the context?
The three main components defined in the simple kubeconfig file are:

1. **Clusters**: Defines all known Kubernetes clusters. In the example, there's a single cluster called "shield" with its API endpoint at https://192.168.1.77:8443 and certificate authority data.

2. **Users**: Contains user authentication credentials. In the example, there's a single user called "coulson" with client certificate data and client key data for authentication.

3. **Contexts**: Combines clusters and users to define where commands should be sent and with which credentials. In the example, there's a context called "director" that pairs the "shield" cluster with the "coulson" user credentials.

These components work together where the context specifies which cluster to send commands to and which user credentials to use for authentication. The "current-context" field determines which context kubectl will use by default.

### ❓ Explain what each section of the following YAML kubeconfig file defines and how they work together to configure kubectl access.
Here is the detailed explanation of each section in the kubeconfig YAML file:

```yaml
apiVersion: v1
kind: Config
clusters:                                    <<---- All known clusters are listed in this block
- name: shield                               <<---- Friendly name for a cluster
  cluster:
    server: https://192.168.1.77:8443        <<---- Cluster's AIP endpoint (API server)
    certificate-authority-data: LS0tLS...    <<---- Cluster's certificate
users:                                       <<---- Users are in this block
- name: coulson                              <<---- Friendly name (not used by Kubernetes)
  user:
    client-certificate-data: LS0tLS1CRU...   <<---- User certificate/credentials
    client-key-data: LS0tLS1CRUdJTiBFQyB     <<---- User private key
contexts:                                    <<---- List of contexts (cluster:user pairs)
- context:                                 
  name: director                             <<---- Context called "director"
    cluster: shield                          <<---- Send commands to this cluster
    user: coulson                            <<---- Authenticate with these credentials
current-context: director                    <<---- kubectl will use this context
```

**How they work together:**
- The **clusters** block defines the Kubernetes clusters you can connect to, including their API server endpoints and security certificates.
- The **users** block contains authentication credentials (certificates and keys) for different users.
- The **contexts** block combines specific clusters with specific users to create named contexts that define where commands go and how to authenticate.
- The **current-context** field specifies which context should be used by default when running kubectl commands.

This configuration allows kubectl to know which cluster to communicate with and which credentials to use for authentication.

### ❓ What does the output of 'kubectl config current-context' indicate about the system's current Kubernetes configuration?
The output of `kubectl config current-context` indicates which context is currently active in the Kubernetes configuration. In the example:

```bash
$ kubectl config current-context
docker-desktop
```

This shows that the system is configured to send kubectl commands to the cluster and use the credentials defined in the "docker-desktop" context. The current context determines which cluster kubectl will communicate with and which user credentials it will use for authentication.

### ❓ How do Secrets differ from ConfigMaps in their intended use case and what are the security considerations for a comprehensive secrets management system?
Secrets differ from ConfigMaps in their intended use case:

**Secrets** are designed to work as part of a more comprehensive secrets management solution for storing sensitive data such as passwords, certificates, and OAuth tokens.

**ConfigMaps** are for general configuration data that is not sensitive.

However, a secure secrets management system involves much more than just using Kubernetes Secrets. You have to consider all of the following security requirements:

- Encryption of secrets while at rest in the cluster store
- Encryption of secrets while in flight on the network
- Protection of secrets when surfaced in nodes/Pods/containers
- Controlling API access to secrets via least privilege RBAC
- Controlling access to etcd nodes (cluster store)
- Preventing privileged containers from accessing secrets
- Preventing exposure via source code repositories like GitHub
- Securely deleting secrets when no longer in use

Most new Kubernetes clusters do none of these things by default - they store Secrets unencrypted in the cluster store, send them over the network in plain text, and mount them in containers as plain text.

### ❓ What are the two primary ways that Kubernetes handles secrets in an unsecured manner by default, what configuration option exists to encrypt Secrets at rest, and what are three security measures mentioned to enhance Kubernetes Secrets security?
By default, Kubernetes handles secrets in an unsecured manner by:

1. Sending them over the network in plain text
2. Mounting them in containers as plain text

However, you can configure objects to encrypt Secrets at rest in the cluster store.

Three security measures mentioned to enhance Kubernetes Secrets security include:

1. Deploy a service mesh to encrypt all network traffic
2. Configure strong RBAC to secure API access to Secrets
3. You can even restrict access to control plane nodes and etcd nodes

Additional security measures mentioned include securely deleting secrets after use and implementing other security practices.

### ❓ What is a common real-world practice for securing secrets instead of relying solely on Kubernetes Secrets and what native Kubernetes component exists for integrating with external vaults?
A common real-world practice for securing secrets is to store secrets outside of Kubernetes in 3rd party vaults such as HashiCorp's Vault or similar cloud services.

Kubernetes even has a native Secrets Store CSI Driver for integrating with external vaults.

### ❓ Explain the typical Kubernetes secrets workflow when only using native Secrets, from creation to application consumption, and how does the container runtime handle Secrets when mounting them into containers?
A typical secrets workflow that only uses Kubernetes Secrets looks like this:

1. You create the Secret which Kubernetes persists to the cluster store as an un-encrypted object
2. You schedule a Pod with a container that uses the Secret
3. Kubernetes transfers the un-encrypted Secret over the network to the node running the Pod
4. The kubelet on the node starts the Pod and its containers
5. The container runtime mounts the Secret into the container via an in-memory tmpfs filesystem and decodes it from base64 to plain text
6. The application consumes it

When you delete the Pod, Kubernetes deletes the copy of the Secret on the node (it keeps the copy in the cluster store).

### ❓ What does Figure 12.5 illustrate about injecting Secrets at runtime across different environments?
Figure 12.5 shows a single image configured with three different Secrets for three different environments. Each of the Secrets could contain the TLS certificates for the specific environment. Kubernetes loads the appropriate Secret into each container at run time.

![Figure 12.5 - Injecting Secrets at run time](media/figure12-5.png)

### ❓ What does the base64 decoding command demonstrate about the security of base64 encoding?
The base64 decoding command demonstrates that base64 encoding is not secure because it can be decoded without a key:

```bash
$ echo UGFzc3dvcmQxMjM= | base64 -d
Password123
```

The decoding completes successfully without a key, proving that base64 encoding is not secure. As such, you should never store Secrets on platforms like GitHub, as anyone with access can read them.

### ❓ List the security considerations mentioned for a complete secrets management solution beyond basic Kubernetes Secrets storage and where most real-world solutions typically store secret data.
Remember, a complete secrets management solution involves a lot more than just storing sensitive data in Kubernetes Secrets. You need to:

- Encrypt them at rest
- Encrypt them in flight
- Control API access
- Restrict etcd node access
- Handle privileged containers
- Prevent Secret manifest files from being hosted on public source control repos
- And more

Most real-world solutions store secret data outside of Kubernetes in a 3rd-party vault.

### ❓ What types of data are ConfigMaps specifically designed to handle and what is the intended use case for Secrets and how do they fit into a broader security strategy?
ConfigMaps are specifically designed for "application configuration parameters and even entire configuration files." This includes non-sensitive configuration data such as:
- Environment-specific settings
- Configuration files (JSON, XML, YAML, properties files, etc.)
- Application parameters and flags
- Any other non-sensitive configuration data

Secrets, on the other hand, are "for sensitive data and intended for use as part of a wider secrets management solution." Their intended use case includes:
- Storing sensitive information like passwords, API keys, tokens, and certificates
- Serving as one component in a comprehensive secrets management strategy
- Providing a basic mechanism for sensitive data storage within Kubernetes

However, it's important to note that Secrets are designed to be part of a broader security solution rather than a complete secrets management system on their own.

### ❓ What security limitations should be considered when using Kubernetes Secrets regarding encryption?
When using Kubernetes Secrets, there is an important security limitation to consider: "Kubernetes does not automatically encrypt Secrets in the cluster store or while transmitting them over the network." 

This means that:
- Secrets are stored in etcd (the Kubernetes cluster store) in base64-encoded but unencrypted form by default
- Secrets transmitted over the network between components are not automatically encrypted
- Additional security measures must be implemented for proper encryption at rest and in transit

This limitation highlights why Secrets should be used as part of a wider secrets management solution that includes proper encryption, access controls, and auditing capabilities rather than relying solely on Kubernetes' built-in Secret functionality for comprehensive security.

### ❓ What does the diagram in Figure 14.1 illustrate about the flow of a typical API request through Kubernetes security checks?
Figure 14.1 illustrates the flow of a typical API request as it passes through the various security-related checks in Kubernetes. The diagram shows:

![Figure 14.1](media/figure14-1.png)

The flow demonstrates that all requests to the API server must pass through three main security checks, regardless of where the request originates (operators using kubectl, Pods, Kubelets, control plane services, or Kubernetes-native apps):

1. **Authentication (authN)**: The first checkpoint where the request's credentials are verified to prove identity
2. **Authorization (authZ)**: The second checkpoint where authenticated requests are checked for permission to perform specific actions
3. **Admission Control**: The final checkpoint where policies are enforced on requests that attempt to modify the cluster

The flow is the same no matter where the request originates. authN is short for authentication, whereas authZ is short for authorization.

The diagram illustrates the sequential nature of these security checks - a request must pass authentication before moving to authorization, and must pass authorization before moving to admission control. Only after passing all three checks is the request executed on the cluster.

### ❓ Interpret the full kubeconfig YAML structure below, explaining the purpose of each major section and how they work together for cluster authentication.
A kubeconfig file is structured with four top-level sections that work together to authenticate and authorize access to Kubernetes clusters. Here's the complete YAML structure with explanations:

```yaml
apiVersion: v1
kind: Config
clusters:                        <<---- Cluster block defining one or more clusters and certs
- cluster:                  
  name: prod-shield              <<---- This block defines a cluster called "prod-shield"
    server: https://<url-or-ip-address-of-api-server>:443     <<---- This is the cluster's URL
    certificate-authority-data: LS0tLS1C...LS0tCg==           <<---- Cluster's certificate
 users:                          <<---- Users block defining one or more users and credentials
- name: njfury                   <<---- User called njfury
  user:
    as-user-extra: {}
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6IlZwMzl...SZY3uUQ     <<---- User's credentials
contexts:                        <<---- Context block. A context is a cluster + user
- context:
  name: shield-admin             <<---- This block defines a context called "shield-admin"
    cluster: prod-shield         <<---- Cluster
    user: njfury                 <<---- User  
    namespace: default
current-context: shield-admin    <<---- Context used by kubectl
```

The four top-level sections are:

1. **Clusters**: Defines one or more Kubernetes clusters. Each cluster has a friendly name, an API server endpoint, and the public key of its certificate authority (CA).

2. **Users**: Defines one or more users. Each user requires a name and token. The token is often an X.509 certificate signed by the cluster's CA (or a CA trusted by the cluster).

3. **Contexts**: A list of user and cluster pairs. A context combines a cluster and user to define where commands should be sent and which credentials to use.

4. **Current-context**: Specifies which context should be used by kubectl commands.

These sections work together by: the current-context determines which context to use, the context specifies which cluster to connect to and which user to authenticate as, the cluster section provides the API server endpoint and CA certificate for TLS verification, and the user section provides the authentication credentials. The cluster's authentication module then validates the user's credentials, and if successful, requests progress to authorization.

### ❓ What three pieces of information does each cluster definition in the clusters section contain and what type of authentication token is typically used for users?
Each cluster definition in the clusters section contains three key pieces of information:

1. A friendly name for the cluster
2. An API server endpoint (URL or IP address)
3. The public key of its certificate authority (CA)

For user authentication, the token is typically an X.509 certificate signed by the cluster's CA (or a CA trusted by the cluster). This certificate-based authentication ensures that the cluster can validate the user's credentials using its trusted certificate authority.

### ❓ Explain what the current-context setting determines for kubectl commands and what happens to API requests after successful authentication?
The current-context setting in a kubeconfig file determines which cluster and user combination should be used for kubectl commands. In the provided YAML snippet:

```bash
current-context: shield-admin    <<---- Current context
contexts:                        
- context:
  name: shield-admin             <<---- This block defines a context called "shield-admin"
    cluster: prod-shield         <<---- Send commands to this cluster 
    user: njfury                 <<---- User to authenticate as
    namespace: default
```

The current-context: shield-admin setting means all kubectl commands will be sent to the prod-shield cluster using the njfury user credentials for authentication.

After successful authentication, the cluster's authentication module determines whether the user is genuine. If your cluster integrates with an external IAM system, it'll hand off authentication to that system. Assuming authentication is successful, requests progress to the authorization phase where RBAC determines what actions the user is permitted to perform.

### ❓ Analyze the following RBAC permissions table and explain what specific actions each user is authorized to perform on which resources.
The RBAC permissions table shows specific authorization examples for different users:

| User (subject) | Action | Resource | Effect |
| --- | --- | --- | --- |
| Bao | create | Pods | Bao can create Pods |
| Kalila | list | Deployments | Kalila can list Deployments |
| Josh | delete | ServiceAccounts | Josh can delete ServiceAccounts |

- **Bao** is authorized to **create** Pod objects
- **Kalila** is authorized to **list** Deployment objects  
- **Josh** is authorized to **delete** ServiceAccount objects

This table demonstrates how RBAC rules define which users can perform specific actions on particular Kubernetes resources.

### ❓ What are the two key characteristics of RBAC in Kubernetes as described in the text and explain the fundamental difference between allow rules and deny rules in Kubernetes RBAC implementation?
RBAC in Kubernetes has two key characteristics: it is enabled on most Kubernetes clusters and it is a least-privilege deny-by-default system. This means RBAC locks everything down by default, and you need to create allow rules to open things up.

The fundamental difference between allow and deny rules is that Kubernetes doesn't support deny rules - it only supports allow rules. This design choice makes implementation and troubleshooting much simpler because you only define what is permitted rather than having to manage both allow and deny rules that could potentially conflict with each other.

### ❓ What are the two vital concepts for understanding Kubernetes RBAC according to the text and explain the relationship between Roles and RoleBindings in Kubernetes RBAC?
Two concepts are vital to understanding Kubernetes RBAC: Roles and RoleBindings.

Roles define a set of permissions, and RoleBindings bind them to users. This means Roles specify what actions can be performed on which resources, while RoleBindings connect those permissions to specific users, determining who gets those permissions.

### ❓ Analyze the following YAML Role definition and explain what specific permissions it grants, including the namespace, allowed actions, and target resources.
The YAML Role definition grants specific read permissions on Deployment resources:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: shield
  name: read-deployments
rules:
- verbs: ["get", "watch", "list"]   <<---- Allowed actions
  apiGroups: ["apps"]                ----┐ on resources 
  resources: ["deployments"]         ----┘ of this type
```

This Role grants permission to:
- **Namespace**: shield
- **Allowed actions**: get, watch, and list (read operations)
- **Target resources**: Deployment objects
- **API group**: apps

The Role called "read-deployments" specifically allows read access operations against Deployment objects in the shield namespace.

### ❓ What is required for a Role to become effective in granting permissions to users and what specific requirement must the username in a RoleBinding meet to be effective?
Roles don't do anything until you bind them to users. A Role only defines permissions but doesn't grant them to anyone until it's associated with users through a RoleBinding.

The username listed in the RoleBinding must be a string and has to match an authenticated user. This means the name specified in the RoleBinding must correspond to an actual authenticated user in the Kubernetes system for the binding to be effective.

### ❓ Explain what the following RoleBinding YAML accomplishes, including which user it binds to which role and in what namespace.
The RoleBinding YAML binds a specific Role to a user in a particular namespace:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-deployments
  namespace: shield
subjects:
- kind: User
  name: sky                   <<---- Name of the authenticated user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: read-deployments      <<---- Bind this Role to the authenticated user above
  apiGroup: rbac.authorization.k8s.io
```

This RoleBinding accomplishes:
- **Binds user**: sky (an authenticated user)
- **To Role**: read-deployments
- **In namespace**: shield
- **Result**: The authenticated user called sky gains the permissions defined in the read-deployments Role within the shield namespace

Deploying both the Role and RoleBinding objects to your cluster will allow an authenticated user called sky to run commands that require read access to Deployments.

### ❓ What are the three properties defined in the rules section of a Role object and how do the verbs, apiGroups, and resources fields work together to define permissions in a Role?
Role objects have a rules section that defines three properties: verbs, apiGroups, and resources.

Together, they define which actions are allowed against which objects. The verbs field lists permitted actions, whereas the apiGroups and resources fields identify which objects the actions are permitted on. This combination specifies exactly what operations can be performed on which Kubernetes resources within which API groups.

### ❓ Explain the function of each field in the following rules snippet and what permissions it grants.
The rules snippet defines specific permissions for read access to Deployment resources:

```bash
rules:
- verbs: ["get", "watch", "list"]
  apiGroups: ["apps"] 
  resources: ["deployments"]
```

**Function of each field:**
- **verbs**: ["get", "watch", "list"] - Specifies the allowed actions: get (retrieve specific object), watch (stream events), and list (retrieve multiple objects)
- **apiGroups**: ["apps"] - Specifies the API group where the resources belong (Deployments are in the apps API group)
- **resources**: ["deployments"] - Specifies the type of Kubernetes resource that the actions apply to

**Permissions granted:** This rule allows read access (get, watch, and list operations) against Deployment objects in the apps API group.

### ❓ How can the asterisk (*) wildcard be used in RBAC rules to grant permissions across all API groups, resources, and verbs, and analyze the YAML Role definition that demonstrates this?
When building RBAC rules, you can use the asterisk (*) wildcard to refer to all API groups, all resources, and all verbs. This allows you to grant comprehensive permissions across the entire cluster. For example, the following Role object grants all actions on all resources in every API group:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: shield
  name: read-deployments
rules:
- verbs: ["*"]       <<---- All actions
  resources: ["*"]   <<---- on all resources
  apiGroups: ["*"]   <<---- in every API group
```

This YAML definition uses wildcards in three key places:
- `verbs: ["*"]` - grants permission to perform all actions/operations
- `resources: ["*"]` - applies to all resources types
- `apiGroups: ["*"]` - covers all API groups

It's important to note that this is just for demonstration purposes, and you probably shouldn't create rules like this in production as it grants extremely broad permissions equivalent to root access.

### ❓ What are the four RBAC objects in Kubernetes, how do they differ in scope between namespaced and cluster-wide objects, and describe the powerful pattern of using ClusterRoles with RoleBindings for role reusability across namespaces?
Kubernetes has four RBAC objects:

1. **Roles** - namespaced objects applied to specific Namespaces
2. **RoleBindings** - namespaced objects applied to specific Namespaces
3. **ClusterRoles** - cluster-wide objects that apply to all Namespaces
4. **ClusterRoleBindings** - cluster-wide objects that apply to all Namespaces

Roles and RoleBindings are namespaced objects, meaning you apply them to specific Namespaces. On the other hand, ClusterRoles and ClusterRoleBindings are cluster-wide objects and apply to all Namespaces. They're all defined in the same API sub-group, and their YAML structures are almost identical.

A powerful pattern is to use ClusterRoles to define roles once at the cluster level and then use RoleBindings to bind them to multiple specific Namespaces. This lets you define common roles once and re-use them in as many Namespaces as required. This approach provides the benefit of role reusability - you can define a standard set of permissions as a ClusterRole and then apply those same permissions consistently across multiple namespaces without having to redefine the role each time.

![Figure 14.2 - Combining ClusterRoles and RoleBindings](media/figure14-2.png)

Figure 14.2 shows a single ClusterRole applied to two Namespaces via two separate RoleBindings, demonstrating this efficient pattern of role definition and binding.

### ❓ Compare the ClusterRole YAML definition with the earlier Role definition - what are the key structural differences that make it cluster-scoped?
The key structural difference between a ClusterRole and a regular Role is in the `kind` property and the absence of a `namespace` property in the metadata. Here's the ClusterRole YAML definition:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole                   <<---- Cluster-scoped role
metadata:
  name: read-deployments
rules:
- verbs: ["get", "watch", "list"]
  apiGroups: ["apps"] 
  resources: ["deployments"]
```

If you look closely at the YAML, the only difference with the earlier Role definition is that this one has its `kind` property set to `ClusterRole` and it doesn't have a `namespace` property in the metadata section. The `ClusterRole` kind makes it cluster-scoped, and the absence of a namespace property indicates it applies cluster-wide rather than being limited to a specific namespace.

### ❓ How does the authentication identity (user and group) relate to the authorization mechanism using ClusterRoles and ClusterRoleBindings?
Now that you know you're authenticating as the kubernetes-admin user, which is a member of the kubeadm:cluster-admins group, the cluster uses ClusterRoles and ClusterRoleBindings to give you cluster-wide admin access.

### ❓ What is the purpose of the cluster-admin ClusterRole and how does it relate to the kubeadm:cluster-admins group membership?
Many Kubernetes clusters use a ClusterRole called cluster-admin to grant admin rights on the cluster. This means your kubernetes-admin user (a member of the kubeadm:cluster-admins group) needs binding to the cluster-admin ClusterRole.

### ❓ What does Figure 14.3 illustrate about the relationship between user authentication, group membership, and cluster-wide admin access?
![Figure 14.3](media/figure14-3.png)

### ❓ What command would you use to inspect the permissions granted by the cluster-admin ClusterRole?
Run the following command to see what access the cluster-admin ClusterRole has: `kubectl describe clusterrole cluster-admin`

### ❓ Analyze the PolicyRule section of the cluster-admin ClusterRole output and explain what level of access it provides across resources and namespaces.
The PolicyRule section shows this Role has access to all verbs on all resources in all Namespaces:

```bash
$ kubectl describe clusterrole cluster-admin
Name:         cluster-admin
Labels:       kubernetes.io/bootstrapping=rbac-defaults
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  *.*        []                 []              [*]
             [*]                []              [*]
```

This means accounts bound to this role can do everything to every object everywhere.

### ❓ Why is the cluster-admin ClusterRole considered equivalent to root access, and what security implications does this level of permission carry?
The cluster-admin ClusterRole is considered equivalent to root because it has access to all verbs on all resources in all Namespaces. This means accounts bound to this role can do everything to every object everywhere. This is a powerful and dangerous set of permissions.

### ❓ What does the cluster-admin Role grant according to the description, and why is it considered powerful and dangerous?
The cluster-admin Role grants access to all verbs on all resources in all Namespaces. This means accounts bound to this role can do everything to every object everywhere. This is the equivalent of root and is a powerful and dangerous set of permissions because it provides complete administrative control over the entire Kubernetes cluster.

### ❓ What command would you use to check for ClusterRoleBindings referencing the cluster-admin role and explain what the following full command sequence does and interpret its output showing ClusterRoleBindings?
To check for ClusterRoleBindings referencing the cluster-admin role, you would use the command: `kubectl get clusterrolebindings | grep cluster-admin`

This command sequence lists all ClusterRoleBindings and filters for those containing "cluster-admin" in their output. The command and its output show:

```bash
$ kubectl get clusterrolebindings | grep cluster-admin
NAME                        ROLE
cluster-admin               ClusterRole/cluster-admin
kubeadm:cluster-admins      ClusterRole/cluster-admin
```

The output reveals two ClusterRoleBindings that reference the cluster-admin role: "cluster-admin" and "kubeadm:cluster-admins", both binding to the ClusterRole called "cluster-admin".

### ❓ Which specific ClusterRoleBinding from the previous output are we interested in, what are we hoping to find listed there, and what command would you use to inspect the kubeadm:cluster-admins ClusterRoleBinding?
We're interested in the kubeadm:cluster-admins binding from the previous output. We're hoping it will list our kubernetes-admin user or the kubeadm:cluster-admins group our user is a member of.

To inspect the kubeadm:cluster-admins ClusterRoleBinding, you would use the command: `kubectl describe clusterrolebindings kubeadm:cluster-admins`

### ❓ Analyze the full output of the following kubectl describe command and explain what it reveals about the binding structure and how the kubeadm:cluster-admins ClusterRoleBinding grants admin rights to the kubernetes-admin user?
The output of the `kubectl describe clusterrolebindings kubeadm:cluster-admins` command reveals the binding structure:

```bash
$ kubectl describe clusterrolebindings kubeadm:cluster-admins
Name:         kubeadm:cluster-admins
Labels:       <none>
Annotations:  <none>
Subjects:                        ----┐ 
  Kind   Name                        | Bind subjects (users) that are
  ----   ----                        | members of the
  Group  kubeadm:cluster-admins      | "kubeadm:cluster-admins group"
Role:                                | to the
  Kind:  ClusterRole                 | ClusterRole 
  Name:  cluster-admin           ----┘ called "cluster-admin"
```

This binding structure shows that it binds subjects (users) that are members of the "kubeadm:cluster-admins" group to the ClusterRole called "cluster-admin". Since our kubernetes-admin user is a member of that group, it gets full admin rights on the cluster through this binding mechanism.

### ❓ Summarize the complete authentication and authorization flow that grants the kubernetes-admin user full admin rights in a Docker Desktop Kubernetes cluster and what does Figure 14.4 illustrate about how kubectl users are mapped to cluster admin privileges?
The complete authentication and authorization flow in Docker Desktop Kubernetes cluster works as follows: Docker Desktop configures your kubeconfig file with a client certificate that identifies a user called kubernetes-admin that is a member of the kubeadm:cluster-admins group. The certificate is signed by the cluster's CA, meaning it will pass authentication. The Docker Desktop Kubernetes cluster has a ClusterRoleBinding called kubeadm:cluster-admins that binds authenticated members of the kubeadm:cluster-admins group to a ClusterRole called cluster-admin. This cluster-admin ClusterRole grants admin rights to all objects in all Namespaces.

![Figure 14.4 - Mapping kubectl users to cluster admin](media/figure14-4.png)

### ❓ What is the primary purpose of Kubernetes authorization and how does RBAC implement the principle of least privilege, and what are the four main components of Kubernetes RBAC and what is the function of each?
The primary purpose of Kubernetes authorization is to ensure authenticated users are allowed to execute actions. RBAC is a popular Kubernetes authorization module that implements least privilege access based on a deny-by-default model that denies all actions unless you create rules that allow them.

Kubernetes RBAC uses four main components:
1. Roles and ClusterRoles - to grant permissions
2. RoleBindings and ClusterRoleBindings - to bind those permissions to users

### ❓ What happens to a request after it successfully passes both authentication and authorization?
Once a request passes authentication and authorization, it moves to admission control.

### ❓ When does admission control occur in the API request flow and what is its primary focus?
Admission control runs immediately after successful authentication and authorization and is all about policies.

### ❓ What are the two types of admission controllers supported by Kubernetes and what is the fundamental difference between mutating and validating admission controllers in terms of their capabilities?
Kubernetes supports two types of admission controllers: Mutating and Validating.

The fundamental difference is that mutating controllers check for compliance and can modify requests, whereas validating controllers check for compliance but cannot modify requests.

### ❓ In what order does Kubernetes run admission controllers, and what types of requests are subject to admission control?
Kubernetes always runs mutating controllers first, and both types only apply to requests attempting to modify the cluster. Read requests are not subjected to admission control.

### ❓ What is the key functional difference between a mutating admission controller and a validating admission controller in enforcing cluster policies?
The key functional difference is that mutating admission controllers can modify requests to meet policy requirements, while validating admission controllers can only reject requests that don't meet policy requirements. For example, if you have a production cluster policy requiring all new and updated objects to have a specific label, a mutating controller can check for the presence of the label and add it if it doesn't exist. However, a validating controller can only reject the request if the label doesn't exist.

### ❓ Using the label requirement example, explain how a mutating controller differs from a validating controller in handling non-compliant requests.
Using the label requirement example where you might have a production cluster with a policy that all new and updated objects must have a specific label: A mutating controller can check new and updated objects for the presence of the label and add it if it doesn't exist. However, a validating controller can only reject the request if the label doesn't exist.

### ❓ Explain what the following full command sequence does to inspect the API server's admission controller configuration and what specific admission controller is shown to be enabled in the Docker Desktop cluster according to the command output?
The command sequence inspects the API server's admission controller configuration by describing the kube-apiserver pod in the kube-system namespace and filtering for admission-related settings. The output shows that the API server is configured to use the NodeRestriction admission controller.

```bash
$ kubectl describe pod kube-apiserver-desktop-control-plane  \
  --namespace kube-system | grep admission

--enable-admission-plugins=NodeRestriction
```

### ❓ How does the mutating admission controller's image pull policy enforcement affect container runtime behavior across cluster nodes and what infrastructure requirement does this create?
The mutating admission controller's image pull policy enforcement forces the container runtime on all nodes to pull all images from the configured registry, preventing Pods from using locally cached images. This creates an infrastructure requirement that all nodes must have valid credentials to pull images from the registry.

### ❓ What is the consequence if any single admission controller rejects an API request during the admission control phase?
If any admission controller rejects a request, the request is immediately denied without checking other admission controllers. This means all admission controllers must approve a request before it runs on the cluster.

### ❓ What are the three sequential security layers that all API server requests must pass through according to the chapter summary and what additional security measure protects the connection?
All requests to the API server must pass through three sequential security layers: authentication, authorization, and then admission control checks. Additionally, the connection between the client and the API server is secured with TLS.

### ❓ What authentication method do most clusters support, and what type of solution should production clusters use instead?
Most clusters support client certificates for authentication. However, production clusters should use enterprise-grade Identity and Access Management (IAM) solutions instead.

### ❓ What is the primary function of the authorization layer in the API server security model and what are the key characteristics of RBAC?
The authorization layer checks whether authenticated requests have permission to carry out specific actions. RBAC (Role-Based Access Control) is the most common authorization module and comprises four objects that let you define permissions and grant them to authenticated users. This layer is pluggable, meaning different authorization modules can be used.

### ❓ At what point in the request processing pipeline do admission controllers take action, what is their primary responsibility, and how do validating versus mutating controllers differ in operational behavior?
Admission controllers kick in after authorization and are responsible for enforcing policies. Validating admission controllers reject requests if they don't meet policy requirements, whereas mutating controllers can modify requests to meet policy requirements. This means mutating controllers can actively change requests to comply with policies, while validating controllers can only accept or reject requests based on policy compliance.

### ❓ What security measures does the API server implement and how does it handle authentication and authorization?
The API server implements several security measures:

1. **TLS encryption** - "It uses TLS to encrypt client connections"
2. **Authentication and authorization** - "It leverages authentication and authorization mechanisms to ensure only valid requests are accepted and executed"
3. **Uniform security** - "Requests from internal and external sources all have to pass through the same authentication and authorization"

The API server ensures that all requests, regardless of their source (internal or external), must pass through the same authentication and authorization processes before being accepted and executed.

### ❓ What are three recommended strategies for preventing or mitigating botnet-based DoS attacks against the Kubernetes API server?
The following may be helpful in either preventing or mitigating botnet-based DoS attacks against the API server: Highly available control plane nodes (multiple replicas of the API server running on multiple nodes across multiple availability zones), monitoring and alerting on API server requests based on sane thresholds, and using things like firewalls to limit API server exposure to the internet.

### ❓ What are the three authorization modes that Kubernetes offers to help safeguard access to the API server and how do they work together?
Kubernetes offers several authorization modes that help safeguard access to the API server. These include: Role-based Access Control (RBAC), Webhook, and Node. You should run multiple authorizers at the same time. For example, it's common to use the RBAC and node authorizers.

RBAC mode lets you restrict API operations to sub-sets of users (these users can be regular user accounts or system services). The idea is that all requests to the API server must be authenticated and authorized. Authentication ensures that requests come from a validated user, while authorization ensures the validated user can perform the requested operation.

Webhook mode lets you offload authorization to an external REST-based policy engine, but it requires additional effort to build and maintain the external engine and makes the external engine a potential single point of failure for every request to the API server.

Node authorization is all about authorizing API requests made by kubelets (Nodes), as the types of requests made to the API server by kubelets are obviously different from those generally made by regular users.

### ❓ What are the two common authorizers mentioned that should be run simultaneously to protect the API server?
The two common authorizers that should be run simultaneously are the RBAC and node authorizers. Running multiple authorizers at the same time provides layered security for the API server.

### ❓ Explain the difference between authentication and authorization in the context of API server requests, using the example of Mia creating Pods.
Authentication ensures that requests come from a validated user, making sure that it really is Mia making the request. Authorization determines if the validated user can perform the requested operation - in this case, whether Mia is allowed to create Pods. In the example, Mia is the user, create is the operation, and Pods is the resource. The idea is that all requests to the API server must be both authenticated and authorized.

### ❓ What are the advantages and disadvantages of using webhook mode for authorization compared to built-in authorizers like RBAC?
Webhook mode lets you offload authorization to an external REST-based policy engine, which can provide more flexible and customized authorization logic. However, it requires additional effort to build and maintain the external engine, and it also makes the external engine a potential single point of failure for every request to the API server. For example, if the external webhook system becomes unavailable, you may be unable to make any requests to the API server. With this in mind, you should be rigorous in vetting and implementing any webhook authorization service.

### ❓ What specific purpose does the node authorizer serve in Kubernetes authorization?
Node authorization is all about authorizing API requests made by kubelets (Nodes). The types of requests made to the API server by kubelets are obviously different from those generally made by regular users, and the node authorizer is designed to help with this specific use case.

### ❓ What security measures should be implemented for public registry usage regarding network access and RBAC controls?
When using a public registry, you should implement strict RBAC rules on the registry to control who can push and pull images from which repositories. For example, you might restrict developers so they can only push and pull against dev and test repositories, whereas you may allow your operations teams to push and pull against production repos. You should also limit internet access to the addresses and ports your registries use.

### ❓ How should push access to image repositories be restricted in a secure container environment?
You may only want a subset of nodes (build nodes) to be able to push images. You may even want to lock things down so that only your automated build systems can push to specific repositories. Configure access controls that only allow authorized users and nodes to push to repositories.

### ❓ What security risks can result from failing to review configuration files like Dockerfiles for embedded secrets?
Failing to review configuration files can lead to serious security risks. A well-publicized example was when an IBM data science experiment embedded private TLS keys in its container images. This meant attackers could pull the image and gain root access to the nodes hosting the containers. The whole thing would've been easily avoided if they'd performed a security review against their Dockerfiles.

### ❓ How does Kubernetes integrate with existing IAM providers and what benefits does this integration provide for employee lifecycle management, including the automatic access management process that occurs when new employees join or leave an organization?
Kubernetes has a robust RBAC subsystem that integrates with existing IAM providers such as Active Directory, other LDAP systems, and cloud-based IAM solutions. Most organizations already have a centralized IAM provider that's integrated with company HR systems to simplify employee lifecycle management.

Kubernetes leverages existing IAM providers instead of implementing its own. This integration provides the following automatic access management benefits:

**For new employees:** They get an identity in the corporate IAM database, and assuming you make them members of the appropriate groups, they will automatically get permissions in Kubernetes.

**For departing employees:** When an employee leaves the organization, an HR process will automatically remove their identity from the IAM database, and their Kubernetes access will cease.

This automated process ensures proper access control throughout the employee lifecycle without manual intervention.

### ❓ Since which Kubernetes version has RBAC been a stable feature, and what should organizations do with its capabilities?
RBAC has been a stable Kubernetes feature since v1.8 and you should leverage its full capabilities.

### ❓ What are the three specific types of activities that should justify remote SSH access to Kubernetes cluster nodes?
Remote SSH access to Kubernetes cluster nodes should only be for the following types of activity:

1. Node management activities that you cannot perform via the Kubernetes API
2. Break the Glass activities, such as when the API server is down
3. Deep troubleshooting

### ❓ Why are accounts with root access to the API server and cluster nodes considered prime targets that require multi-factor authentication protection, and describe the two-stage authentication process for MFA as outlined in the security guidelines?
Accounts with root access to the API server and root access to cluster nodes are extremely powerful and are prime targets for attackers and disgruntled employees. As such, you should protect their use via multi-factor authentication (MFA).

The two-stage MFA authentication process is:

**Stage 1:** Tests knowledge of a username and password

**Stage 2:** Tests possession of something like a one-time password

This two-factor approach ensures that authentication requires both something the user knows (username/password) and something the user possesses (one-time password device or token).

### ❓ What additional security measure beyond MFA should be implemented for workstations with kubectl installed?
You should also secure access to workstations and user profiles that have kubectl installed.

### ❓ What are the two primary methods through which Kubernetes administration can occur, and what security risks are associated with each?
Most of your Kubernetes configuration and administration will be done via the API server, where all requests should be logged. However, it's also possible for malicious actors to gain remote SSH access to control plane nodes and directly manipulate Kubernetes objects. This may include access to the cluster store and etcd nodes.

### ❓ What security measures should be implemented for SSH access to control plane nodes according to the text?
We've already said you should limit SSH access to cluster nodes and bolster security with multi-factor authentication (MFA). However, you should also log all SSH activity and ship it to a secure log aggregator. You should also consider mandating that two competent people be present for all SSH access to control plane nodes.

