## TIER 1A — EXAM FOUNDATION & KUBECTL SPEED

1. PSI UI layout (question panel vs terminal)
2. Copy/paste behavior quirks
3. Keyboard efficiency (tmux optional)
4. Default shell speed tuning (.bashrc)
5. kubectl aliases (k, kgp, kaf, kdel)
6. kubectl autocompletion
7. Default namespace awareness
8. Filesystem locations
9. Kubernetes docs navigation speed
10. Search with `/`
11. `site:kubernetes.io` search
12. Time triage: easy → medium → hard
13. Skipping + flagging strategy
14. Partial credit mindset
15. Safe rollback strategy
16. Saving partial YAML safely
17. YAML validation
18. Server-side vs client-side apply
19. Fields safe to delete
20. When NOT to fix root cause


## TIER 1B — KUBECONFIG, CONTEXTS, ACCESS

### 1. Multi-Cluster Context Navigation Failure

**Task:** You are given access to multiple Kubernetes clusters via kubeconfig. Determine why your commands are targeting the wrong cluster and correct the active context and namespace so workloads appear as expected.
**Covers:** List contexts, show current context, switch context, set namespace per context, context mismatch troubleshooting, detect wrong kubeconfig

---

### 2. Broken Access After Kubeconfig Merge

**Task:** After merging multiple kubeconfig files, cluster access fails for one environment. Identify configuration issues and restore connectivity without regenerating credentials.
**Covers:** Multiple kubeconfigs, merge kubeconfigs, kubectl config view --raw, validate cluster connectivity, detect wrong kubeconfig

---

### 3. Authentication Failure on Previously Working Cluster

**Task:** kubectl access to a cluster suddenly fails. Determine whether the issue is related to expired credentials, invalid certificates, or misconfigured authentication data.
**Covers:** Diagnose credential failures, expired cert vs bad cert symptoms, validate cluster connectivity, kubectl config view --raw

---

### 4. RBAC Access Denied Despite Correct Context

**Task:** You are connected to the correct cluster and namespace but access to resources is denied. Determine whether the issue is namespace-scoped or cluster-scoped RBAC and validate permissions.
**Covers:** kubectl auth can-i, namespace vs cluster RBAC failures, impersonation

---

### 5. Impersonation-Based Access Verification

**Task:** Without modifying RBAC objects, verify whether a specific user or service account would be allowed to perform an action in the cluster.
**Covers:** Impersonation, kubectl auth can-i, context mismatch troubleshooting

---

## TIER 1C — kubectl CORE MECHANICS (SPEED)

### 6. Rapid Pod Creation and Validation

**Task:** Quickly create a Pod with a specific image and runtime command, then verify scheduling and runtime details.
**Covers:** Create Pod imperatively, Pod with command + args, get -o wide, describe signal vs noise

---

### 7. Deployment Lifecycle via CLI

**Task:** Create a Deployment using kubectl, expose it for network access, and adjust replica count to meet demand.
**Covers:** Create Deployment imperatively, expose Deployment as Service, scale workloads

---

### 8. YAML Generation and Idempotent Updates

**Task:** Generate Kubernetes manifests without creating resources, then apply changes repeatedly while ensuring no unintended modifications occur.
**Covers:** Generate YAML without creating, apply idempotently, kubectl diff

---

### 9. Live Resource Modification Under Load

**Task:** Modify running workloads safely to adjust configuration while minimizing disruption and avoiding conflicts.
**Covers:** Edit live resources safely, replace resources, patch resources, server-side apply conflict symptoms

---

### 10. Metadata Management for Selection and Automation

**Task:** Adjust resource metadata to support automation and selection logic without breaking existing selectors.
**Covers:** Annotate resources, label add vs overwrite, show labels, delete by label selector

---

### 11. Controlled and Forced Resource Deletion

**Task:** Remove resources that are stuck terminating and ensure dependent objects behave as required.
**Covers:** Graceful deletion, force delete, finalizers blocking deletion, orphaned resources

---

### 12. Advanced Resource Discovery and Filtering

**Task:** Efficiently locate resources across namespaces and large clusters using filtering, sorting, and output customization.
**Covers:** All-namespaces queries, sort output, custom columns, no-headers output, field selectors

---

### 13. Identifying Non-Running and Failed Workloads

**Task:** Quickly identify Pods that are not running and determine recent failure patterns.
**Covers:** Field selector non-running Pods, watch resources, events sorted by time

---

### 14. JSON and Text Output Processing

**Task:** Extract specific fields from Kubernetes objects and filter results using command-line tooling.
**Covers:** jsonpath extraction, kubectl get -o json | jq, grep / filter kubectl output

---

### 15. Container-Level Debugging and Log Analysis

**Task:** Access containers in multi-container Pods to investigate crashes and runtime behavior.
**Covers:** Exec into container, exec into specific container, logs single container, logs multi-container, logs previous crash, follow logs

---

### 16. Network and Runtime Debugging

**Task:** Debug a running application’s network behavior and temporarily access internal services.
**Covers:** Port-forward, get -o wide

---

### 17. Ephemeral and Copy-Based Pod Debugging

**Task:** Debug a failing workload without modifying the original Pod specification.
**Covers:** Ephemeral containers, debug Pod copy

---

### 18. Node-Level Debugging via Kubernetes

**Task:** Investigate node-level issues using Kubernetes primitives without SSH access.
**Covers:** Debug node with privileged Pod

---

### 19. Observing Cluster Changes in Real Time

**Task:** Monitor resource state changes live to confirm successful rollouts or detect regressions.
**Covers:** Watch resources, describe signal vs noise

---

### 20. Advanced Deletion and Selection Operations

**Task:** Clean up large numbers of resources efficiently using selectors and output control.
**Covers:** Delete by label selector, no-headers output, field selectors

## TIER 1D — PODS & WORKLOAD FAILURES (HUGE ROI)

Below is a **clean, minimal, 100%-coverage Tier 1D list**, refactored to **explicitly cover every item (79–106)** with **no gaps, no redundancy, and CKA-accurate task pressure**.

This is the **final Anki FRONT set**.

---

### 1. Batch Pod Completing and Failing Correctly

A one-off Pod is used for a batch task and must exit cleanly on success and remain stopped on failure. Verify correct phase transitions and restart behavior.
**Covers:** Pod lifecycle phases, RestartPolicy behavior

---

### 2. CrashLooping Service Pod with Misconfigured Probes

A long-running service Pod repeatedly restarts and never stabilizes. Identify probe misconfiguration and correct startup and liveness behavior.
**Covers:** Liveness probe, Startup probe, Probe failure behavior, Probe timing fields

---

### 3. Service Pod Removed from Traffic While Running

A Pod remains Running but stops receiving traffic from a Service. Diagnose and fix readiness signaling without restarting the Pod.
**Covers:** Readiness probe, Probe failure behavior

---

### 4. Application Pod Using Environment Configuration

Deploy a Pod that injects application configuration and sensitive data via environment variables from cluster-managed sources.
**Covers:** Pod with env vars, Pod with ConfigMap env, Pod with Secret env

---

### 5. Multi-Container Pod with Shared Data

A Pod runs a main application container and a helper container that must read files written by the application at runtime.
**Covers:** Multi-container Pod behavior, Shared volumes

---

### 6. Init-Driven Dependency Setup Pod

A Pod must perform setup tasks and validation before the main container starts. Ensure correct execution order and failure handling.
**Covers:** Init containers, Pod lifecycle phases

---

### 7. Metadata-Aware Pod

A Pod must consume its own runtime metadata (name, namespace, and resource limits) and expose it to the application.
**Covers:** Downward API, Pod with env vars

---

### 8. Pod Failing Due to Security Context Misconfiguration

An application Pod fails to start because of permission and ownership issues on mounted volumes. Fix the security context to allow execution.
**Covers:** Pod-level securityContext, Container-level securityContext, runAsUser / runAsGroup, fsGroup

---

### 9. Privilege-Restricted Application Pod

Deploy a Pod under strict security constraints, preventing filesystem mutation and privilege escalation while allowing required capabilities.
**Covers:** readOnlyRootFilesystem, allowPrivilegeEscalation, Linux capabilities

---

### 10. Graceful Pod Shutdown with Cleanup Logic

An application must shut down cleanly when deleted, completing in-flight work and executing cleanup steps before termination.
**Covers:** Pod termination flow, terminationGracePeriodSeconds, PreStop hooks

---

### 11. Pod Stuck in ContainerCreating

A Pod never transitions to Running and remains in ContainerCreating. Diagnose image access and sandbox initialization failures.
**Covers:** ContainerCreating failure causes, Pod sandbox creation failures, ImagePullSecrets precedence

---

### 12. Static Pod vs Deployment Failure Recovery

An identical workload is run as both a static Pod and a Deployment. Observe and compare how failures and restarts are handled.
**Covers:** Static Pod vs Deployment behavior, RestartPolicy behavior

---

## TIER 1E — CONTROLLERS THAT SCORE

**Scenario 1: Zero-Downtime Deployment Image Update**
You are given an existing Deployment running a production web service. Update the container image safely while ensuring zero downtime, control surge/unavailable capacity, and verify rollout progress and completion.
**Covers:** Declarative Deployment, Safe image update, RollingUpdate strategy, maxUnavailable / maxSurge, Rollout status

---

**Scenario 2: Deployment Rollout Control and Recovery**
A Deployment rollout is in progress but needs to be paused for investigation, then resumed. Later, the rollout fails and must be rolled back to a previous stable revision, with history reviewed and a restart triggered.
**Covers:** Pause rollout, Resume rollout, Rollback revision, Restart rollout, Failed rollout recovery, Rollout history

---

**Scenario 3: Deployment Blocked by Application Readiness**
A Deployment update does not complete because new Pods never become ready. Identify the cause related to readiness signaling and ensure the rollout can eventually proceed.
**Covers:** Readiness probe blocking rollout, Rollout status

---

**Scenario 4: ReplicaSet Ownership and Drift Investigation**
A Deployment unexpectedly creates or retains extra ReplicaSets after manual changes to Pods. Determine ownership rules and identify configuration drift between desired and actual state.
**Covers:** ReplicaSet ownership rules, ReplicaSet drift detection, Declarative Deployment

---

**Scenario 5: One-Off Batch Job Execution**
Create a Job to run a batch task that must complete successfully a fixed number of times, control retry behavior, and ensure Pods restart correctly on failure.
**Covers:** Create Job, Job completions, BackoffLimit, Job restartPolicy

---

**Scenario 6: Parallel Job Processing and Troubleshooting**
A Job is configured to run tasks in parallel but remains stuck in Active state. Investigate execution behavior and retrieve logs to determine the cause.
**Covers:** Job parallelism, Retrieve Job logs, Job stuck Active causes

---

**Scenario 7: Scheduled Job with Execution Constraints**
Create a CronJob to run on a specific schedule, prevent overlapping executions, temporarily suspend it, and control how much execution history is retained.
**Covers:** CronJob creation, Cron schedule format, ConcurrencyPolicy, Suspend CronJob, CronJob history limits

---

**Scenario 8: Node-Level Agent Deployment**
Deploy a DaemonSet to run a monitoring agent on all eligible nodes. Update the DaemonSet configuration and diagnose why Pods are not scheduled on certain nodes.
**Covers:** Create DaemonSet, Update DaemonSet, DaemonSet scheduling failures

---

**Scenario 9: Stateful Application Deployment and Scaling**
Deploy a StatefulSet for a database workload, scale it up and down, and ensure Pods maintain stable identities and start and stop in the correct order.
**Covers:** Create StatefulSet, Scale StatefulSet, Stable identity, Ordered startup/termination

---

**Scenario 10: Stateful Networking and Storage Configuration**
A StatefulSet requires stable network identities and persistent storage. Configure the required Service and verify DNS naming and PVC allocation per Pod.
**Covers:** StatefulSet DNS, PVC naming, Headless Service requirement

---

## TIER 1F — SCHEDULING & RESOURCES

**Scenario 1: Hardware-Restricted Placement With Soft Preferences**
A payment API must run only on SSD-capable nodes and should prefer a specific zone when available. Update scheduling rules so the Pods still run if the preferred zone has no capacity.
**Covers:** nodeSelector, required node affinity, preferred node affinity, node label management

---

**Scenario 2: Keep Replicas Apart Across the Cluster**
A 3-replica web deployment must not place two replicas on the same hostname, and it should also avoid sharing a zone with another specified workload when possible. Apply the necessary scheduling rules.
**Covers:** pod anti-affinity, required node affinity, preferred node affinity

---

**Scenario 3: Add and Verify Node Labels for Scheduling**
A new node pool is added, but workloads are not landing there as intended. Apply the correct labels to nodes, verify they exist, and update a workload to target those nodes.
**Covers:** node label management, nodeSelector, required node affinity

---

**Scenario 4: Hard Block a Node Pool Using NoSchedule**
A “gpu” node pool must reject all general workloads immediately, but an existing monitoring DaemonSet must still run there. Configure the node pool and the monitoring Pods accordingly.
**Covers:** apply taints, tolerations, NoSchedule vs PreferNoSchedule

---

**Scenario 5: Soft Repel Using PreferNoSchedule**
A “spot/preemptible” node pool should be avoided by normal workloads, but workloads may land there if the cluster is full. Implement this behavior and confirm that scheduling differs from a hard block.
**Covers:** apply taints, tolerations, NoSchedule vs PreferNoSchedule

---

**Scenario 6: Remove an Incorrect Taint Blocking Scheduling**
A critical deployment is stuck Pending because nodes were tainted by mistake. Identify the offending taint and remove it so the workload can schedule normally.
**Covers:** remove taints, scheduler failure event patterns

---

**Scenario 7: NoExecute Evictions With Timed Tolerations**
A node is automatically marked with a NoExecute taint when it becomes unhealthy. Ensure one system Pod stays on the node indefinitely, while another app Pod is evicted only after a grace period.
**Covers:** NoExecute behavior, tolerations, eviction symptoms

---

**Scenario 8: Protect Availability During Maintenance and Drains**
A node maintenance operation is planned that will drain nodes hosting a frontend deployment. Configure the deployment’s disruption protections so that too many replicas cannot be evicted at once, and proceed with the maintenance safely.
**Covers:** PodDisruptionBudget, eviction symptoms

---

**Scenario 9: Priority Scheduling and Forced Preemption**
A “tier-0” service must always schedule even when the cluster is full. Create and apply an appropriate PriorityClass so lower-priority Pods are preempted if needed, and recognize the symptoms/events of preemption.
**Covers:** PriorityClass, pod preemption symptoms, scheduler failure event patterns

---

**Scenario 10: Resource Requests/Limits and QoS Under Pressure**
A batch job is destabilizing nodes. Set appropriate requests and limits for multiple Pods so one becomes Guaranteed, one Burstable, and one BestEffort, and observe the behavioral differences during contention.
**Covers:** requests & limits, QoS classes

---

**Scenario 11: OOMKills vs CPU Throttling Investigation**
A pod repeatedly restarts and performance is poor. Determine whether the issue is OOMKilled or CPU throttling, and apply resource changes to address the specific failure mode.
**Covers:** OOMKilled behavior, CPU throttling, requests & limits

---

**Scenario 12: Enforce Namespace Defaults With LimitRange**
Developers keep deploying Pods without resources specified. Configure a LimitRange to apply default requests/limits and prevent extreme values, then deploy a test Pod to confirm the defaults are applied.
**Covers:** LimitRange defaults, requests & limits, QoS classes

---

**Scenario 13: Create and Enforce CPU/Memory Quotas**
A namespace must be capped to a fixed total CPU and memory usage. Create a ResourceQuota and demonstrate enforcement by attempting to schedule workloads that would exceed the allowed totals.
**Covers:** create ResourceQuota, CPU/memory quota enforcement, quota exceeded symptoms

---

**Scenario 14: Troubleshoot Quota-Blocked Pod Creation**
New Pods in a namespace fail to create or remain Pending after a quota change. Identify which quota is being exceeded, fix the namespace configuration or workload specs, and restore successful scheduling.
**Covers:** troubleshoot quota-blocked Pods, quota exceeded symptoms, scheduler failure event patterns

---

**Scenario 15: Node Unschedulable vs Cordoned Behavior**
During an incident, an operator “cordoned” a node, but workloads are still behaving unexpectedly. Determine the node’s true scheduling state, ensure no new Pods land there, and distinguish this from eviction/drain behavior.
**Covers:** node unschedulable vs cordoned, eviction symptoms

---

**Scenario 16: Multi-Cause Pending Pods and Scheduler Event Diagnosis**
Several Pods are Pending across different namespaces. Use scheduler events to classify each failure as (a) insufficient resources/requests, (b) affinity/anti-affinity constraints, or (c) taints/tolerations/quota restrictions, and fix the underlying causes.
**Covers:** scheduler failure event patterns, requests & limits, required node affinity, preferred node affinity, pod anti-affinity, apply taints, tolerations, CPU/memory quota enforcement


## TIER 1F-A — AUTOSCALING (HPA)

**Scenario 1: Create and Validate an HPA**
Create an HPA for an existing Deployment using CPU utilization. Ensure CPU requests are set, generate load, and verify scale-out and scale-in behavior.
**Covers:** Configure workload autoscaling, HPA creation, CPU requests prerequisite, Observe scaling behavior (`kubectl get hpa`, events)

---

**Scenario 2: HPA Exists but Never Scales**
An HPA is present but remains at current replicas or shows `<unknown>` metrics. Identify and fix the root cause without recreating the workload.
**Failure causes to identify:** Missing Metrics Server; Missing CPU requests; Wrong HPA target / selector; Metrics access/RBAC issues
**Covers:** HPA troubleshooting, Metrics Server dependency, `kubectl top` + HPA status/events


## TIER 1G — NETWORKING (VERY HIGH SCORE)

**Scenario 168–176: Internal Service Reachability Failure**
A backend application is exposed via a Service, but traffic from other pods fails. Investigate Service type usage, selectors, and endpoints, and restore connectivity without recreating workloads.
**Covers:** ClusterIP, Headless Service, Service selectors, selector mismatch debugging, Service exists but no Endpoints, Manual Endpoints, EndpointSlices

---

**Scenario 169–170: External Access via NodePort and LoadBalancer**
An application must be exposed externally for testing and then production use. Configure appropriate Service types and verify traffic flow from outside the cluster.
**Covers:** NodePort, LoadBalancer

---

**Scenario 177–182: Pod and Service DNS Outage**
Pods cannot resolve Service names, and cluster-wide name resolution is partially failing. Diagnose DNS behavior, inspect CoreDNS configuration and logs, and restore DNS functionality.
**Covers:** Pod DNS resolution, Service DNS format, CoreDNS ConfigMap, CoreDNS logs, DNS debug workflow, Cluster-wide DNS outage workflow

---

**Scenario 183–185: Ingress Not Handling Traffic**
Ingress resources exist but traffic never reaches backend Services. Identify the active Ingress controller, ensure the correct IngressClass is used, and create a functional Ingress resource.
**Covers:** Identify Ingress controller, IngressClass, Create Ingress

---

**Scenario 186–189: Ingress Routing and TLS Issues**
Ingress routes some paths incorrectly and HTTPS access fails. Fix path matching behavior, TLS configuration, and ensure unmatched requests are handled properly.
**Covers:** PathType behavior, TLS Ingress, Default backend, Debug Ingress routing

---

**Scenario 190–193: Pod-to-Pod Traffic Blocked by NetworkPolicy**
After applying NetworkPolicies, application pods cannot communicate or resolve DNS. Adjust policies to allow required ingress and egress traffic while keeping default deny behavior.
**Covers:** Default deny ingress NetworkPolicy, Allow podSelector, Allow namespaceSelector, Allow DNS egress

---

**Scenario 194–195: Application Cannot Reach External Services**
An application suddenly loses outbound connectivity. Determine whether egress restrictions are the cause and debug NetworkPolicy rules to restore access.
**Covers:** Default deny egress, NetworkPolicy debugging workflow


### New Scenario: Gateway API routing

**Task:** Use Gateway API resources to route traffic to a backend service; debug why traffic does not route, covering GatewayClass availability/support, controller status, parentRefs/sectionName, allowedRoutes namespace policy, and listener hostname/port mismatches; then fix.
**Covers:** “Use the Gateway API to manage Ingress traffic” (listed on LF pages). ([Linux Foundation - Education][1]) Debug GatewayClass/controller, parentRef + sectionName matching, allowedRoutes namespace policy, Listener hostname/port issues

---

## TIER 1H — END-TO-END TROUBLESHOOTING (EXAM GOLD)
### 1) “Why is the app down?” Events-first triage

A user reports the `payments` API is unavailable. You have cluster-admin access but no prior context. Use an events-first approach to identify the *primary* failure and the most relevant affected objects, then state what you would fix first to restore service fastest.
**Covers:** Events-first debugging; Multi-issue prioritization order

---

### 2) CrashLoopBackOff with init container complications

A Deployment `reporting-api` is failing to become Ready. Pods show `CrashLoopBackOff`, and at least one init container intermittently fails. Determine the failing component(s) and the most likely root cause(s) using Kubernetes-native signals.
**Covers:** CrashLoopBackOff workflow; Init container failures; Events-first debugging; Multi-issue prioritization order

---

### 3) Image won’t pull in one namespace, works in another

A newly deployed workload in namespace `staging` is stuck with `ImagePullBackOff`, but the same image runs in `dev`. Identify what is preventing the image from being pulled and where the configuration mismatch lives.
**Covers:** ImagePullBackOff workflow; Events-first debugging; Multi-issue prioritization order

---

### 4) Pending Pods during a rollout

A rollout of `web-frontend` stalls: new Pods remain `Pending` while old Pods still serve traffic. Determine why scheduling is blocked and what cluster signals confirm the cause.
**Covers:** Pending Pods workflow; Events-first debugging; Multi-issue prioritization order

---

### 5) “It’s slow”: resource pressure + missing metrics clues

The cluster feels slow and HPA is not reacting. You are asked to quickly verify whether resource pressure exists and whether metrics are being collected. Use the appropriate commands/signals and identify symptoms consistent with a missing or broken Metrics Server.
**Covers:** kubectl top pod; kubectl top node; Metrics Server missing symptoms; Events-first debugging

---

### 6) Service has no endpoints (but Pods exist)

A `ClusterIP` Service `inventory-svc` has no endpoints and clients get connection failures. Pods for `inventory` exist and appear Running. Find why the Service is not selecting endpoints and what object(s) must be corrected.
**Covers:** Service has no endpoints; Events-first debugging; Multi-issue prioritization order

---

### 7) Ingress not routing to backend

External traffic reaches the Ingress controller, but requests to `/api` return 404 or default backend. Identify whether the issue is in Ingress rules, Service wiring, endpoints, or controller behavior, and narrow it to the most likely misconfiguration.
**Covers:** Ingress not routing; Service has no endpoints; Events-first debugging; Multi-issue prioritization order

---

### 8) NetworkPolicy blocks traffic in a “working” namespace

After applying NetworkPolicies in namespace `secure`, Pods are Running but cross-service calls fail. Determine whether NetworkPolicy is blocking the traffic path, and identify which direction (ingress/egress) and which selectors/ports are implicated.
**Covers:** NetworkPolicy blocking traffic; Service has no endpoints (verification angle); Events-first debugging; Multi-issue prioritization order

---

### 9) DNS failure breaks service discovery

Multiple workloads fail when calling `*.svc.cluster.local`. Some Pods can reach IPs directly, but names do not resolve (or resolve inconsistently). Use a standard DNS failure workflow to confirm the failure domain and likely culprit components.
**Covers:** DNS failure workflow; Events-first debugging; Multi-issue prioritization order

---

### 10) Volume mount errors + PVC stuck Pending

A StatefulSet `orders-db` will not start. Pods show volume mount errors, and the PVC remains `Pending`. Determine why storage is not being provisioned/attached and what Kubernetes objects/events confirm the cause.
**Covers:** Volume mount errors; PVC Pending workflow; Pending Pods workflow (storage-driven); Events-first debugging

---

### 11) “Deletion stuck”: finalizers and terminating namespace

A team tries to delete namespace `sandbox`, but it remains stuck in `Terminating`. Several resources also won’t delete cleanly. Identify how finalizers are preventing progress and what needs to be removed/adjusted to complete deletion safely.
**Covers:** Finalizers blocking progress; Namespace stuck Terminating; Events-first debugging; Multi-issue prioritization order


---


Below is a **minimal but complete** set of **CKA-style exam scenario fronts** covering **TIER 2A (Storage)** and **TIER 2B (ConfigMaps & Secrets)**.
Each scenario is **task-based**, **non-redundant**, and **covers multiple related skills where possible**.
No solutions or explanations are included.

---

## TIER 2A — STORAGE

---

### 1. Temporary and Node-Local Storage Usage

**Task:** Deploy a Pod that requires scratch storage during runtime and persistent data tied to a specific node. Investigate data loss after Pod restart.
**Covers:** emptyDir, hostPath, Node-specific storage issues

---

### 2. Application Fails Due to Volume Permission Errors

**Task:** A Pod crashes with `permission denied` errors when writing to a mounted volume. Modify the workload so it runs successfully without changing the container image.
**Covers:** Permission denied on volume mounts, fsGroup fixing volume permissions, initContainer chown pattern

---

### 3. Persistent Volume with Incorrect Access Behavior

**Task:** An application using a PVC cannot be scheduled or fails when scaled. Identify and correct the storage configuration.
**Covers:** Access modes, ReadWriteOnce edge cases, PV nodeAffinity mismatch

---

### 4. Manually Provisioned Persistent Volume

**Task:** Create storage for a legacy workload that cannot use dynamic provisioning. Ensure data handling after PVC deletion matches requirements.
**Covers:** Create PV, Create PVC, Reclaim policies, Access modes

---

### 5. Dynamic Provisioning and Default StorageClass

**Task:** A PVC is created without specifying a StorageClass and remains Pending. Fix the cluster so the workload provisions storage automatically.
**Covers:** Default StorageClass, Dynamic provisioning, Pending PVC troubleshooting

---

### 6. WaitForFirstConsumer Scheduling Issue

**Task:** A Stateful workload remains Pending due to storage binding behavior. Diagnose and resolve the issue without changing application manifests.
**Covers:** VolumeBindingMode, WaitForFirstConsumer behavior, PV nodeAffinity mismatch

---

### 7. Resize an In-Use Persistent Volume Claim

**Task:** An application is running out of disk space. Increase available storage without data loss or Pod recreation.
**Covers:** Resize PVC, Dynamic provisioning

---

### 8. PVC Resize Has No Effect

**Task:** A PVC resize is applied but capacity does not increase or the filesystem remains unchanged. Identify why and fix it without data loss.
**Covers:** StorageClass `allowVolumeExpansion`, Filesystem resize requirements, Verification of expanded capacity

---

### 9. PVC Stuck in Terminating State

**Task:** A PVC cannot be deleted and blocks namespace cleanup. Investigate and safely remove it.
**Covers:** PVC stuck Terminating, Reclaim policies

---

### 10. CSI Volume Attachment Failure

**Task:** A Pod cannot start due to volume attachment errors reported by the CSI driver. Restore application availability.
**Covers:** CSI attach failures, Node-specific storage issues

---

### 11. CSI Volume Mount Failure

**Task:** A Pod is scheduled but fails during startup with mount errors. Identify whether the issue is related to node configuration or volume setup.
**Covers:** CSI mount failures, Node-specific storage issues

---

## TIER 2B — CONFIGMAPS & SECRETS

---

### 11. Externalize Application Configuration

**Task:** Move hardcoded configuration values into Kubernetes-native resources and inject them into a running application.
**Covers:** Create ConfigMap from literals, envFrom, Single-key env

---

### 12. Configuration from Files with Volume Mount

**Task:** An application expects configuration files at runtime. Provide the configuration using Kubernetes primitives.
**Covers:** Create ConfigMap from files, ConfigMap volume mount

---

### 13. ConfigMap Update Not Reflected in Pod

**Task:** Configuration changes are not picked up by the application. Ensure the new values are applied correctly.
**Covers:** ConfigMap reload patterns, Rollout restart after ConfigMap update

---

### 14. Inject Sensitive Data as Environment Variables

**Task:** Provide credentials securely to an application using environment variables.
**Covers:** Create Secret from literals, Base64 rules, envFrom Secret

---

### 15. Provide Secrets as Files

**Task:** An application requires certificate files to be present on disk. Mount the required data securely.
**Covers:** Create Secret from files, Mount Secret

---

### 16. Image Pull Failure from Private Registry

**Task:** Pods fail with `ImagePullBackOff` when pulling from a private registry. Fix the authentication issue.
**Covers:** docker-registry Secret, ImagePullSecrets, Fix ImagePullBackOff

---

### 17. Application Crashes Due to Missing Secret Keys

**Task:** A Pod starts but crashes immediately due to configuration errors. Identify and correct the secret-related issue.
**Covers:** Secret key mismatch symptoms, Single-key env, Mount Secret

---

Below is a **100% complete, blueprint-tight revision**.
I made **minimal changes**: one scenario refined, one scenario added.
No redundancy, no solutions, exam-style only.

---
# TIER 2C — RBAC & SECURITY


### **Scenario 1: Investigate a Pod Failing Due to ServiceAccount Permissions**

A Pod running in a namespace cannot access the Kubernetes API and is crashing. Determine whether the failure is caused by default ServiceAccount behavior, missing RBAC permissions, or disabled token automount, and restore access securely.
**Covers:** Default ServiceAccount behavior, Pod fails due to SA permissions, Disable token automount, RBAC troubleshooting workflow

---

### **Scenario 2: Create and Use a Custom ServiceAccount with Least Privilege**

An application requires read-only access to specific resources within its namespace. Create a custom ServiceAccount, define namespace-scoped RBAC rules, bind them correctly, and configure the Pod to use the new ServiceAccount.
**Covers:** Custom ServiceAccount, Role, RoleBinding, Namespace vs cluster scope

---

### **Scenario 3: Grant Cluster-Wide Permissions to a Namespace-Scoped Workload**

A controller running in a single namespace needs to list nodes and watch cluster-wide resources. Configure the required RBAC objects without overprivileging and verify access.
**Covers:** ClusterRole, ClusterRoleBinding, Namespace vs cluster scope

---

### **Scenario 4: Validate and Debug User Permissions**

A kubectl command fails with an authorization error. Verify whether the current user is allowed to perform the action, then impersonate another user to compare permissions and identify missing RBAC bindings.
**Covers:** kubectl auth can-i, kubectl auth can-i with impersonation, RBAC troubleshooting workflow

---

### **Scenario 5: Provision a New User Certificate and Kubeconfig**

A new engineer needs authenticated access to the cluster. Generate credentials, create and submit a CSR, approve it, build a kubeconfig for the user, and confirm access works as expected.
**Covers:** Create kubeconfig for user, CSR creation, Approve CSR

---

### **Scenario 6: Handle Invalid, Rejected, or Expired Certificates**

Multiple users lose access to the cluster due to certificate problems. Identify expired certificates or invalid CSRs, deny or delete requests as appropriate, and restore access.
**Covers:** Deny/delete CSR, Certificate expiration failures

---

### **Scenario 7: Resolve Pod Security Admission Rejections**

New Pods are rejected in a namespace due to enforced security policies. Identify the active Pod Security Admission mode, distinguish between Baseline and Restricted requirements, adjust namespace labels, and fix the rejected Pods.
**Covers:** Pod Security Admission modes, Baseline vs Restricted, Namespace PSA labels, Fix PSA rejections

## TIER 2D — API DISCOVERY
### **Scenario 8: Audit Cluster APIs and Fix Deprecated Manifests**

Workloads fail after a cluster upgrade. Discover available API resources and versions, identify deprecated APIs in manifests, update them to supported versions, and verify schema fields using API documentation tools.
**Covers:** kubectl api-resources, kubectl api-versions, Identify deprecated APIs, Fix deprecated manifests, kubectl explain, kubectl explain --recursive

---
##  **TIER 2E — HELM & KUSTOMIZE (EXPLICIT OBJECTIVE)**

### Scenario 1: Install a cluster component with Helm

**Task:** Add a Helm repo, install a chart into a specified namespace with required values, verify resources created, then upgrade the release to change a value and confirm rollout.
**Covers:** `helm repo add/update`, `helm install`, `helm upgrade`, `helm list/status`, values overrides, verify objects/rollout. ([Linux Foundation - Education][1])

### Scenario 2: Repair a broken Helm release

**Task:** A Helm release is stuck/failed (bad values or missing dependency). Identify the failing resource, fix via values or rollback, and return the release to healthy state.
**Covers:** `helm history`, `helm rollback`, `helm uninstall` (if instructed), reading chart-created resources.

### Scenario 3: Apply Kustomize overlays

**Task:** Given a base + overlay, apply the overlay to change image/tag, replicas, and labels; verify rendered output and applied state.
**Covers:** `kubectl apply -k`, `kustomize build` (if present), overlays/patches. ([Linux Foundation - Education][1])

### Scenario 4: Kustomize debugging

**Task:** `kubectl apply -k` fails due to name/namespace/patch targeting. Fix the kustomization so resources render and apply cleanly without changing base manifests.
**Covers:** common kustomize failure patterns.

## **TIER 2F — EXTENSION INTERFACES + CRDs/OPERATORS**

### Scenario 1: Diagnose “CNI/CRI/CSI” class failures from symptoms

**Task:** Pods fail with symptoms indicating either (a) CNI networking not set up, (b) CRI/runtime socket issues, or (c) CSI storage attach/mount issues. Classify which interface is implicated and identify the node/component to inspect first.
**Covers:** “Understand extension interfaces (CNI, CSI, CRI)” as an objective. ([Linux Foundation - Education][1])

### Scenario 2: Install an operator / CRD-backed component

**Task:** Apply manifests that introduce CRDs and an operator/controller, verify CRDs exist, verify controller is running, and create a sample Custom Resource that becomes Ready.
**Covers:** CRDs, operators install/verify, basic CR lifecycle. ([Linux Foundation - Education][1])

### Scenario 3: Debug a CRD/Operator mismatch

**Task:** A Custom Resource fails validation or is ignored because apiVersion/kind/schema doesn’t match what’s installed. Fix the manifest using discovery tools and re-apply.
**Covers:** `kubectl api-resources`, `kubectl explain`, CRD/version alignment.


## TIER 3A — CLUSTER ADMIN (KUBEADM)

Below is the **complete, finalized 100% coverage set** for **TIER 3A (kubeadm) + TIER 3B (etcd)**.
This version is **minimal, non-redundant, exam-realistic**, and fully compliant with your rules.

---

### **Scenario 0: kubeadm Preflight Failure (Infrastructure Prep)**

`kubeadm init` fails due to node-level preflight errors. Identify and fix the issue so initialization succeeds.
**Possible failure causes:** Container runtime not running; Wrong CRI socket; Cgroup driver mismatch; Swap enabled; Missing kernel modules / sysctl settings
**Covers:** Prepare underlying infrastructure, kubeadm preflight checks, CRI + kubelet alignment

---

### **Scenario 1: Initialize a New Control Plane with kubeadm**

Bootstrap a new Kubernetes control plane on a provided host using kubeadm with a specified control-plane endpoint and pod network CIDR. Ensure cluster access is configured for a non-root user.
**Covers:** kubeadm init, Control-plane endpoint, kubectl for non-root, Control-plane static manifests

---

### **Scenario 2: Add a Worker Node to an Existing Cluster**

Generate the required join command on the control plane, manage authentication tokens as needed, and successfully join a new worker node to the cluster.
**Covers:** Generate join command, Token management, Join worker node

---

### **Scenario 3: Worker Join Fails After Token Acceptance**

A worker successfully receives the join command but never becomes `Ready`. Diagnose and fix the node so it registers correctly.
**Failure causes:** Wrong CRI socket; CNI not installed yet; kubelet misconfiguration; Runtime not healthy
**Covers:** Node bootstrap troubleshooting, kubelet/runtime dependency, Join failure symptoms

---

### **Scenario 4: Join an Additional Control Plane Node**

Scale the control plane by joining a new control-plane node to the cluster and ensure it integrates correctly with the existing control-plane endpoint.
**Covers:** Join control-plane node, Control-plane endpoint, Stacked vs external etcd


---

### **Scenario 5: Inspect kubeadm Cluster Configuration**

Review the cluster bootstrap configuration stored by kubeadm to determine current settings and control-plane topology.
**Covers:** kubeadm config view, Stacked vs external etcd

---

### **Scenario 6: Safely Reset a Node Bootstrapped with kubeadm**

Remove a node from the cluster and reset it using kubeadm while ensuring no unintended data or configuration loss occurs.
**Covers:** kubeadm reset safety

---

### **Scenario 7: Upgrade the Kubernetes Control Plane**

Upgrade the control plane to a newer Kubernetes version while adhering to version skew rules and maintaining cluster availability.
**Covers:** Upgrade control plane, Version skew rules

---

### **Scenario 8: Upgrade Node Components After Control Plane Upgrade**

Upgrade kubelet and kubectl on cluster nodes to versions compatible with the upgraded control plane and validate node readiness.
**Covers:** Upgrade kubelet, Upgrade kubectl, Version skew rules

---

### **Scenario 9: Recover from a Failed Control Plane Upgrade**

An attempted control plane upgrade fails mid-process. Restore cluster functionality and bring all components to a consistent, supported state.
**Covers:** Failed upgrade recovery, Version skew rules

---

### **Scenario 10: Modify kubelet Configuration Using Supported Methods**

Adjust kubelet behavior by editing the kubelet configuration file and applying systemd drop-in changes without breaking node registration.
**Covers:** kubelet config file, kubelet systemd drop-ins

---

### **Scenario 11: Apply a Change to a Control-Plane Static Pod**

Update a control-plane component running as a static Pod by modifying its manifest and ensuring the component restarts correctly.
**Covers:** Control-plane static manifests, Static Pod edit workflow

### Scenario 12: Implement/verify a highly-available control plane

**Task:** Given a multi-control-plane design requirement, confirm the control-plane endpoint/load-balancer config, add/verify an additional control plane node, and validate HA behavior from kubeadm config and node state.
**Covers:** “Implement and configure a highly-available control plane” objective. ([Linux Foundation - Education][2])

---


# TIER 3B — ETCD

### **Scenario 13: Inspect etcd Cluster Membership and Health**

Determine current etcd cluster membership and verify that all etcd members are healthy using the correct certificates.
**Covers:** etcd member list, etcd health, etcd cert paths

---

### **Scenario 14: Create and Verify an etcd Snapshot**

Create a snapshot backup of etcd data and verify that the snapshot is valid and usable for restoration.
**Covers:** etcd snapshot, Verify snapshot, etcd data dir

---

### **Scenario 15: Restore etcd from a Snapshot**

Restore etcd from a snapshot after a simulated data loss event and bring the control plane back online.
**Covers:** Restore snapshot, Restore impact on cluster, etcd data dir

---

### **Scenario 16: Diagnose a Failed etcd Snapshot Restore**

An etcd snapshot restore fails. Identify and correct issues related to certificates, data directories, or restore configuration.
**Covers:** Snapshot restore failure causes, etcd cert paths, etcd data dir

---

### **Scenario 17: Recover Control Plane After etcd Restore**

After restoring etcd, the API server remains unavailable. Determine the etcd topology in use, verify certificate and data paths, update the kubelet configuration file if required, and restore control-plane availability while accounting for expected cluster impact.
**Covers:** kubelet config file, Stacked vs external etcd, Restore impact on cluster, etcd cert paths, etcd data dir, Control-plane static manifests

# TIER 3C — CONTROL PLANE & NODE INTERNALS

**1. Recover a Failing Static Control Plane Pod**
A control plane component is down after a configuration change on the node. Identify the static Pod manifest location, safely correct the configuration, and verify the component restarts automatically.
**Covers:** Static Pod locations, safe static Pod edits, static Pod restart behavior, static Pod failure recovery

---

**2. Diagnose API Server Unavailability**
Cluster users report kubectl commands hanging or failing. Inspect API server health endpoints and system behavior to determine the cause and impact of the failure.
**Covers:** API server health endpoints, API server failure symptoms

---

**3. Investigate Scheduling Failures**
New Pods remain in Pending state across the cluster. Determine whether the scheduler is running correctly by checking symptoms and relevant logs.
**Covers:** Scheduler failure symptoms, scheduler logs

---

**4. Resolve Controller Manager Instability**
Core controllers are not reconciling resources correctly. Identify controller-manager failures and assess how leader election is affecting cluster behavior.
**Covers:** Controller-manager failures, leader election impact

---

**5. Safely Perform Node Maintenance**
Prepare a worker node for maintenance by preventing new workloads, evicting existing Pods with appropriate options, and returning the node to service afterward.
**Covers:** Cordon, drain, drain flags, uncordon

---

**6. Troubleshoot a Node in NotReady State**
A node transitions to NotReady unexpectedly. Analyze kubelet status, logs, and configuration to identify the root cause.
**Covers:** Node NotReady causes, kubelet logs, kubelet config errors

---

**7. Diagnose Container Runtime Issues on a Node**
Pods fail to start and kubelet reports runtime-related errors. Investigate container runtime health and CRI socket configuration.
**Covers:** Container runtime failures, CRI socket mismatch symptoms

---

**8. Resolve Node Resource Pressure Conditions**
A node reports resource pressure conditions affecting scheduling and Pod stability. Determine which pressures are present and their operational impact.
**Covers:** Disk pressure, memory pressure, PID pressure

---

**9. Debug Network Plugin Failure**
Pods are running but cannot communicate over the network. Identify symptoms indicating a missing or broken CNI plugin and assess node-level impact.
**Covers:** CNI plugin missing/broken symptoms


