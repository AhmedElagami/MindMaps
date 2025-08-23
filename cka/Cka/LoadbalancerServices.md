# LoadBalancer Services

## Content

### ❓ What does the kubectl apply command output indicate about the creation of the Service object and what does the kubectl get svc output reveal about the Service type, cluster IP, external IP, and port mapping configuration?
The `kubectl apply -f lb.yml` command output `service/lb-svc created` indicates that the Service object was successfully created from the YAML manifest. The `kubectl get svc lb-svc` output reveals:

- **Service Type**: `LoadBalancer` - exposes the service externally using a cloud provider's load balancer
- **Cluster IP**: `10.100.247.251` - the internal cluster IP address for the service
- **External IP**: `localhost` - the external access point (shows as localhost for Docker Desktop)
- **Port Mapping**: `8080:31086/TCP` - maps external port 8080 to node port 31086 using TCP protocol

```bash
$ kubectl apply -f lb.yml
service/lb-svc created

$ kubectl get svc lb-svc
NAME     TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)       
lb-svc   LoadBalancer   10.100.247.251   localhost     8080:31086/TCP
```

### ❓ What does Figure 6.6 illustrate about accessing a Kubernetes application through a LoadBalancer Service?
Figure 6.6 illustrates a browser accessing the application through a LoadBalancer Service, showing how external traffic reaches the application Pods via the Service's external endpoint.

![Figure 6.6](media/figure6-6.png)

### ❓ How do LoadBalancer Services simplify external access compared to NodePort Services, and what additional component do they introduce, and what does Figure 7.4 illustrate about the architecture of a LoadBalancer Service and how it builds upon NodePort functionality?
LoadBalancer Services simplify external access compared to NodePort Services by putting a cloud load balancer in front of them. They introduce a highly-available cloud load balancer with a publicly resolvable DNS name and low port number.

Figure 7.4 shows that a LoadBalancer Service is basically a NodePort Service fronted by this cloud load balancer. It builds upon NodePort functionality by adding an external load balancer that handles node discovery and health checking, making external access much simpler for clients.

### ❓ Explain the relationship between a LoadBalancer Service and a NodePort Service as described in the text, and describe the complete traffic flow path from client to Pod when using a LoadBalancer Service.
A LoadBalancer Service is essentially a NodePort Service fronted by a highly-available load balancer with a publicly resolvable DNS name and low port number. The relationship is hierarchical: LoadBalancer Services build on top of NodePort Services, which in turn build on top of ClusterIP Services.

The complete traffic flow path from client to Pod when using a LoadBalancer Service is:
1. The client connects to the load balancer via a reliable, friendly DNS name on a low-numbered port
2. The load balancer forwards the request to a NodePort on a healthy cluster node
3. From there, it's the same as a NodePort Service - send to the internal ClusterIP Service
4. The ClusterIP Service selects a Pod from the EndpointSlice
5. The request is finally sent to the target Pod

This stacking relationship is visually represented in Figure 7.5:

![Figure 7.5 - Service stacking](media/figure7-5.png)

### ❓ What does the diagram in Figure 7.4 show about how LoadBalancer Services handle client requests through cloud load balancers to backend Pods, and describe the complete request path from external client to application Pod when using a LoadBalancer Service, including all intermediate components?
![Figure 7.4 - LoadBalancer Service](media/figure7-4.png)

The complete request path for a LoadBalancer Service is:

1. The client connects to the load balancer via a reliable, friendly DNS name on a low-numbered port
2. The load balancer forwards the request to a NodePort on a healthy cluster node
3. The node forwards the request to the internal ClusterIP Service
4. The Service selects a Pod from the EndpointSlice
5. The Service sends the request to the chosen Pod

This shows how LoadBalancer Services handle client requests through cloud load balancers that provide reliable external access points, which then route traffic through the NodePort mechanism to the backend Pods.

### ❓ What additional constructs does Kubernetes automatically create in the background when deploying a LoadBalancer Service, and what additional infrastructure does it create beyond the basic Service constructs?
When deploying a LoadBalancer Service, Kubernetes automatically creates the required NodePort and ClusterIP constructs in the background. This includes all the port mappings and internal routing infrastructure needed for the service to function.

Beyond the basic Service constructs, a LoadBalancer Service creates a load balancer on the underlying cloud platform, as well as all the constructs and mappings to forward traffic from the load balancer to the Pods. This means it provisions one of your cloud's internet-facing load balancers and provides you with public IPs or public DNS names (or local constructs like localhost for local clusters).

### ❓ Explain what the following YAML configuration does, including the purpose of each annotated field.
The following YAML creates a LoadBalancer Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: lb               <<---- Registered with cluster DNS
spec:
  type: LoadBalancer
  ports:
  - port: 8080           <<---- Load balancer port
    targetPort: 9000     <<---- Application port inside container
  selector:
    project: tkb
```

Purpose of each annotated field:
- `name: lb` - The Service name that gets registered with the cluster DNS
- `type: LoadBalancer` - Specifies this as a LoadBalancer Service type
- `port: 8080` - The port that the load balancer will listen on (external facing port)
- `targetPort: 9000` - The port where the application is running inside the container
- `selector: project: tkb` - The label selector that identifies which Pods this Service should route traffic to

This configuration creates a LoadBalancer Service that listens on port 8080 and maps traffic to port 9000 on Pods with the `project: tkb` label.

### ❓ What command should you run to create a new LoadBalancer Service for Pods in the svc-test Deployment?
Run the following command to create a new LoadBalancer Service for the Pods in the svc-test Deployment.

### ❓ What does the TYPE column indicate about a LoadBalancer Service, and what variations might you see in the EXTERNAL-IP column depending on your cluster environment?
The TYPE column shows this one's a LoadBalancer Service, and the one in the example is assigned an EXTERNAL-IP of 212.2.245.220. If you're on a local cluster such as Docker Desktop, the EXTERNAL-IP will show \<pending\>. Some Docker Desktop clusters incorrectly return a localhost IP address in the EXTERNAL-IP column, it should be \<pending\>.

### ❓ What are the differences in accessing a LoadBalancer Service between cloud environments and local Docker Desktop clusters?
The Service in the example is exposed to the internet via a cloud load balancer on the external IP address. If you're running a local Docker Desktop cluster, you'll access it via your laptop's localhost interface. If your Docker Desktop cluster shows a local IP address in the EXTERNAL-IP column, ignore it and use localhost on port 9000.

Once Kubernetes has created your Service, point your browser to the value in the EXTERNAL-IP column on port 9000 to make sure you can see the app. Remember to use localhost for a Docker Desktop cluster.

