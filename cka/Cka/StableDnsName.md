# Stable DNS name

## Content

### ❓ What is the purpose of the enterprise app developers configuring their application with the name of the Service in front of the cerritos app?
The enterprise app developers need to configure their application with the name of the Service in front of the cerritos app so that the enterprise app knows where to send requests. This allows the enterprise app to send requests to the Service name (e.g., "cer") rather than needing to know the specific IP addresses of individual Pods, which provides the abstraction and load balancing benefits of Kubernetes Services.

