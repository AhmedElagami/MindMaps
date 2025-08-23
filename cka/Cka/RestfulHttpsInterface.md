# RESTful HTTPS Interface

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

