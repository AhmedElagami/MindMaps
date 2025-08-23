# Security Admission Controllers

## Content

See [Pod Security](PodSecurity.md) for policy details.

### ❓ At what level does Pod Security Admission (PSA) operate, what type of Kubernetes component implements it, and what are its three enforcement modes?
It works at the Namespace level and is implemented as a validating admission controller. PSA offers three enforcement modes: warn (Allows violating Pods to be created but issues a user-facing warning), audit (Allows violating Pods to be created but logs an audit event), and enforce (Rejects Pods if they violate the policy).

### ❓ What is the recommended practice for configuring Pod Security policies across all Namespaces, and what security risk is posed by unconfigured Namespaces?
It's a good practice to configure every Namespace with at least the baseline policy configured to either warn or audit. This allows you to start gathering data on which Pods are failing the policy and why. Any Namespaces without a Pod Security configuration are a gap in your security configuration, and you should attach a policy as soon as possible, even if it's only warning and auditing.

### ❓ What does the following label format indicate about Pod Security Admission configuration, and what are the limitations of PSA as a validating admission controller?
Applying the following label to a Namespace will apply the baseline policy to it. It will allow violating Pods to run but will generate a user-facing warning:

```bash
pod-security.kubernetes.io/warn: baseline
```

The format of the label is pod-security.kubernetes.io/\<mode\>: \<policy\> with the following options: Prefix is always pod-security.kubernetes.io/, Mode is one of warn, audit, or enforce, Policy is always one of privileged, baseline or restricted. PSAs operate as validating admission controllers, meaning they cannot modify Pods. They also cannot have any impact on running Pods.

### ❓ What is the operational nature of Pod Security Admission (PSA) controllers and what are their limitations regarding Pod modification and impact?
PSAs operate as validating admission controllers, meaning they cannot modify Pods. They also cannot have any impact on running Pods. This is because PSA runs as an admission controller and, therefore, only acts on the creation and modification of Pods.

### ❓ Explain what the following full command sequence does to prepare the Git repository for the PSA examples.
The following commands clone the book's GitHub repository and switch to the 2025 branch:

```bash
$ git clone https://github.com/nigelpoulton/TKB.git
<Snip>

$ cd TKB

$ git fetch origin

$ git checkout -b 2025 origin/2025
```

### ❓ Analyze the full kubectl label command below and explain what policy enforcement it applies to the psa-test Namespace.
The following command applies a label to enforce the baseline Pod Security policy on the psa-test Namespace:

```bash
$ kubectl label --overwrite ns psa-test \
    pod-security.kubernetes.io/enforce=baseline
```

This will prevent the creation of any new Pods violating the baseline PSS policy.

### ❓ Interpret the full output from the kubectl describe command below and identify how it confirms the PSA policy label was correctly applied.
The output from the kubectl describe command shows that the baseline policy label was correctly applied to the psa-test Namespace:

```bash
$ kubectl describe ns psa-test

Name:         psa-test
Labels:       kubernetes.io/metadata.name=psa-test
              pod-security.kubernetes.io/enforce=baseline   <<---- label correctly applied
Annotations:  <none>
Status:       Active
```

The line `pod-security.kubernetes.io/enforce=baseline` confirms that the baseline policy enforcement label was successfully applied.

### ❓ Analyze the full error output below that occurs when attempting to deploy the privileged Pod and explain the specific policy violation cited.
The error output shows that Pod creation was forbidden due to violating the baseline policy:

```bash
$ kubectl apply -f psa-pod.yml

Error from server (Forbidden): error when creating "psa-pod.yml": pods "psa-pod" is 
forbidden: violates PodSecurity "baseline:latest": privileged (container "psa-ctr" 
must not set securityContext.privileged=true)
```

The specific policy violation cited is that the container "psa-ctr" must not set `securityContext.privileged=true`, which is prohibited by the baseline Pod Security policy.

### ❓ What does the successful deployment output indicate about the modified Pod's compliance with PSA policies?
The successful deployment output indicates that the modified Pod now complies with the baseline PSA policy requirements:

```bash
$ kubectl apply -f psa-pod.yml
pod/psa-pod created
```

The output shows the Pod was successfully created, confirming that it passed the requirements for the baseline policy and was successfully deployed.

### ❓ Explain the purpose and output of the dry-run command below for testing restricted policy enforcement on the psa-test Namespace.
The dry-run command tests the impact of applying a restricted policy to the psa-test Namespace without actually applying it:

```bash
$ kubectl label --dry-run=server --overwrite ns psa-test \
    pod-security.kubernetes.io/enforce=restricted

Warning: existing pods in namespace "psa-test" violate the new PodSecurity enforce 
level "restricted:latest"
Warning: psa-pod: allowPrivilegeEscalation != false, unrestricted capabilities, 
runAsNonRoot != true, seccompProfile
<Snip>
```

The purpose is to test potential policy impacts before implementation. The output shows warnings that existing pods violate the restricted policy level and lists specific compliance failures for the psa-pod.

### ❓ Explain what happens when running the following full kubectl command sequence to apply a restricted Pod Security Admission policy to a namespace and why doesn't Pod Security Admission terminate existing Pods when a new enforcement policy is applied to a namespace?
When running the kubectl command to apply a restricted Pod Security Admission policy to a namespace:

```bash
$ kubectl label --overwrite ns psa-test \
    pod-security.kubernetes.io/enforce=restricted

Warning: existing pods in namespace "psa-test" violate the new PodSecurity enforce level 
"restricted:latest"
Warning: psa-pod: allowPrivilegeEscalation != false, unrestricted capabilities, 
runAsNonRoot != true, seccompProfile
namespace/psa-test labeled

$ kubectl get pods --namespace psa-test

NAME      READY   STATUS    RESTARTS   AGE
psa-pod   1/1     Running   0          3m9s
```

The command applies the restricted policy to the namespace, which generates warning messages showing that the existing psa-pod violates four policy requirements: allowPrivilegeEscalation not set to false, running unrestricted capabilities, runAsNonRoot not set to true, and seccompProfile issues. However, the existing pod continues running (status remains "Running").

Pod Security Admission doesn't terminate existing Pods when a new enforcement policy is applied because PSA runs as an admission controller and, therefore, only acts on the creation and modification of Pods. It does not affect pods that are already running.

### ❓ What are the three different Pod Security Admission modes that can be configured against a single namespace and explain what the following full kubectl command does when applying multiple Pod Security Admission labels to a namespace?
The three different Pod Security Admission modes that can be configured against a single namespace are:

1. Enforce the policy
2. Warn against the policy
3. Audit against the policy

The following kubectl command applies all three modes to a namespace:

```bash
$ kubectl label --overwrite ns psa-test \
    pod-security.kubernetes.io/enforce=baseline \
    pod-security.kubernetes.io/warn=restricted \
    pod-security.kubernetes.io/audit=restricted
```

This command applies three labels to the Namespace:
- `pod-security.kubernetes.io/enforce=baseline` enforces the baseline policy level
- `pod-security.kubernetes.io/warn=restricted` warns against the restricted policy level
- `pod-security.kubernetes.io/audit=restricted` audits against the restricted policy level

This configuration enforces the baseline policy while providing warnings and audit logging for the more restrictive policy level, which is a good way to implement the policy and prepare for future enforcement.

### ❓ What are the two main limitations of Pod Security Standards and Pod Security Admission mentioned in the text?
The two main limitations of Pod Security Standards (PSS) and Pod Security Admission (PSA) are:

1. Being implemented as a validating admission controller
2. Being unable to modify, import, or create your own policies

### ❓ List three third-party alternatives to Pod Security Admission for Kubernetes security policy enforcement.
Three third-party alternatives to Pod Security Admission for Kubernetes security policy enforcement are:

1. OPA Gatekeeper
2. Kubewarden
3. Kyverno

Others also exist beyond these three solutions.

