# Resolution Patterns

## Content

### ❓ What is the final configuration step needed after setting up the Ingress controller, Services, and load balancer to make the hostnames accessible?
The final configuration step needed is DNS name resolution so that shield.mcu.com, hydra.mcu.com, and mcu.com all send traffic to the load balancer.

### ❓ What are the two main approaches for configuring DNS resolution to point hostnames to an Ingress load balancer in production environments?
In production environments, you'll configure either your internal DNS or internet DNS to point hostnames to the Ingress load balancer. How you do this varies depending on your environment and who provides your internet DNS.

### ❓ What is the purpose of creating entries for shield.mcu.com, hydra.mcu.com, and mcu.com in the /etc/hosts file and what is the consequence of this configuration?
The purpose of creating entries for shield.mcu.com, hydra.mcu.com, and mcu.com in the /etc/hosts file is to map these domain names to the IP address of the Ingress load balancer. This configuration ensures that any traffic sent to shield.mcu.com, hydra.mcu.com, or mcu.com will be directed to the Ingress load balancer that Kubernetes automatically created when the Ingress was deployed.

```bash
$ sudo vi /etc/hosts

# Host Database
<Snip>
212.2.246.150 shield.mcu.com
212.2.246.150 hydra.mcu.com
212.2.246.150 mcu.com
```

Remember to save your changes. With this done, any traffic you send to shield.mcu.com, hydra.mcu.com, or mcu.com will be sent to the Ingress load balancer.

### ❓ How do applications use short names versus fully qualified domain names for service discovery?
Apps can use short names such as ent and cer to connect to Services in the local Namespace, but they must use fully qualified domain names to connect to Services in remote Namespaces. Kubernetes automatically resolves short names to the local namespace, while FQDNs are required for cross-namespace connections.

### ❓ How does Figure 10.8 demonstrate the service discovery setup across namespaces?
Figure 10.8 demonstrates the service discovery architecture across namespaces, showing how the configuration deploys two namespaces (dev and prod) with identical services and deployments, plus a jump Pod in the dev namespace for testing service discovery.

![Figure 10.8](media/figure10-8.png)

### ❓ Describe the complete DNS resolution and traffic flow process that occurs when a container connects to a service using its short name within the same namespace.
When a container connects to a service using its short name within the same namespace, the complete DNS resolution and traffic flow process is:

1. The container automatically appended the search domain to the service name
2. The query is sent to the cluster DNS specified in its resolv.conf file
3. The cluster DNS returned the ClusterIP for the ent Service in the local dev Namespace
4. The app sent the traffic to that IP address
5. En route to the node's default gateway, the traffic caused a trap in the node's kernel
6. The kernel redirected the traffic to a Pod hosting the app

This process allows seamless service discovery using short names within the same namespace.

### ❓ What does the cluster DNS return when resolving a service name, and how does this relate to Namespaces? How does the origin of the response differ between Namespace-specific service calls? What two key principles about Kubernetes service discovery are proven by the tests described?
When resolving a service name, the cluster DNS returns the ClusterIP of the Service in the appropriate Namespace. This time, the response comes from a Pod in the prod Namespace. The tests prove that Kubernetes automatically resolves short names to the local Namespace, and that you need to specify FQDNs to connect across Namespaces.

