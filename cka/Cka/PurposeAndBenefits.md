# Purpose and Benefits

## Content

### ❓ What is the primary purpose of Kubernetes Ingress?
Ingress is all about accessing multiple web applications through a single LoadBalancer Service. It fixes the limitations of NodePort and LoadBalancer Services by letting you expose multiple Services through a single cloud load balancer, creating a single cloud load balancer on port 80 or 443 and using host-based and path-based routing to map connections to different Services on the cluster.

### ❓ According to the chapter summary, what are the two primary benefits of using Ingress to expose multiple applications and what relationship exists between Ingress and service meshes?
The two primary benefits of using Ingress are: 1) It exposes multiple applications (ClusterIP Services) via a single cloud load balancer, and 2) They're stable objects in the API. Regarding the relationship with service meshes, Ingress has features that overlap with a lot of service meshes, and if you're running a service mesh, you may not need Ingress.

### ❓ What relationship exists between Ingress and service meshes according to the chapter summary, and when might Ingress not be needed?
According to the chapter summary, Ingress has features that overlap with a lot of service meshes. If you're running a service mesh, you may not need Ingress. Ingress is a way to expose multiple applications (ClusterIP Services) via a single cloud load balancer, and they're stable objects in the API, but their functionality can be duplicated or superseded by service mesh capabilities.

