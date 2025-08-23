# Restricted Policy

## Content

### ❓ What does the following YAML configuration accomplish in terms of Pod security and service account token mounting?
This YAML configuration disables the automatic mounting of service account tokens for Pods that don't need to communicate with the API server, reducing the attack surface for spoofing:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: service-account-example-pod
spec:
  serviceAccountName: some-service-account
  automountServiceAccountToken: false       <<---- This line
  <Snip>
```

### ❓ Explain what the following YAML configuration does in terms of setting token expiration and audience restrictions for a projected volume.
The YAML configuration sets an expiry period of one hour (3600 seconds) and restricts the service account token to the 'vault' audience in a projected volume. This is configured in the `serviceAccountToken` section of the projected volume source.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    volumeMounts:
    - mountPath: /var/run/secrets/tokens
      name: vault-token
  serviceAccountName: my-pod
  volumes:
  - name: vault-token
    projected:
      sources:
      - serviceAccountToken:
          path: vault-token
          expirationSeconds: 3600     <<---- This line
          audience: vault             <<---- And this one
```

### ❓ How does setting a container's filesystem to read-only help prevent tampering with live Pods and explain what the following YAML configuration accomplishes in terms of filesystem security for Pods?
A good way to prevent a live Pod from being tampered with is setting its filesystems to read-only. This guarantees filesystem immutability.

The following YAML shows how to configure this in a Pod manifest via the `securityContext` section. It makes the container's root filesystem read-only by setting the `readOnlyRootFilesystem` property to `true`. It also uses the `allowedHostPaths` property to ensure anything mounted beneath the `/test` directory will also be read-only.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readonly-test
spec:
  securityContext:
    readOnlyRootFilesystem: true     <<---- R/O root filesystem
    allowedHostPaths:                <<---- Make anything below 
      - pathPrefix: "/test"          <<---- this mount point
        readOnly: true               <<---- read-only (R/O)
<Snip>
```

### ❓ How does configuring a Pod to restrict the number of processes it can create help mitigate a fork bomb attack?
Assume a Pod is the target of a fork bomb attack where a rogue process attempts to bring the system down by creating enough processes to consume all system resources. If you've configured the Pod with process restrictions to restrict the number of processes the Pod can create, you'll prevent it from exhausting the node's resources and confine the attack's impact to the Pod. This also ensures a single Pod doesn't exhaust the PID range for all the other Pods on the node, including the kubelet.

### ❓ What three application-layer security measures help protect Pods against DoS attacks beyond resource request limits and what does Figure 16.1 illustrate about combining these measures?
Beyond resource request limits, the following application-layer security measures help protect Pods against DoS attacks: Define Kubernetes Network Policies to restrict Pod-to-Pod and Pod-to-external communications, utilize mutual TLS and API token-based authentication for application-level authentication (reject any unauthenticated requests), and for defense in depth, you should also implement application-layer authorization policies that implement the least privilege.

Figure 16.1 shows how these can be combined to make it hard for an attacker to successfully DoS an application:

![Figure 16.1](media/figure16-1.png)

### ❓ What are the four main technologies discussed for reducing elevation of privilege risks against Pods and containers?
The four main technologies discussed for reducing elevation of privilege risks against Pods and containers are:
1. Preventing processes from running as root
2. Dropping capabilities
3. Filtering syscalls
4. Preventing privilege escalation

### ❓ Why is running application processes as root considered dangerous in container environments?
Running application processes as root is almost always a bad idea as it grants the application process full access to the container. This is made even worse by the fact the root user of a container sometimes has unrestricted root access to the host system as well. The root user is the most powerful user on a Linux system and is always User ID 0 (UID 0), which means a compromised root process could potentially affect the entire host system.

### ❓ What does the following Pod manifest configure in terms of user ID execution, and how does it apply to multiple containers?
The following Pod manifest configures all containers that are part of this Pod to run processes as UID 1000 (a non-root user):

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo
spec:
  securityContext:      <<---- Applies to all containers in this Pod
    runAsUser: 1000     <<---- Non-root user
  containers:
  - name: demo
    image: example.io/simple:1.0
```

If the Pod has multiple containers, all container processes will run as UID 1000. The securityContext property at the Pod level applies to all containers in the Pod, ensuring they all run with the same non-root user ID. This is one of many settings that fall under the category of PodSecurityContext.

### ❓ What potential problems can occur when multiple Pods or containers use the same UID, and what solutions are suggested to address these issues?
When multiple Pods or containers use the same UID, they will run with the same security context and potentially have access to the same resources. This might be fine if they are replicas of the same Pod, but there's a high chance this will cause problems if they're not replicas. For example, two different containers with R/W access to the same volume can cause data corruption (both writing to the same dataset without coordinating write operations). Shared security contexts also increase the possibility of a compromised container tampering with a dataset it shouldn't have access to.

Solutions suggested to address these issues include:
- User namespaces
- Maintaining a map of UID usage

### ❓ Explain what the following YAML configuration does and how the security context is applied at different levels.
The following YAML configuration demonstrates multi-level security context application:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo
spec:
  securityContext:      <<---- Applies to all containers in this Pod
    runAsUser: 1000     <<---- Non-root user
  containers:
  - name: demo
    image: example.io/simple:1.0
    securityContext:
      runAsUser: 2000   <<---- Overrides the Pod-level setting
```

This example sets the UID to 1000 at the Pod level but overrides it at the container level so that processes in the demo container run as UID 2000. Unless otherwise specified, all other containers in the Pod will use UID 1000. This shows how security context can be applied at both the Pod level (affecting all containers) and at the individual container level (overriding Pod-level settings).

### ❓ What is the purpose of maintaining a map of UID usage in Kubernetes security?
The purpose of maintaining a map of UID usage is to implement user namespaces, which is a Linux kernel technology that allows a process to run as root within a container but run as a different user outside the container. For example, a process can run as UID 0 (the root user) inside the container but get mapped to UID 1000 on the host. This can be a good solution for processes that need to run as root inside the container.

### ❓ Explain how user namespaces work in Linux to provide container security by mapping internal root users to external non-root users and provide a specific example of how UID mapping works between a container and the host system.
User namespaces is a Linux kernel technology that allows a process to run as root within a container but run as a different user outside the container. For example, a process can run as UID 0 (the root user) inside the container but get mapped to UID 1000 on the host. This can be a good solution for processes that need to run as root inside the container.

