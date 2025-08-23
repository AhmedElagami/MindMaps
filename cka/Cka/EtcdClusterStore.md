# etcd Cluster Store

## Content

### ❓ What types of sensitive configuration data are stored in the Kubernetes cluster store (etcd) that make it a prime target for information disclosure attacks?
The Kubernetes cluster store (usually etcd) contains the entire configuration of a Kubernetes cluster, including network and storage configuration, passwords, the cluster CA, and more. This comprehensive collection of sensitive data makes the cluster store a prime target for information disclosure attacks.

### ❓ Why does storing the data encryption key (DEK) on the same node as the Secret create a security vulnerability even when encryption is enabled, and what security improvement did Kubernetes 1.11 introduce with KMS v1 to address this limitation?
Storing the data encryption key (DEK) on the same node as the Secret creates a security vulnerability because gaining access to a node lets you bypass encryption. This is especially worrying on nodes that host the cluster store (etcd nodes). Even when encryption becomes the default, the DEK being stored on the same node as the Secret means node access compromises the encryption.

Kubernetes 1.11 introduced Key Management Service (KMS v1) as a beta feature allowing you to store key encryption keys (KEK) outside of your Kubernetes cluster. KEKs are used to encrypt and decrypt data encryption keys (DEK) and should be safely guarded. We call this approach "envelope encryption".

### ❓ Explain the relationship between Key Encryption Keys (KEKs) and Data Encryption Keys (DEKs) in the envelope encryption approach used by Kubernetes.
In the envelope encryption approach used by Kubernetes, Key Encryption Keys (KEKs) are used to encrypt and decrypt Data Encryption Keys (DEKs). KEKs should be safely guarded and stored outside of the Kubernetes cluster, while DEKs are used for the actual encryption of data. This approach provides an additional layer of security where compromising the DEK alone is insufficient without also compromising the KEK.

### ❓ What are the three major improvements introduced in KMS v2 compared to KMS v1, and how do they enhance security and performance?
KMS v2, which has been generally available (GA) since Kubernetes v1.29, introduced three major improvements:

1. **Generating a new DEK for each encryption** - A new DEK is generated for every write operation and encrypted at rest with KEK

2. **Enhanced cache capabilities** - The cache size for KMS v2 plugins is only limited by underlying control plane node resources, allowing previous DEKs to be cached in memory for faster encryption and decryption operations

3. **Zero downtime re-encryption operations** - Re-encryption of secrets after key rotations no longer requires API server downtime or Pod restarts

These improvements enhance both security (through per-operation key generation) and performance (through better caching and zero-downtime operations).

### ❓ How does the KMS v2 architecture with external KMS providers and Unix socket communication increase the security barrier for attackers attempting to access Secrets, and why is a snapshot of a control plane node insufficient to read a Secret in plain text when using this architecture?
The KMS v2 architecture significantly increases the security barrier through several mechanisms:

KMS plugins run on control plane nodes and communicate with the API server via Unix sockets. KMS plugins then connect to external KMS providers to handle the encryption and decryption of Data Encryption Keys (DEKs).

This architecture means that attackers now need to compromise both the Kubernetes control plane and the external KMS to access Secrets. A snapshot of a control plane node is not enough to read a Secret in plain text because the encryption keys (KEKs) are stored externally with the KMS provider, not on the control plane nodes themselves.

### ❓ What are the recommended storage locations for Key Encryption Keys (KEKs) and why are Hardware Security Modules (HSM) or cloud-based KMS preferred?
For maximum security, you should seriously consider storing your KEKs in Hardware Security Modules (HSM) or cloud-based Key Management Stores (KMS). These specialized systems provide enhanced physical and logical security protections for cryptographic keys, including tamper resistance, access controls, and auditing capabilities that are superior to general-purpose computing environments.

### ❓ What two major contingency scenarios should be planned for when implementing KMS v2 plugin-based encryption, according to the trust but verify model?
When implementing KMS v2 plugin-based encryption, you should follow the trust but verify model and plan for contingencies such as:

1. A lost KEK
2. Loss of connectivity to external KMS provider

These scenarios represent critical failure modes that could prevent access to encrypted data and require robust disaster recovery planning.

### ❓ Why are Kubernetes Secrets considered the preferred method for storing sensitive data like passwords compared to plain-text files or environment variables?
Kubernetes Secrets are considered the preferred method for storing sensitive data like passwords because they provide a more secure alternative to storing decryption keys in plain-text files or environment variables. For example, a front-end container accessing an encrypted back-end database can have the key to decrypt the database mounted as a Secret, which is far better than storing the decryption key in a plain-text file or environment variable.

