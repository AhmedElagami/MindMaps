# Pod Security

## Content

See **Admission → Security Admission Controllers → PodSecurity** ([Security Admission Controllers](SecurityAdmissionControllers.md)).

### ❓ Why should privileged Pod creation by human users trigger immediate investigation, and how are privileged Pods typically created when needed?
Privileged Pod creation by a human user: Privileged Pods can often gain root-level access on the node, and you will typically have policies in place to prevent their creation. On the rare occasions they are needed, they will usually be created by automated processes with service accounts.

### ❓ What is the purpose of the 'crawl' phase in the application migration strategy, and what specific security assessment does it involve?
Crawl: Threat modeling your existing apps will help you understand their current security posture. For example, which of your existing apps do and don't communicate over TLS.

### ❓ What is the key principle of the 'walk' phase in application migration to Kubernetes regarding security posture?
Walk: When moving to Kubernetes, ensure the security posture of these apps remains unchanged. For example, if an app doesn't communicate over TLS, do not change this as part of the migration.

### ❓ Describe the progressive security improvement approach recommended in the 'run' phase of application migration.
Run: Start improving the security of applications after the migration. Start with simple non-critical apps, and carefully work your way up to mission-critical apps. You may also want to methodically deploy deeper levels of security, such as initially configuring apps to communicate over one-way TLS and then eventually over two-way TLS.

### ❓ Why should you avoid changing an application's security posture during migration to Kubernetes?
The key point is not to change the security posture of an app as part of migrating it to Kubernetes. This is because performing a migration and making changes can make it easier to misdiagnose issues --- was it the security change or the migration?

### ❓ What container-related vulnerability occurred in February 2019 and what specific access did it allow?
The container-related vulnerability that occurred in February 2019 was CVE-2019-5736. This vulnerability allowed a container process running as root to gain root access on the worker node and all containers running on the host.

### ❓ Which three security practices discussed in this chapter would have prevented the CVE-2019-5736 vulnerability?
The three security practices that would have prevented the CVE-2019-5736 vulnerability are:

1. Image vulnerability scanning
2. Not running processes as root
3. Enabling SELinux

### ❓ How would image vulnerability scanning have helped prevent the CVE-2019-5736 exploit?
Image vulnerability scanning would have helped prevent the CVE-2019-5736 exploit because the vulnerability has a CVE number, so scanning tools would've found it and alerted on it.

### ❓ Why is not running processes as root an effective measure against container escape vulnerabilities?
Not running processes as root is an effective measure against container escape vulnerabilities because policies that prevent root containers would have prevented exploitation of vulnerabilities like CVE-2019-5736.

### ❓ What role does SELinux play in preventing container escape vulnerabilities like CVE-2019-5736?
SELinux plays a protective role in preventing container escape vulnerabilities like CVE-2019-5736 because standard SELinux policies would have prevented exploitation of the vulnerability.

### ❓ Explain how both scanning tools and security policies work together to prevent exploitation of CVE-2019-5736 even if one method fails.
Scanning tools and security policies work together in a defense-in-depth approach to prevent exploitation of CVE-2019-5736. As the vulnerability has a CVE number, scanning tools would've found it and alerted on it. Even if scanning platforms missed it, policies that prevent root containers and standard SELinux policies would have prevented exploitation of the vulnerability.

### ❓ What was the primary purpose of this chapter according to the summary?
The primary purpose of this chapter was to introduce some of the real-world security considerations affecting many Kubernetes.

### ❓ What were the three main image-related best practices discussed for securing the software delivery pipeline?
The three main image-related best practices discussed for securing the software delivery pipeline were:

1. Securing your image registries
2. Scanning images for vulnerabilities
3. Cryptographically signing and verifying images

### ❓ At which three infrastructure stack layers did the chapter discuss workload isolation options?
The chapter discussed workload isolation options at these three infrastructure stack layers:

1. Cluster-level isolation
2. Node-level isolation
3. Runtime isolation options

### ❓ What additional security topics beyond isolation and image security were covered in the chapter summary?
Beyond isolation and image security, the chapter summary covered these additional security topics:

- Identity and access management, including places where additional security measures might be useful
- Auditing

### ❓ How does the chapter connect the real-world vulnerability example to the security best practices covered throughout the chapter?
The chapter connects the real-world vulnerability example to the security best practices by finishing up with a real-world issue that could have been avoided by implementing some of the best practices already covered. Specifically, the CVE-2019-5736 vulnerability could have been prevented by implementing practices like image vulnerability scanning, not running processes as root, and enabling SELinux.

