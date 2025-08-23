# mTLS Security Requirements

## Content

### ❓ How does Docker Desktop automatically configure kubeconfig authentication, and why does the cluster trust these credentials?
Docker Desktop automatically configures your kubeconfig file with a client certificate defining an admin user. The certificate is signed by the cluster's built-in CA, meaning the cluster will trust its credentials.

### ❓ Explain what the following command output reveals about the Docker Desktop kubeconfig user entry and its authentication components.
The command output shows the Docker Desktop kubeconfig entry for the built-in user including client certificate data:

```bash
$ kubectl config view
<Snip>
users:
- name: docker-desktop                  ----┐ 
  user:                                     | This is the Docker Desktop 
    client-certificate-data: DATA+OMITTED   | kubeconfig entry for the built-in user
    client-key-data: DATA+OMITTED           | including client certificate
<Snip>                                  ----┘
```

### ❓ What is the relationship between the friendly name 'docker-desktop' in kubeconfig and the actual username used for authentication?
The user entry is called docker-desktop, but this is just a friendly name. The username that authenticates with is embedded within the client certificate in the same section of the file.

### ❓ What are the system requirements and context prerequisites for successfully running the certificate decoding command?
The command only works on Linux-style systems, and you'll need the utility installed. You'll also need to make sure your kubeconfig's current context is set to your Docker Desktop user and cluster.

### ❓ Explain step-by-step what the following full command sequence does to extract and decode the client certificate information from kubeconfig.
The command sequence extracts and decodes the client certificate information through this pipeline:

```bash
$ kubectl config view --raw -o json \
    | jq ".users[] | select(.name==\"docker-desktop\")" \
    | jq -r '.user["client-certificate-data"]' \
    | base64 -d | openssl x509 -text | grep "Subject:"

    Subject: O=kubeadm:cluster-admins, CN=kubernetes-admin
```

It extracts the raw kubeconfig data, filters for the docker-desktop user, extracts the client-certificate-data field, base64 decodes it, then uses openssl to parse the certificate and extract the Subject information.

### ❓ Based on the certificate decoding output, what user identity and group membership does the Docker Desktop client certificate authenticate?
The output shows that commands will authenticate as the kubernetes-admin user that is a member of the kubeadm:cluster-admins group.

### ❓ Which certificate properties encode the username and group memberships for Kubernetes authentication?
Client certificates encode usernames in the CN (Common Name) property and group memberships in the O (Organization) property.

### ❓ What is the purpose of threat modeling according to the STRIDE model in Kubernetes security, what are the six threat categories defined by STRIDE, and how is spoofing defined in this context?
Threat modeling is the process of identifying vulnerabilities so you can put measures in place to prevent and mitigate them. This chapter introduces the popular STRIDE model and shows how you can apply it to Kubernetes.

STRIDE defines six potential threat categories:
- Spoofing
- Tampering
- Repudiation
- Information disclosure
- Denial of service
- Elevation of privilege

Spoofing is pretending to be somebody else with the aim of gaining extra privileges.

### ❓ What are the key Kubernetes components that require secure communication with the API server to prevent spoofing?
Kubernetes comprises lots of small components that work together. These include the API server, controller manager, scheduler, cluster store, and others. It also includes node components such as the kubelet and container runtime. Each has its own privileges that allow it to interact with and modify the cluster.

### ❓ What are the two primary malicious objectives that tampering attempts to achieve and how does the concept of tamper-proof seals in medication packaging relate to cybersecurity countermeasures?
Tampering is the act of changing something in a malicious way to achieve one of two primary objectives:

1.  **Denial of service**: Tampering with the resource to make it unusable.
2.  **Elevation of privilege**: Tampering with a resource to gain additional privileges.

A common non-Kubernetes example is packaging medication — most over-the-counter drugs are packaged with tamper-proof seals that make it obvious if the product has been tampered with. This relates directly to cybersecurity countermeasures, as tampering can be hard to avoid, so a common strategy is to make it obvious when something has been tampered with, similar to these physical seals.

### ❓ List the five specific Kubernetes components mentioned that are vulnerable to tampering attacks.
Tampering with any of the following five Kubernetes components can cause problems:

*   etcd
*   Configuration files for the API server, controller-manager, scheduler, etcd, and kubelet
*   Container runtime binaries
*   Container images
*   Kubernetes binaries

### ❓ What is the fundamental difference between data 'in transit' and 'at rest' in the context of tampering and how does TLS provide protection against in-transit tampering?
In the context of tampering, the fundamental difference is:

*   **In transit** refers to data while it is being transmitted over the network.
*   **At rest** refers to data stored in memory or on disk.

TLS is a great tool for protecting against in-transit tampering as it provides built-in integrity guarantees that warn you when data has been tampered with.

### ❓ What are the five recommended practices for preventing tampering with data at rest in Kubernetes?
The following five recommendations can help prevent tampering with data when it is at rest in Kubernetes:

1.  Restrict access to the servers that are running Kubernetes components, especially control plane components.
2.  Restrict access to repositories that store Kubernetes configuration files.
3.  Only perform remote bootstrapping over SSH (remember to keep your SSH keys safe).
4.  Always run SHA-2 checksums against downloads.
5.  Restrict access to your image registry and associated repositories.

### ❓ What is information disclosure in the context of information security?
Information disclosure is when sensitive data is leaked. Common examples include hacked data stores and APIs that unintentionally expose sensitive data.

### ❓ What types of non-sensitive configuration data are appropriate for storage in ConfigMaps and why should sensitive data like certificates and passwords not be stored in ConfigMaps?
You'll typically use ConfigMaps to store non-sensitive configuration data such as:

- Environment variables
- Configuration files like web server configs and database configs
- Hostnames
- Service name and Service ports
- Account names

You should not use ConfigMaps to store sensitive data such as certificates and passwords, as Kubernetes makes no effort to protect their contents. For sensitive data, you should use Kubernetes Secrets as part of a comprehensive secrets management solution.

