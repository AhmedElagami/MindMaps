# CSI Fundamentals

## Content

### ❓ Explain the purpose and function of the Container Storage Interface (CSI) as described in the content and what benefit does it provide to storage providers?
The CSI is an open-source project that defines an industry-standard interface so container orchestrators can leverage external storage resources in a uniform way. For example, it gives storage providers a documented interface to work with. It also means that CSI plugins should work on any orchestration platform that supports the CSI.

### ❓ What are the two main responsibilities of a CSI plugin for external storage providers?
Each external provider has its own CSI plugin that:

- creates the volumes
- surfaces them inside Kubernetes

