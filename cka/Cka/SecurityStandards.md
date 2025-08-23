# Security Standards

## Content

### ❓ How does the security model of Wasm differ fundamentally from traditional containers, and compare and contrast their default security postures?
The security model of Wasm differs fundamentally from traditional containers in their default security postures:

**Wasm Security Model:**
- Wasm starts with everything locked down
- Wasm apps execute in a deny-by-default sandbox where the runtime has to explicitly allow access to resources outside the sandbox
- The runtime distrusts the application, meaning access to everything is denied and must be explicitly allowed
- This sandbox has been battle-hardened through many years of use in one of the most hostile environments in the world... the web!

**Traditional Container Security Model:**
- Containers start with everything wide open
- They use an allow-by-default model with broad access to a shared kernel
- Despite containers defaulting to an open security policy, it's important to acknowledge the incredible work done by the community securing containers and container orchestration platforms
- It's easier than ever to run secure containerized apps, especially on hosted Kubernetes platforms
- However, the allow-by-default model with broad access to a shared kernel will always present security challenges for containers

### ❓ Explain the security model of Wasm apps regarding resource access outside their execution environment and what environment is cited as having battle-hardened this model?
Wasm apps execute in a deny-by-default sandbox where the runtime has to explicitly allow access to resources outside the sandbox. This security model has been battle-hardened through many years of use in one of the most hostile environments in the world... the web!

### ❓ Why do most applications require more capabilities than a typical non-root user but less than the full root capabilities set?
While most applications don't need the complete set of root capabilities, they usually require more capabilities than a typical non-root user. What we need, is a way to grant the exact set of privileges a process requires in order to run.

### ❓ How does the modular capabilities system in Linux differ from the traditional all-or-nothing root vs non-root approach and what challenge exists when working with the over 30 Linux capabilities available for container security?
Instead of an all-or-nothing (root --vs-- non-root) approach, you can grant a process the exact set of capabilities required. There are currently over 30 capabilities, and choosing the right ones can be daunting. The root user holds every capability and is, therefore, extremely powerful. However, its power is a combination of lots of small privileges that we call capabilities. Having a modular set of capabilities allows you to be extremely granular when granting permissions.

### ❓ Describe the iterative process used to determine the minimum set of capabilities required by an application.
A common way to find the absolute minimum set of capabilities an application requires, is to run it in a test environment with all capabilities dropped. This causes the application to fail and log messages about the missing permissions. You map those permissions to capabilities, add them to the application's Pod spec, and run the application again. You rinse and repeat this process until the application runs properly with the minimum set of capabilities.

### ❓ What are the two main risks associated with capability testing and implementation in production environments?
With these considerations in mind, it is vital that you have testing procedures and production release processes that can handle all of this. Firstly, you must perform extensive testing of each application. The last thing you want is a production edge case that you hadn't accounted for in your test environment. Such occurrences can crash your application in production! Secondly, every application revision requires the same extensive testing against the capability set.

### ❓ Explain what the following YAML manifest does to enhance container security through capability management.
The following Pod manifest shows how to add the NET_ADMIN and CHOWN capabilities to a container:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: capability-test
spec:
  containers:
  - name: demo
    image: example.io/simple:1.0
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "CHOWN"]
```

By default, Kubernetes implements your chosen container runtime's default set of capabilities (E.g., containerd). However, you can override this as part of a container's securityContext field.

### ❓ How does seccomp differ from capabilities in its approach to container security and what is the fundamental mechanism through which applications interact with the Linux kernel that seccomp controls?
Seccomp, short for secure computing, is similar in concept to capabilities but works by filtering syscalls rather than capabilities. The way an application asks the Linux kernel to perform an operation is by issuing a syscall. seccomp lets you control which syscalls a particular container can make to the host kernel. As with capabilities, you should implement a least privilege model where the only syscalls a container can make are the ones it needs in order to run.

### ❓ Analyze the four types of seccomp profiles available in Kubernetes and their respective security trade-offs, including the advantages and limitations of using the Runtime Default seccomp profile.
Seccomp went GA in Kubernetes 1.19, and you can use it in different ways based on the following seccomp profiles:

- **Non-blocking**: Allows a Pod to run, but records every syscall to an audit log you can use to create a custom profile
- **Blocking**: Blocks all syscalls. It's extremely secure but prevents a Pod from doing anything useful
- **Runtime Default**: Forces a Pod to use the seccomp profile defined by its container runtime. This is a common place to start if you still need to create a custom profile. Profiles that ship with container runtimes are designed to be a balance of usable and secure. They're also thoroughly tested.
- **Custom**: A profile that only allows the syscalls your application needs in order to run. Everything else is blocked. Custom profiles operate the least privilege model and are the preferred approach from a security perspective.

### ❓ Describe the process of creating a custom seccomp profile using the non-blocking audit approach.
The idea is to extensively test your application Pod in a dev/test environment with a non-blocking profile that records all syscalls to an audit log. After that, you'll have a log file listing every syscall the Pod needs in order to run. You then use this to create a custom profile that only allows those syscalls (least privilege). It's common to extensively test your application in dev/test environment with a non-blocking profile that records all syscalls to an audit log. You then use this log to identify your app's syscalls and build the customized profile.

### ❓ What is the purpose of testing applications with a non-blocking profile in dev/test environments, how is the audit log used to create security profiles, and what are the risks and advantages of this approach?
It's common to extensively test your application in dev/test environment with a non-blocking profile that records all syscalls to an audit log. You then use this log to identify your app's syscalls and build the customized profile. The danger with this approach is that your app has some edge cases you miss during testing. If this happens, your application can fail in production when it hits an edge case and uses a syscall not captured during testing. Custom profiles operate the least privilege model and are the preferred approach from a security perspective.

### ❓ What are the two main Linux technologies that extend beyond seccomp filters and Capabilities for process restriction, how are they configured in Kubernetes, and what makes their configuration challenging?
However, we can take things even further with mandatory access control (MAC) systems such as AppArmor and SELinux. These are Linux-only technologies that you configure at the Kubernetes node level and then apply them to Pods via Pod Security Contexts. Both technologies control how processes interact with other system resources and can be hard to configure. However, tools are available that simplify configuration by reading audit logs and generating profiles that you can test and tweak.

### ❓ How do mandatory access control systems differ from seccomp and Capabilities in terms of enforcement, and what caution should be exercised when implementing AppArmor and SELinux?
This is different from seccomp and Capabilities, which are voluntary. However, once enabled, they are mandatory. AppArmor and SELinux are powerful enforcement points that can stop your apps from working if misconfigured. As such, you should perform extensive testing before implementing them.

### ❓ Explain the process creation mechanism in Linux, the relationship between parent and child processes, and the default Linux behavior regarding privilege inheritance.
The only way to create a new process in Linux is for one process to clone itself and then load new instructions onto the new process. We're over-simplifying, but the original process is called the parent process, and the copy is called the child process. By default, Linux allows a child process to claim more privileges than its parent. This is usually a bad idea.

### ❓ Why is it particularly important to prevent privilege escalation in container environments, and what does the following YAML configuration do to prevent it?
This is especially true for containers, as their security configurations are defined against their initial configuration and not against potentially escalated privileges. Fortunately, it's possible to prevent privilege escalation through the allowPrivilegeEscalation property of individual containers, as shown:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo
spec:
  containers:
  - name: demo
    image: example.io/simple:1.0
    securityContext:
      allowPrivilegeEscalation: false    <<---- This line
```

### ❓ What are the two Kubernetes technologies that work together to enforce Pod security settings, and describe their relationship?
Modern Kubernetes clusters implement two technologies to help enforce Pod security settings: Pod Security Standards (PSS) are policies that specify required Pod security settings. Pod Security Admission (PSA) enforces one or more PSS policies when Pods are created. Both work together for effective centralized enforcement of Pod security --- you choose which PSS policies to apply, and PSA enforces them.

### ❓ What are the three PSS policies maintained by the Kubernetes community, how do they differ in security levels, and why might many Pods fail to meet the Restricted policy requirements?
Every Kubernetes cluster gets the following three PSS policies that are maintained and kept up-to-date by the community: Privileged, Baseline, and Restricted. Privileged is a wide-open allow-all policy. Baseline implements sensible defaults. It's more secure than the privileged policy but less secure than restricted. Restricted is the gold standard that implements the current Pod security best practices. Be warned though, it's highly restricted, and lots of Pods will fail to meet its strict requirements.

### ❓ Explain what the following YAML configuration does and identify the specific element that violates the baseline Pod Security policy.
The following YAML defines a Pod with a privileged container that violates the baseline policy:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: psa-pod
  namespace: psa-test     <<---- Deploy it to the new psa-test Namespace
spec:
  containers:
  - name: psa-ctr
    image: nginx
    securityContext:
      privileged: true    <<---- Violates the baseline policy
```

The specific element that violates the baseline policy is `privileged: true` in the container's securityContext, which is not allowed under the baseline Pod Security policy.

### ❓ What modification was made to the YAML configuration to make the Pod compliant with the baseline policy requirements?
The modification made to make the Pod compliant was changing the container's `privileged` setting from `true` to `false`:

```yaml
apiVersion: v1
kind: Pod
<Snip>
spec:
  containers:
  - name: psa-ctr
    image: nginx
    securityContext:
      privileged: false        <<---- Change from true to false
```

### ❓ What four specific policy requirements does the psa-pod Pod fail to meet according to the output?
The psa-pod Pod fails to meet four specific policy requirements:

1. The property is not set to false (allowPrivilegeEscalation != false)
2. It's running unrestricted capabilities
3. The field is not set to true (runAsNonRoot != true)
4. It fails the test (seccompProfile)

### ❓ Based on the dry-run output, what are the four specific policy requirements that the existing psa-pod fails to meet for the restricted policy level?
Based on the dry-run output, the psa-pod Pod fails to meet four policy requirements for the restricted policy level:

1. The allowPrivilegeEscalation property is not set to false
2. It's running unrestricted capabilities
3. The runAsNonRoot field is not set to true
4. It fails the seccompProfile test

### ❓ What security feature was implemented starting from Kubernetes 1.27 that provides default seccomp profiles for all containers?
Starting from Kubernetes 1.27, all containers inherit a default seccomp profile from the container runtime that implements sensible security defaults.

### ❓ What are the key benefits of the third-party security audits and the Cloud Native Security Whitepaper for Kubernetes environments?
These are great tools for identifying potential threats to your Kubernetes environments, as well as potential ways to mitigate them. The Cloud Native Security Whitepaper is worth reading as a way to level up and gain a more holistic perspective on securing cloud-native environments such as Kubernetes.

### ❓ How does the STRIDE model help in threat modeling Kubernetes environments according to the chapter summary?
This chapter taught you how the STRIDE model can be used to threat-model Kubernetes. You stepped through the six threat categories and looked at some ways to prevent and mitigate them. You saw that one threat can often lead to another and that multiple ways exist to mitigate a single threat.

### ❓ Why is defense in depth considered a key tactic in Kubernetes security according to the chapter summary?
As always, defense in depth is a key tactic.

