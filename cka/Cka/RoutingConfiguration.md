# Routing Configuration

## Content

### ❓ Analyze the full Ingress YAML configuration and explain how it implements both host-based and path-based routing for the MCU applications, including the specific annotation used, the difference between routing rules, and how requests to shield.mcu.com/ and mcu.com/shield would be processed.
The Ingress YAML configuration implements both host-based and path-based routing for MCU applications through different rule configurations:

**Full Ingress YAML Configuration:**
```bash
1  apiVersion: networking.k8s.io/v1
2  kind: Ingress
3  metadata:
4    name: mcu-all
5    annotations:
6      nginx.ingress.kubernetes.io/rewrite-target: /
7  spec:
8    ingressClassName: nginx
9    rules:
10   - host: shield.mcu.com     ----┐ 
11     http:                        |
12       paths:                     |
13       - path: /                  |
14         pathType: Prefix         | Host rule block for shield app
15         backend:                 |
16           service:               |
17             name: svc-shield     |
18             port:                |
19               number: 8080   ----┘
20   - host: hydra.mcu.com      ----┐
21     http:                        |
22       paths:                     |
23       - path: /                  |
24         pathType: Prefix         | Host rule block for hydra app
25         backend:                 |
26           service:               |
27             name: svc-hydra      |
28             port:                |
29               number: 8080   ----┘
30   - host: mcu.com
31     http:
32       paths:
33       - path: /shield        ----┐
34         pathType: Prefix         |
35         backend:                 |
36           service:               |  Path rule block for shield app
37             name: svc-shield     |
38             port:                |
39               number: 8080   ----┘
40       - path: /hydra         ----┐ 
41         pathType: Prefix         |
42         backend:                 |
43           service:               | Path rule block for shield app
44             name: svc-hydra      |
45             port:                |
46               number: 8080   ----┘
```

**Specific Annotation:** The configuration uses `nginx.ingress.kubernetes.io/rewrite-target: /` which rewrites the URL path for path-based routing to ensure proper backend service handling.

**Host-Based vs Path-Based Routing:**
- **Host-based routing (lines 10-29):** Routes traffic based on the hostname in the HTTP request. Requests to `shield.mcu.com/` go to `svc-shield:8080` and requests to `hydra.mcu.com/` go to `svc-hydra:8080`.
- **Path-based routing (lines 30-46):** Routes traffic based on URL paths under a single hostname. Requests to `mcu.com/shield` go to `svc-shield:8080` and requests to `mcu.com/hydra` go to `svc-hydra:8080`.

**Request Processing:**
- A request to `shield.mcu.com/` would be processed by the host-based routing rule (lines 10-19) and routed to the `svc-shield` service on port 8080.
- A request to `mcu.com/shield` would be processed by the path-based routing rule (lines 33-39) and also routed to the `svc-shield` service on port 8080, with the annotation ensuring proper URL rewriting for the backend service.

### ❓ What are the four routing rules that the Ingress object must implement for both host-based and path-based routing, and what does Figure 8.2 illustrate about the overall architecture?
The Ingress object must implement four routing rules:

1. Host-based: shield.mcu.com >> svc-shield
2. Host-based: hydra.mcu.com >> svc-hydra
3. Path-based: mcu.com/shield >> svc-shield
4. Path-based: mcu.com/hydra >> svc-hydra

Figure 8.2 illustrates the overall architecture using host-based and path-based routing:

![Figure 8.2 Host-based routing](media/figure8-2.png)

The diagram shows how a single Ingress object creates a load balancer that routes traffic to multiple applications based on either hostnames or URL paths, demonstrating the overall architecture of how host-based and path-based routing works in Kubernetes Ingress.

### ❓ Describe the complete traffic flow process from client to shield Pods and what the kubectl apply command output indicates about successful deployment.
The complete traffic flow process from client to shield Pods is as follows:

1. Client sends traffic to shield.mcu.com or mcu.com/shield
2. DNS name resolution ensures the traffic goes to the cloud load balancer
3. Ingress controller reads the HTTP headers and finds the hostname (shield.mcu.com) or path (/shield)
4. Ingress rule triggers and routes the traffic to the svc-shield ClusterIP backend Service
5. The ClusterIP Service ensures the traffic reaches a shield Pod

The `kubectl apply -f app.yml` command output indicates successful deployment of both applications and their services:

```bash
$ kubectl apply -f app.yml
service/svc-shield created
service/svc-hydra created
pod/shield created
pod/hydra created
```

This output confirms that both ClusterIP Services (svc-shield and svc-hydra) and both Pods (shield and hydra) were successfully created and deployed.

### ❓ What type of routing rules are defined in lines 20-29, 30-39, and 40-49 of the Ingress configuration?
Lines 20-29 define a host-based rule for traffic arriving on a specific hostname. Lines 30-39 define a path-based rule for traffic arriving on a specific path. Lines 40-49 define another path-based rule for traffic arriving on a specific path.

### ❓ What command is used to deploy the Ingress object and what output confirms its creation?
Deploy the Ingress object with the following command:

```bash
$ kubectl apply -f ig-all.yml 
ingress.networking.k8s.io/mcu-all created
```

### ❓ Analyze the output of 'kubectl get ing' to interpret what the CLASS, HOSTS, ADDRESS, and PORTS fields represent for the mcu-all Ingress object.
Here is the output of `kubectl get ing` for the mcu-all Ingress object:

```bash
$ kubectl get ing
NAME      CLASS   HOSTS                                  ADDRESS          PORTS
mcu-all   nginx   shield.mcu.com,hydra.mcu.com,mcu.com   212.2.246.150    80
```

The CLASS field shows which Ingress class is handling this set of rules. The HOSTS field is a list of hostnames the Ingress will handle traffic for. The ADDRESS field is the load balancer endpoint (if you're on a cloud, it will be a public IP or public DNS name; if you're on a local cluster, it'll probably be different). The PORTS field shows port 80, which indicates HTTP traffic.

### ❓ What protocols are supported by Ingress according to the port limitations mentioned?
On the topic of ports, Ingress only supports HTTP and HTTPS.

### ❓ Interpret the full output of the 'kubectl describe ing mcu-all' command, explaining the purpose of each section including the rules, annotations, and events.
The `kubectl describe ing mcu-all` command output provides detailed information about the Ingress resource configuration:

```bash
$ kubectl describe ing mcu-all
Name:             mcu-all
Namespace:        default
Address:          212.2.246.150
Ingress Class:    nginx
Default backend:  <default>
Rules:
  Host            Path      Backends
  ----            ----      --------
  shield.mcu.com  /         svc-shield:8080 (10.36.1.5:8080)
  hydra.mcu.com   /         svc-hydra:8080 (10.36.0.7:8080)
  mcu.com         /shield   svc-shield:8080 (10.36.1.5:8080)
                  /hydra    svc-hydra:8080 (10.36.0.7:8080)
Annotations:      nginx.ingress.kubernetes.io/rewrite-target: /
Events:           <none>
  Type    Reason  Age                From                      Message
  ----    ------  ----               ----                      -------
  Normal  Sync    27s (x2 over 28s)  nginx-ingress-controller  Scheduled for sync
```

**Name and Namespace**: Identifies the Ingress resource name and its Kubernetes namespace.

**Address**: The IP or DNS name of the load balancer created by the Ingress. This might be different on local clusters.

**Ingress Class**: Specifies which Ingress controller should handle this resource (nginx in this case).

**Default backend**: Where the controller sends traffic arriving on a hostname or path it doesn't have a route for. Not all Ingress controllers implement a default backend.

**Rules section**: Defines the mappings between hosts, paths, and backends. This shows both host-based routing (shield.mcu.com and hydra.mcu.com) and path-based routing (mcu.com/shield and mcu.com/hydra). Backends are usually ClusterIP Services that send traffic to Pods.

**Annotations**: Used to define controller-specific features and integrations with cloud back ends. The `nginx.ingress.kubernetes.io/rewrite-target: /` annotation tells the controller to rewrite all paths so they look like they arrived on root "/". This is a best-effort approach that doesn't work with all apps.

**Events**: Shows recent activity related to the Ingress resource. In this case, it shows normal sync events from the nginx-ingress-controller.

### ❓ What is the purpose of the Default backend in an Ingress controller, and is this feature universally implemented across all Ingress controllers?
The Default backend is where the controller sends traffic arriving on a hostname or path it doesn't have a route for. Not all Ingress controllers implement a default backend, meaning this feature is not universally implemented across all Ingress controllers.

### ❓ How does the Rules section in an Ingress object define traffic routing, and what types of Kubernetes resources typically serve as backends?
The Rules section defines the mappings between hosts, paths, and backends. It specifies how incoming traffic should be routed based on hostnames and URL paths. Backends are usually ClusterIP Services that send traffic to Pods, acting as the final destination for the routed traffic.

### ❓ What is the purpose of annotations in an Ingress object, and how does the nginx.ingress.kubernetes.io/rewrite-target annotation specifically affect path handling?
Annotations are used to define controller-specific features and integrations with your cloud back end. The `nginx.ingress.kubernetes.io/rewrite-target: /` annotation specifically tells the controller to rewrite all paths so they look like they arrived on root "/". This is a best-effort approach, and as you'll see later, it doesn't work with all apps.

### ❓ Based on the architecture description, how does the Ingress controller determine which internal ClusterIP Service to route traffic to and what does Figure 8.4 illustrate?
The Ingress controller determines which internal ClusterIP Service to route traffic to based on the hostname in the HTTP headers. Traffic for shield.mcu.com goes to the svc-shield Service, and traffic for hydra.mcu.com goes to the svc-hydra Service. The traffic arrives on port 80 and the Ingress sends it to the appropriate internal ClusterIP Service.

Figure 8.4 shows the overall architecture and traffic flow for host-based routing:

![Figure 8.4 - host-based routing](media/figure8-4.png)

Traffic hits the load balancer that Kubernetes automatically created when you deployed the Ingress, and the Ingress routes it to the appropriate backend service based on the hostname.

### ❓ Why are requests to mcu.com routed to the default backend and what limitation exists for path-based routing with rewrite targets?
Requests to mcu.com are routed to the default backend because no Ingress rule was created specifically for mcu.com. Depending on your Ingress controller, the message returned will be different, and your Ingress may not even implement a default backend. The default backend configured by the GKE built-in Ingress returns a helpful message saying: "response 404 (backend NotFound), service rules for [ / ] non-existent."

For path-based routing like mcu.com/shield and mcu.com/hydra, the Ingress uses the rewrite targets feature as specified in the object annotation. However, the image doesn't display because path rewrites like this don't work for all apps, indicating a limitation in path-based routing functionality.

### ❓ What are the key components that make up the successful Ingress configuration and what routing methods are supported?
The successful Ingress configuration consists of two applications fronted by two ClusterIP Services, both published through a single load balancer created and managed by Kubernetes Ingress. This configuration demonstrates that you've successfully configured Ingress for host-based and path-based routing.

According to the chapter summary, Kubernetes Ingress objects support two main routing methods:
1. Host-based HTTP routing
2. Path-based HTTP routing

Once you've installed an Ingress controller, you create and deploy Ingress objects, which are lists of rules governing how incoming traffic is routed to applications on your cluster.

### ❓ Describe the process for implementing Ingress routing once an Ingress controller is installed, including what objects are created and what they contain, and what are the two specific types of HTTP routing that Ingress supports?
Once you've installed an Ingress controller, you create and deploy Ingress objects, which are lists of rules governing how incoming traffic is routed to applications on your cluster. The two specific types of HTTP routing that Ingress supports are host-based and path-based HTTP routing.

