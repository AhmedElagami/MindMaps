# Network Routing

## Content

### ❓ Describe the fundamental characteristics and purpose of the pod network in a Kubernetes cluster, including its technical specifications, implementation details, plugin selection process, security considerations, and what Figure 4.4 illustrates about connectivity.
The Kubernetes pod network has several fundamental characteristics and purposes:

**Fundamental Characteristics:**
- Every Kubernetes cluster runs a pod network and automatically connects all Pods to it
- It's usually a flat Layer-2 overlay network that spans every cluster node
- Allows every Pod to talk directly to every other Pod, even if the remote Pod is on a different cluster node

**Implementation:**
- The pod network is implemented by a third-party plugin that interfaces with Kubernetes via the Container Network Interface (CNI)
- You choose a network plugin at cluster build time, and it configures the Pod network for the entire cluster
- Many plugins exist with different pros and cons, with Cilium being the most popular at the time of writing due to its advanced security and observability features

**Connectivity Illustration (Figure 4.4):**
![Pod Network Diagram](media/figure4-4.png)

Figure 4.4 shows three nodes running five Pods. The pod network spans all three nodes, and all five Pods connect to it, meaning all Pods can communicate despite being on different nodes. Notice how the nodes connect to external networks and do not connect directly to the pod network.

**Security Considerations:**
- Newly created clusters commonly implement very open pod networks with little or no security to make Kubernetes easy to use and avoid network security frustrations
- However, you should use Kubernetes Network Policies and other measures to secure the pod network

### ❓ Why is it important to restrict internet access for cluster nodes when managing container image security?
It's important to restrict which cluster nodes have internet access, keeping in mind that your image registry may be on the internet. This helps control security risks and limits exposure to potential threats from external networks.

### ❓ What fundamental role do firewalls play in a layered information security system according to the Kubernetes security context?
Firewalls are an integral part of any layered information security system. The goal is only to allow authorized communications.

### ❓ Describe how Kubernetes implements the pod network and what the two broad categories of CNI plugins are that implement this functionality.
In Kubernetes, Pods communicate over an internal network called the pod network. However, Kubernetes doesn't implement the pod network directly. Instead, it implements a plugin model called the Container Network Interface (CNI) that allows 3rd-party vendors to implement the pod network. Lots of CNI plugins exist, but they fall into two broad categories: Overlay and BGP.

### ❓ What does the following diagram illustrate about overlay networking in Kubernetes and how it handles network complexity between cluster nodes?
![Figure 17.4](media/figure17-4.png)

### ❓ How do overlay networks use VXLAN technologies to encapsulate traffic, and what security implications does this encapsulation have for firewall operations?
Overlay networks use VXLAN technologies to encapsulate traffic for transmission over a simple flat Layer-2 network operating on top of existing Layer-3 infrastructure. This encapsulation hides the original source and target IP addresses, making it harder for firewalls to know what's going on and inspect the actual traffic content.

### ❓ What does the encapsulation diagram demonstrate about how overlay networks handle packet transmission and why this creates challenges for traditional firewalls?
![Figure 17.5 - Encapsulation on overlay network](media/figure17-5.png)

