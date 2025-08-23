# ClusterIP Services

## Content

### ❓ What three key characteristics define a ClusterIP Service according to the text?
ClusterIP Services have three key characteristics:

1. The IP is only routable on the internal Pod network
2. The name is automatically registered with the cluster's internal DNS
3. All containers are pre-programmed to use the cluster's DNS to resolve names

### ❓ Describe the complete process that occurs when creating a ClusterIP Service called 'skippy' and how other applications can access it within the cluster, and what are the limitations of ClusterIP Services that prevent external access?
When creating a ClusterIP Service called 'skippy':

1. You create a new ClusterIP Service called skippy
2. Kubernetes creates the Service and assigns it an internal IP
3. Kubernetes creates the DNS records in the cluster's internal DNS
4. Kubernetes configures all containers on the cluster to use the cluster DNS for name resolution

This means every app on the cluster can connect to the new app using the 'skippy' name.

However, ClusterIP Services have limitations that prevent external access:
- They aren't routable outside the cluster
- They require access to the cluster's internal DNS service
- They don't work outside the cluster because they rely on internal cluster networking and DNS infrastructure

### ❓ What information does the CLUSTER-IP column provide and what is its network scope?
The CLUSTER-IP column lists the Service's internal IP that's only routable on the internal cluster network.

### ❓ Analyze the following full kubectl command output to determine the details of the kube-dns service, including its type, cluster IP, exposed ports, and age.
```bash
$ kubectl get svc -n kube-system -l k8s-app=kube-dns
NAME       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                  AGE
kube-dns   ClusterIP   10.96.0.10     <none>        53/UDP,53/TCP,9153/TCP   13d
```

The kubectl command output shows that the kube-dns service is a ClusterIP type service with IP address 10.96.0.10. It exposes three ports: 53/UDP, 53/TCP, and 9153/TCP. The service has been running for 13 days.

### ❓ Explain the head and tail analogy used to describe regular Kubernetes Service objects versus headless Services and what is the primary purpose of headless Services in relation to StatefulSet Pods and DNS records?
It's helpful to visualize Service objects with a head and a tail. The head is the stable ClusterIP address, and the tail is the list of Pods it forwards traffic to. A headless Service is a regular Kubernetes Service object without the head/ClusterIP address.

The primary purpose of headless Services is to create DNS SRV records for StatefulSet Pods. Clients query DNS for individual Pods and then send queries directly to those Pods instead of via the Service's ClusterIP. This is why headless Services don't have a ClusterIP.

### ❓ Explain what the following full YAML configuration does to create a headless Service called 'dullahan' and what is the single key difference that distinguishes a headless Service from a regular Service in Kubernetes?
```yaml
apiVersion: v1
kind: Service        <<---- Normal Kubernetes Service
metadata:
  name: dullahan     <<---- Only use valid DNS characters in name
  labels:
    app: web
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None    <<---- Make this a headless Service
  selector:
    app: web
```

This YAML configuration creates a headless Service called 'dullahan' that:
- Is a normal Kubernetes Service object
- Has the name 'dullahan' (using valid DNS characters)
- Selects Pods with the label `app: web`
- Exposes port 80 named 'web'
- Has `clusterIP: None` which makes it headless

The only difference from a regular Service is that a headless Service has its `clusterIP` set to `None`.

### ❓ What does the following command sequence do to deploy and verify the headless Service and analyze the service output table to confirm the headless Service properties?
```bash
$ kubectl apply -f headless-svc.yml
service/dullahan created
```

This command deploys the headless Service to your cluster using the YAML configuration file.

```bash
$ kubectl get svc
NAME         TYPE         CLUSTER-IP    EXTERNAL-IP    PORT(S)    AGE
dullahan     ClusterIP    None          <none>         80/TCP     31s
```

The output confirms the headless Service properties:
- **Name**: dullahan
- **Type**: ClusterIP
- **Cluster-IP**: None (confirming it's headless)
- **External-IP**: \<none\>
- **Port**: 80/TCP
- **Age**: 31 seconds

