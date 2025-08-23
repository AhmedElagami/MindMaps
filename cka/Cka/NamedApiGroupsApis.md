# Named API Groups (/apis)

## Content

### ❓ What does GVR stand for and how does it help remember the structure of Kubernetes REST paths, and what does Figure 15.4 demonstrate about the REST path structure for resources in named API groups?
On the topic of REST paths, GVR stands for group, version, and resource, and can be a good way to remember the structure of REST paths. Figure 15.4 shows the REST path to the deployments resource in the apps named group.

![Figure 15.4](media/figure15-4.png)

### ❓ Provide three examples of resources in the apps named group and explain their functional relationship, and which networking-related resources are defined in the networking.k8s.io named group?
For example, the apps group defines resources such as Deployments, StatefulSets, and DaemonSets that manage application workloads. Likewise, we define Ingresses, Ingress Classes, and Network Policies in the networking.k8s.io group.

### ❓ What are the key characteristics of the Kubernetes API as a RESTful interface and how is it organized into groups?
The Kubernetes API is a modern resource-based RESTful API that accepts CRUD-style operations via uniform HTTP methods such as POST, GET, PUT, PATCH, and DELETE. It's divided into named groups for convenience and extensibility. Older resources created in the early days of Kubernetes exist in the original group, which you access via the REST path. All newer objects go into named groups. For example, we define newer network resources in the sub-group available at the REST path.

