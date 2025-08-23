# Namespace-Specific Resolution

## Content

### ❓ How does connecting to an application in a custom Namespace differ from connecting to one in the default Namespace?
Connecting to an application in a custom Namespace is exactly the same as connecting to an app in the default Namespace. The connection process, network access, and service discovery work identically regardless of which Namespace the application is deployed to.

### ❓ How does Kubernetes construct DNS names for StatefulSet Pods and what role does the headless Service play in this process?
Kubernetes constructs predictable and reliable fully qualified DNS names for StatefulSet Pods using the pattern: PodName.ServiceName.Namespace.svc.cluster.local. The headless Service plays a crucial role by registering the Pods and their IP addresses against the Service name, enabling DNS-based service discovery. This allows clients to discover all Pods in the StatefulSet through DNS queries.

### ❓ Explain the full process described for testing DNS discovery using a jump Pod and dig utility, what does the command sequence do to deploy the jump Pod, and what happens when executing the command to connect to the jump Pod?
The process for testing DNS discovery involves deploying a jump Pod with the dig utility pre-installed. First, you deploy the jump Pod using the following command:

```bash
$ kubectl apply -f jump-pod.yml
pod/jump-pod created
```

This command creates a jump Pod from the YAML configuration file. Once the Pod is running, you execute onto it using:

```bash
$ kubectl exec -it jump-pod -- bash
root@jump-pod:/#
```

This command connects your terminal to the jump Pod, changing the prompt to indicate you're inside the Pod. From within the jump Pod, you then use the dig utility to query DNS for the Service's SRV records to test peer discovery.

### ❓ Analyze the following dig command output to explain how DNS SRV records map to StatefulSet Pods and their IP addresses, and how does the ANSWER SECTION differ from the ADDITIONAL SECTION in the DNS query results for StatefulSet Pod discovery?
The dig command output shows how DNS SRV records map StatefulSet Pods to their IP addresses. The output contains three main sections:

```bash
# dig SRV dullahan.default.svc.cluster.local
<Snip>
;; QUESTION SECTION:
;dullahan.default.svc.cluster.local. IN SRV
;; ANSWER SECTION:
dullahan.default.svc.cluster.local. 30 IN SRV... tkb-sts-1.dullahan.default.svc.cluster.local.
dullahan.default.svc.cluster.local. 30 IN SRV... tkb-sts-0.dullahan.default.svc.cluster.local.
dullahan.default.svc.cluster.local. 30 IN SRV... tkb-sts-2.dullahan.default.svc.cluster.local.
;; ADDITIONAL SECTION:
tkb-sts-0.dullahan.default.svc.cluster.local. 30 IN A 10.60.0.5
tkb-sts-2.dullahan.default.svc.cluster.local. 30 IN A 10.60.1.7
tkb-sts-1.dullahan.default.svc.cluster.local. 30 IN A 10.60.2.12
<Snip>
```

The ANSWER SECTION maps requests for the Service (dullahan.default.svc.cluster.local) to the three individual StatefulSet Pods (tkb-sts-0, tkb-sts-1, tkb-sts-2). The ADDITIONAL SECTION maps the Pod names to their specific IP addresses (10.60.0.5, 10.60.2.12, 10.60.1.7 respectively). This means clients asking about the Service will get the DNS names of the three Pods in the ANSWER SECTION, and the IP addresses of those Pods in the ADDITIONAL SECTION.

