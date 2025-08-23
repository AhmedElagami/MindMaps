# Core Functions

## Content

### ❓ What three types of operations cause Pod IP churn and make direct connections to individual Pods unreliable, and how do Kubernetes Services solve this problem?
Rollouts replace old Pods with new ones with new IPs, Scaling up adds new Pods with new IPs, Scaling down deletes existing Pods. Events like these generate IP churn and make Pods unreliable. For example, clients cannot make reliable connections to individual Pods as Kubernetes doesn't guarantee they'll exist. This is where Services come into play by providing reliable networking for groups of Pods.

### ❓ What two key functions does a Kubernetes Service provide according to the API resource definition, and how does the diagram in Figure 2.11 demonstrate client connectivity through a Service?
The Service (capital "S" because it's a Kubernetes API resource) provides: a reliable name and IP, load balances requests to the Pods behind it. Figure 2.11 shows internal and external clients connecting to a group of Pods via a Kubernetes Service.

![Figure 2.11](media/figure2-11.png)

### ❓ Describe the front-end and back-end architecture of Kubernetes Services and how each component functions, including how Services maintain reliable connectivity despite Pod scaling events, rollouts, and failures.
You should think of Services as having a front end and a back end: The front end has a stable DNS name, IP address, and network port. The back end uses labels to load balance traffic across a dynamic set of Pods. Services keep a list of healthy Pods as scaling events, rollouts, and failures cause Pods to come and go. This means they'll always direct traffic to active healthy Pods. The Service also guarantees the name, IP, and port on the front end will never change.

### ❓ Describe the front-end and back-end components of a Kubernetes Service and their respective guarantees.
Every Service has a front end and a back end:

**Front end:** Includes a DNS name, IP address, and network port that Kubernetes guarantees will never change for the entire life of the Service.

**Back end:** A label selector that sends traffic to healthy Pods with matching labels. The Service load balances traffic over a dynamic set of Pods that match the label selector.

### ❓ What are the three main functions that Services provide for Pods according to the summary, and compare and contrast the front-end and back-end characteristics of a Kubernetes Service.
Services provide three main functions for Pods:
1. They sit in front of Pods and make them accessible via a reliable network endpoint
2. They provide stable front-end access points (IP address, DNS name, and port)
3. They load balance traffic over a dynamic set of Pods that match a label selector

The front-end and back-end characteristics of a Kubernetes Service contrast as follows:

**Front-end characteristics:**
- Provides an IP address, DNS name, and a port
- Guaranteed to be stable for the entire life of the Service
- Offers reliable endpoints on either internal cluster network (ClusterIP) or external access (NodePort/LoadBalancer)

**Back-end characteristics:**
- Load balances traffic over a dynamic set of Pods
- Uses label selectors to identify target Pods
- Maintains up-to-date lists of healthy Pods through EndpointSlice objects
- Can distribute traffic across all cluster nodes or preserve source IPs based on configuration

### ❓ What are the three guaranteed properties of a Service's front end that Kubernetes ensures will never change, and how does a Service's back-end determine which Pods receive traffic?
Kubernetes guarantees three properties of a Service's front end that will never change:

1. **DNS name**: A stable, resolvable domain name for the Service
2. **IP address**: A fixed cluster IP address that remains constant
3. **Port**: A consistent port number that doesn't change

These guarantees provide reliable networking for applications that depend on stable service endpoints.

The Service's back-end determines which Pods receive traffic using a label selector mechanism. The Service configuration includes a label selector that matches Pods based on their labels. Only healthy Pods that match the specified label selector will receive traffic from the Service. This label-based selection allows for dynamic Pod discovery and traffic routing without requiring manual endpoint configuration.

### ❓ Describe the traffic flow from the Ingress to the Pods as outlined in the YAML file, including the role of the Service and label selectors, and what does Figure 9.6 illustrate about the application deployment and network traffic routing?
The YAML file includes an Ingress and a Service. The Ingress directs traffic arriving on the specified path to a ClusterIP Service called wasm-spin, which then forwards the traffic to all Pods with the appropriate label on the specified port. The replicas defined in the Deployment all have the required label, enabling the Service to properly route traffic to them.

Figure 9.6 illustrates the traffic flow and application deployment architecture, showing how network traffic is routed from the Ingress through the Service to the appropriate Pods.

![Figure 9.6](media/figure9-6.png)

### ❓ What is the expected response when testing the application with the curl command to http://localhost:5005/tkb?
When testing the application with the curl command to http://localhost:5005/tkb, the expected response is:

```bash
$ curl http://localhost:5005/tkb
The Kubernetes Book loves Wasm!
```

### ❓ What does the diagram in Figure 10.1 illustrate about how applications communicate in Kubernetes, what does Figure 10.2 show about the high-level service discovery process, and what does Figure 10.3 illustrate about the cluster DNS architecture and service registration process?
Figure 10.1 illustrates that applications in Kubernetes communicate via Services rather than directly with each other. It shows app-a talking to app-b via its Service, demonstrating that when one app needs to communicate with another, it actually communicates through the Service in front of the target app.

![Figure 10.1 - Apps connect via Services](media/figure10-1.png)

Figure 10.2 shows the high-level service discovery process with four main steps:
1. The developer configures app-a to talk to app-b
2. app-a asks Kubernetes for the IP address of app-b
3. Kubernetes returns the IP address
4. app-a sends requests to app-b's IP address

![Figure 10.2](media/figure10-2.png)

Figure 10.3 illustrates the Kubernetes service registry architecture, showing the cluster DNS as a Kubernetes-native application running on the control plane as two or more Pods managed by a Deployment and fronted by its own Service. It also shows a Service registering its name and IP and two containers using it for service discovery.

![Figure 10.3 - Cluster DNS architecture](media/figure10-3.png)

### ❓ Explain the two fundamental requirements that applications need to send requests to other applications in Kubernetes.
Applications need two fundamental requirements to send requests to other applications in Kubernetes:

1. A way to know the name of the other app (the name of its Service)
2. A way to convert the name into an IP address

Developers are responsible for step 1 - ensuring apps know the names of the other apps and microservices they consume. Kubernetes is responsible for step 2 - converting names to IP addresses.

### ❓ What is the high-level purpose of putting applications behind Kubernetes Services?
At a high level, you develop applications and put them behind Services for reliable names and IPs.

### ❓ What is the fundamental problem that service discovery solves for applications in Kubernetes and what does Figure 10.5 show about application-service relationships?
![Figure 10.5](media/figure10-5.png)

Applications talk to other applications via names, but they need to convert these names into IP addresses, which is where service discovery comes into play. Figure 10.5 shows the relationship between applications, their Services, and assigned ClusterIPs. For example, the enterprise app sits behind a ClusterIP Service called ent with IP 192.168.201.240, and the cerritos app sits behind a Service called cer with IP 192.168.200.217.

