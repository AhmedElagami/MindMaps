# Shared Resources

## Content

### ❓ What components are included in a Pod's shared execution environment and how does resource sharing differ between single-container and multi-container Pods?
Each Pod is a shared execution environment for one or more containers. The execution environment includes:

- a network stack
- volumes
- shared memory
- and more

Containers in a single-container Pod have the execution environment to themselves, whereas containers in a multi-container Pod share it.

### ❓ What does Figure 2.9 demonstrate about network sharing in multi-container Pods?
![Figure 2.9 - Multi-container Pod sharing Pod IP](media/figure2-9.png)

Figure 2.9 shows a multi-container Pod with both containers sharing the Pods IP address. The main application container is accessible outside the Pod, and the sidecar is also accessible. If they need to communicate with each other, container-to-container within the Pod, they can use the Pod's interface.

### ❓ What resources do multi-container Pods share according to the diagram and how can external applications access containers within them?
According to the provided content, multi-container Pods share several resources:

- **Shared process tree (namespace)**
- **Shared hostname (namespace)**
- **Shared IP address and network resources**
- **Shared volumes**

![Figure 4.2 - Multi-container Pod sharing IP and volume](media/figure4-2.png)

External applications and clients can access the containers via the Pod's IP address. The main app container is available on one port and the sidecar container on another port. Both containers also mount the Pod's volume and can use it to share data.

### ❓ What mechanism allows containers within the same Pod to communicate with each other and describe a practical example of how containers in a Pod can use shared volumes for data exchange?
The two containers can use the Pod's localhost adapter if they need to communicate with each other inside the Pod.

For a practical example of volume sharing: the sidecar container might sync static content from a remote Git repo and store it in the volume where the main app container reads it and serves it as a web page.

### ❓ What does the output of this environment command reveal about container hostnames in multi-container Pods?
The command `$ env | grep HOSTNAME` reveals the container's hostname:

```bash
$ env | grep HOSTNAME
HOSTNAME=hello-pod
```

This output shows that the container's hostname matches the name of the Pod (hello-pod). The important revelation is that if this was a multi-container Pod, all of its containers would have the same hostname, as Kubernetes assigns the Pod name as the hostname for every container within the Pod.

### ❓ What three resources do all containers in a multi-container Pod share?
All containers in a multi-container Pod share the Pod's:
- networking
- volumes
- and memory

