# Certificate Authority Criticality

## Content

### ❓ Why does the cluster trust and accept the client certificate presented by the Docker Desktop configuration?
The cluster's CA signs the certificate, so it will pass authentication.

### ❓ Why is the security of the Certificate Authority (CA) critical for mTLS in Kubernetes, and what risks exist with self-signed CAs?
Mutual TLS (mTLS) is only as secure as the CA issuing the certificates. Compromising the CA can render the entire mTLS layer ineffective. A typical Kubernetes installation auto-generates a self-signed certificate authority (CA) that issues certificates to all cluster components. While this is better than nothing, it's not enough for production environments on its own.

