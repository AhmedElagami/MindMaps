# Multi-container Patterns

## Content

### ❓ Why are Pods and containers sometimes used interchangeably, and what are the use cases for multi-container Pods, and how do multi-container Pods help implement the single responsibility principle in application design?
Pods and containers are sometimes used interchangeably because the simplest configurations run a single container per Pod. However, there are powerful use cases for multi-container Pods, including:

- Service meshes
- Helper services that initialize environments
- Apps with tightly coupled helper functions such as log scrapers

Multi-container Pods help us implement the single responsibility principle where every container performs a single task. For example, in a service mesh Pod, the main app container might be serving a message queue or some other core application feature. Instead of adding the encryption and telemetry logic into the main app, we keep the app simple and implement the additional services in the service mesh container in the same Pod.

### ❓ When should you choose a multi-container Pod versus single-container Pods with network coupling?
You should choose a multi-container Pod when your application has tightly coupled components needing to share resources such as memory or storage. In most other cases, you should use single-container Pods and loosely couple them over the network.

### ❓ When should containers be placed in the same Pod versus separate Pods and what are the four Kubernetes options for ensuring Pods are scheduled to the same node when needed?
You should only put containers in the same Pod if they need to share resources such as memory, volumes, and networking. If your only requirement is to schedule two workloads to the same node, you should put them in separate Pods and use one of the following options to ensure they're scheduled to the same node:

- nodeSelectors
- Affinity and anti-affinity
- Topology spread constraints
- Resource requests and resource limits

### ❓ Why are multi-container Pods considered a powerful pattern, how does the single responsibility principle apply to container design, what are the benefits of separation of concerns, when should multiple containers be placed in the same Pod, what does Figure 4.5 demonstrate, and what are the two main patterns Kubernetes provides?
**Multi-container Pods as a Powerful Pattern:**
Multi-container Pods are a powerful pattern and are very popular in the real world because they enable advanced application architectures and resource sharing.

**Single Responsibility Principle:**
According to microservices design patterns, every container should have a single clearly defined responsibility. For example, an application syncing content from a repository and serving it as a web page has two distinct responsibilities: syncing the content and serving the web page. You should design this app with two microservices and give each one its own container.

**Benefits of Separation of Concerns:**
We call this separation of concerns, or the single responsibility principle, and it keeps containers small and simple, encourages reuse, and makes troubleshooting easier.

**When to Use Same Pod:**
Sometimes it's better to put multiple containers in the same Pod rather than separate Pods. For example, sticking with the sync and serve app, putting both containers in the same Pod allows the sync container to pull content from the remote system and store it in a shared volume where the web container can read it and serve it.

**Multi-container Pod Architecture (Figure 4.5):**
![Multi-container Pod Diagram](media/figure4-5.png)

Figure 4.5 demonstrates a multi-container Pod architecture where containers can share resources like volumes for coordinated operation.

**Two Main Patterns:**
Kubernetes has two main patterns for multi-container Pods: init containers and sidecar containers.

### ❓ How does the 'kubectl logs' command work with multi-container Pods, what is the default behavior when no container is specified, and how does container ordering affect this behavior?
The `kubectl logs` command pulls logs from any container in a Pod. With multi-container Pods, if you run the command without specifying a container, it automatically gets the logs from the **first container** in the Pod. You can override this default behavior by using the `--container` flag to specify a different container name.

Container ordering in the Pod definition directly affects the default behavior. The first container listed in the YAML specification becomes the default target for logs. If you're unsure of container names or their order, you can run a `kubectl describe` command or check the Pod's YAML file.

Here's an example YAML showing a multi-container Pod with two containers where the first container (app) would be the default for logs:

```bash
kind: Pod
apiVersion: v1
metadata:
  name: logtest
spec:
  containers:
  - name: app                   <<---- First container (default)
    image: nginx
      ports:
        - containerPort: 8080
  - name: syncer                <<---- Second container
    image: k8s.gcr.io/git-sync:v3.1.6
    volumeMounts:
    - name: html
<Snip>
```

To get logs from a specific container (like the syncer container), use this syntax:

```bash
$ kubectl logs logtest --container syncer
```

### ❓ What are the two primary modes of operation for the 'kubectl exec' command, how do they differ in functionality, and what is the difference between remote command execution and exec session modes?
The `kubectl exec` command operates in two primary modes:

1. **Remote command execution**: Lets you send commands to a container from your local shell. The container executes the command and returns the output to your shell. This is non-interactive and for single commands.

2. **Exec session**: Connects your local shell to the container's shell, functioning like being logged directly into the container. This provides an interactive session.

The key difference is that remote command execution is for one-off commands where you send a command and get the output, while exec session creates an ongoing interactive connection that feels like an SSH session into the container.

Example of remote command execution:

```bash
$ kubectl exec hello-pod -- ps
PID   USER     TIME  COMMAND
  1   root      0:00 node ./app.js
 17   root      0:00 ps aux
```

Example of interactive exec session:

```bash
$ kubectl exec -it hello-pod -- sh
#
```

### ❓ What is the purpose of the `kubectl exec` command, what is its format, when should you use the container flag, and what happens when you execute a command using it?
The `kubectl exec` command is used to execute commands in containers within Pods. The format of the command is `kubectl exec <pod-name> -- <command>`, and you should use the container flag (`-c`) if you want to run the command in a specific container within a multi-container Pod. When you execute a command using `kubectl exec`, the container executes the command and displays the result in your local terminal.

### ❓ Explain what the following full command sequence does and interpret the process output it generates, and analyze the error output from this full command execution and explain why it failed.
The command `$ kubectl exec hello-pod -- ps` executes the `ps` command in the first container of the hello-pod Pod, showing the running processes:

```bash
$ kubectl exec hello-pod -- ps
PID   USER     TIME  COMMAND
  1   root      0:00 node ./app.js
 17   root      0:00 ps aux
```

This output reveals that process ID 1 is running `node ./app.js`, which is the main application process, and process ID 17 is the `ps` command itself that was just executed.

The command `$ kubectl exec hello-pod -- curl localhost:8080` fails because the container doesn't have the `curl` command installed:

```bash
$ kubectl exec hello-pod -- curl localhost:8080
OCI runtime exec failed:...... "curl": executable file not found in $PATH
```

The error occurs because the `curl` binary is not present in the container's filesystem, so it cannot be executed.

### ❓ What does the interactive `kubectl exec` session accomplish, how does it differ from non-interactive execution, what do the `-i` and `-t` flags accomplish, and explain what the following full command does to create an interactive session and what the resulting prompt indicates.
The interactive `kubectl exec` session creates a terminal connection to a container that feels like an SSH session, allowing you to run multiple commands interactively. This differs from non-interactive execution which runs a single command and returns the output.

The `-i` and `-t` flags work together: the `-i` flag makes the session interactive by connecting your shell's STDIN and STDOUT streams to the STDIN and STDOUT of the first container in the Pod, and the `-t` flag allocates a pseudo-TTY.

The command `$ kubectl exec -it hello-pod -- sh` creates an interactive exec session:

```bash
$ kubectl exec -it hello-pod -- sh
#
```

This command starts a new shell process (`sh`) in the container, and the prompt changes to `#` to indicate you're now inside the container's shell environment.

### ❓ Interpret the full command sequence shown below and explain what each part accomplishes.
The command sequence shows how to install and use `curl` from within an interactive exec session:

```bash
# apk add curl
<Snip>

# curl localhost:8080
<html><head><title>K8s rocks!</title><link rel="stylesheet" href="http://netdna....
```

1. `apk add curl` - Installs the `curl` package using Alpine Linux's package manager (apk)
2. `curl localhost:8080` - Tests the web server running in the container by making an HTTP request to localhost on port 8080

The output shows a successful HTML response from the web server, confirming that `curl` was installed correctly and the application is responding.

