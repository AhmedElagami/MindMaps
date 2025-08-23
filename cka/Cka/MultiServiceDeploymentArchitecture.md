# Multi-Service Deployment Architecture

## Content

### ❓ What does Figure 6.1 illustrate about how Deployments manage multiple microservices with different numbers of Pod replicas?
Figure 6.1 illustrates how Deployments manage multiple microservices with different scaling requirements:

![Multi-Service Deployment Architecture](media/figure6-1.png)

The diagram shows the Deployment controller watching and managing two Deployments:
- The **web Deployment** manages four identical web server Pods
- The **cart Deployment** manages two identical shopping cart Pods

This demonstrates that each microservice gets its own Deployment, and Deployments can manage different numbers of Pod replicas based on the specific requirements of each service.

