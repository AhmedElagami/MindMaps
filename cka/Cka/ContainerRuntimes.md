# Container Runtimes

## Content

### ❓ What does the ability to pick and choose the best runtimes for your needs in Kubernetes enable and what are three different advantages that various container runtimes can provide?
The ability to pick and choose the best runtimes for your needs in Kubernetes enables you to select runtimes that provide specific advantages for different use cases. According to the text, various container runtimes can provide:

1. Better isolation
2. Better performance  
3. Compatibility with Wasm (WebAssembly) containers

This flexibility allows you to optimize your Kubernetes environment based on your specific application requirements and performance needs.

![Multi-Runtime Kubernetes Cluster](media/figure1-2.png)

The diagram shows a four-node Kubernetes cluster running multiple container runtimes, demonstrating the runtime flexibility in modern Kubernetes deployments. Notice how some of the nodes have multiple runtimes. Configurations like this are fully supported and increasingly common. You'll work with a configuration like this in Chapter 9 when you deploy a Wasm (WebAssembly) app to Kubernetes.

### ❓ Why did Kubernetes 1.24 remove support for Docker as a runtime and what has been the default runtime for most new Kubernetes clusters since then?
Kubernetes 1.24 removed support for Docker as a runtime because it was bloated and overkill for what Kubernetes needed. Since then, most new Kubernetes clusters ship with containerd (pronounced "container dee") as the default runtime. Containerd is a stripped-down version of Docker optimized for Kubernetes that fully supports applications containerized by Docker.

### ❓ What are the primary functions of container runtimes on worker nodes, which runtime is typically pre-installed on new clusters, and what specific runtime do RedHat OpenShift clusters use?
Every worker node has one or more runtimes for executing tasks. These tasks include: Pulling container images and Managing lifecycle operations such as starting and stopping containers. Most new Kubernetes clusters pre-install the containerd runtime and use it to execute tasks. RedHat OpenShift clusters use the CRI-O runtime. Older clusters shipped with the Docker runtime, but this is no longer supported.

### ❓ What are the primary functions of container runtimes like containerd and CRI-O on Kubernetes worker nodes?
Container runtimes like containerd and CRI-O run on Kubernetes worker nodes and perform low-level tasks such as starting and stopping containers.

### ❓ What security enhancement was introduced in Kubernetes v1.26 regarding binary artifacts and container images?
Starting with Kubernetes v1.26, all binary artifacts and container images used to build Kubernetes clusters are cryptographically signed.

### ❓ What does the diagram in Figure 17.1 illustrate about container image structure and layering, and what does Figure 17.2 demonstrate about application development using standardized base images?
Figure 17.1 illustrates the fundamental structure of container images through layering. It shows an oversimplified example of an image with three layers: the base layer contains the core OS and filesystem components, the middle layer has the libraries and dependencies, and the top layer contains your application. The combination of these three layers forms the complete image that contains everything needed to run the application.

![Figure 17.1 - Image layering](media/figure17-1.png)

Figure 17.2 demonstrates how applications can be built using standardized, approved base images. It shows three applications built on top of two approved base images: one app builds on top of an approved Alpine Linux base image, while the other two apps are web applications that build on top of an approved Alpine+NGINX base image.

![Figure 17.2 - Using approved base images](media/figure17-2.png)

### ❓ Explain the concept and benefits of using approved base images as described in the text, and list and explain the seven specific benefits of maintaining approved base images as outlined.
The concept of using approved base images involves maintaining a small number of standardized base images that are usually derived from official images and hardened according to corporate policies and requirements. For example, organizations might create a limited number of approved base images based on the official Alpine Linux image that has been tweaked to meet specific requirements including patches, drivers, audit settings, and more.

While creating approved base images requires up-front investment, they provide seven specific benefits:

1. **Standard set of drivers** - Ensures consistency across all applications
2. **Known patches** - Provides certainty about the security patches applied
3. **Standardized audit settings** - Maintains consistent security auditing configurations
4. **Reduced software sprawl (less unofficial base images)** - Minimizes the number of different base images in use
5. **Simplified testing (testing against a small set of known bases)** - Makes testing more efficient with fewer variables
6. **Simplified updates (Fewer base images to patch)** - Reduces the maintenance burden for security updates
7. **Simplified troubleshooting (a well-understood and limited set of base images)** - Makes problem-solving easier with familiar base images

Additionally, approved base images allow developers to focus on applications without worrying about OS-related concerns and may help reduce the number of support contracts and suppliers an organization needs to manage.

### ❓ What processes should organizations establish to manage requests for non-standard base images according to the text?
Organizations should establish good processes to manage requests for non-standard base images that include:

1. **Identify why an existing approved base image cannot be used** - Understand the specific requirements that cannot be met by current approved images
2. **Determine whether an existing approved base image can be updated to meet requirements (including if it's worth the effort)** - Evaluate if modifying an existing base image is feasible and cost-effective
3. **Determine the support implications of bringing an entirely new image into the environment** - Assess the long-term maintenance and support requirements for introducing a new image

In most cases, organizations will want to update an existing base image (such as adding a device driver for GPU computing) rather than introducing an entirely new image, as this approach maintains standardization while accommodating specific needs.

### ❓ Compare and contrast the security implications of hosting private registries versus using public registry private repositories as described in the text.
The text describes two main approaches for securing organizational images with different security implications:

**Hosting Private Registries:**
A secure and practical option is to host your own private registries inside your own firewalls. This approach allows you to:
- Control how registries are deployed
- Control how they're replicated
- Control how they're patched
- Create repositories and policies to fit organizational needs
- Integrate with existing identity management providers such as Active Directory

**Using Public Registry Private Repositories:**
If you can't manage your own private registries, you can host images in private repositories on public registries. However, this approach requires:
- Great care in choosing the right public registry
- Careful configuration of the chosen registry
- Not all public registries are equal in terms of security and features

Regardless of the solution chosen, organizations should only host images that are approved for use within the organization, typically from trusted sources vetted by the information security team. Access controls should be placed on repositories so only approved users can push and pull images.

Additional security measures include:
- Restricting which cluster nodes have internet access (since image registries may be on the internet)
- Configuring access controls that only allow authorized users and nodes to push to repositories
- For public registries: limiting internet access to specific addresses and ports used by registries
- Implementing strict RBAC rules on the registry to control who can push/pull from which repositories
- Potentially restricting push capabilities to only build nodes or automated build systems

### ❓ What are the primary advantages of hosting private image registries within an organization's own firewalls?
Hosting private registries within your own firewalls provides several security advantages: you can control how registries are deployed, how they're replicated, and how they're patched. You can also create repositories and policies to fit your organizational needs, and integrate them with existing identity management providers such as Active Directory.

### ❓ What considerations should be made when choosing to host images in private repositories on public registries instead of self-hosting?
If you can't manage your own private registries, you can host your images in private repositories on public registries. However, not all public registries are equal, and you'll need to take great care in choosing the right one and configuring it correctly. You'll probably need to grant your cluster nodes access to the internet so they can pull images, and it's a good practice to limit internet access to the addresses and ports your registries use.

### ❓ What criteria should be applied to determine which container images are approved for hosting in an organization's registry?
You should only host images that are approved for use within your organization. These will typically be from a trusted source and vetted by your information security team. You should place access controls on repositories so that only approved users can push and pull them.

### ❓ How does vulnerability scanning work at the binary level to detect security issues in container images?
Vulnerability scanning services scan your images at a binary level and check their contents against databases of known security vulnerabilities (CVEs). This helps identify known security issues before images are deployed to production.

### ❓ How should vulnerability scanning be integrated into CI/CD pipelines to automatically enforce security policies?
You should integrate vulnerability scanning into your CI/CD pipelines and implement policies that automatically fail builds and quarantine images if they contain particular categories of vulnerabilities. For example, you might implement a build phase that scans images and automatically fails anything using an image with known critical vulnerabilities.

### ❓ Why is the ability to mark vulnerabilities as false positives important in vulnerability scanning solutions?
The ability to mark vulnerabilities as false positives is important because not all vulnerabilities are relevant to every environment. For example, a Python method that performs TLS verification might be vulnerable to Denial of Service attacks when it contains a lot of wildcards. However, if you never use Python in this way, you might not consider the vulnerability relevant and want to mark it as a false positive. Not all scanning solutions allow you to do this.

### ❓ How does cryptographic signing and verification of container images enhance security in the software delivery pipeline?
Cryptographic signing and verification of container images enhances security by ensuring trust and integrity throughout the software delivery pipeline. In this model, developers cryptographically sign their images, and consumers cryptographically verify them when they pull them and run them. This gives the consumer confidence they're working with the correct image and that it hasn't been tampered with.

### ❓ What does the diagram in Figure 17.3 illustrate about the process of signing and verifying container images?
Figure 17.3 shows the high-level process for signing and verifying images. Image signing and verification is usually implemented by the container runtime.

![Figure 17.3](media/figure17-3.png)

### ❓ Why is it important to implement enterprise-wide signing policies rather than leaving image signing to individual users?
It's important to implement enterprise-wide signing policies so it's not left up to individual users. This ensures consistency, security, and compliance across the entire organization rather than relying on individual practices that may vary in quality and security.

### ❓ What are the key security components that should be included in a comprehensive image promotion workflow?
A comprehensive image promotion workflow should include as many of the following security components as possible:

- Policies forcing the use of signed images
- Network rules restricting which nodes can push and pull images
- RBAC rules protecting image repositories
- Use of approved base images
- Image scanning for known vulnerabilities
- Promotion and quarantining of images based on scan results
- Review and scan infrastructure-as-code configuration files

