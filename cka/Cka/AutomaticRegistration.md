# Automatic Registration

## Content

### ❓ Why is the service registration process automatic in Kubernetes?
The registration process is automatic because the cluster DNS is a Kubernetes-native application that watches the API server for new Services. Whenever it sees a new one, it automatically registers its name and IP.

