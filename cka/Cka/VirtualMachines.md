# Virtual Machines

## Content

### ❓ What are the specific limitations for Docker Desktop clusters when attempting to delete a node, and how are cluster nodes implemented in Docker Desktop?
For Docker Desktop clusters, there are specific limitations when attempting to delete a node:

This only works for Docker Desktop multi-node (kind) clusters, and you'll need to delete and recreate your cluster to restore it back to three nodes. You cannot follow along if you have a Docker Desktop single node (kubeadm) cluster.

Docker Desktop runs cluster nodes as containers, and a three-node cluster will have three containers called desktop-control-plane, desktop-worker, and desktop-worker2.

### ❓ What is the purpose of the Docker Desktop's built-in multi-node Kubernetes cluster mentioned in Chapter 3, and how might other clusters differ in their approach?
The Docker Desktop's built-in multi-node Kubernetes cluster is used for following along with the examples in this chapter. Other clusters will do things slightly differently, but the principles will be similar.

