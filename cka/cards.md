# 1. kubectl SPEED (A1 + B1 FIRST)

## Task: Create a single Pod named nginx in the default namespace (NOT a Deployment)
Constraints:
- Must be a Pod
- Must start immediately
- No YAML
Command:
```bash
kubectl run nginx --image=nginx --restart=Never
```
Verify:
```bash
kubectl get pod nginx -o wide
```
Failure modes:
- Deployment created → forgot --restart=Never
- Pod Pending → image pull or node issue
Tag: memorize

## Task: Expose existing Deployment web internally on port 80 using a ClusterIP Service named web-svc
Command:
```bash
kubectl expose deploy web --name=web-svc --port=80 --target-port=80 --type=ClusterIP
```
Verify:
```bash
kubectl get svc web-svc
kubectl get endpoints web-svc
```
Common failures:
- Empty endpoints → selector mismatch or pods not Ready
- Wrong port → targetPort mismatch
Debug path:
svc → endpoints → pods → readinessProbe
Tag: memorize

## Task: Create a Deployment and expose it via ClusterIP Service (quick compose)
Command:
```bash
kubectl create deploy web --image=nginx
kubectl expose deploy web --port=80 --target-port=80 --type=ClusterIP
```
Verify:
```bash
kubectl get deploy web && kubectl get svc web -o wide
```
Pitfalls:
- Service selector must match Deployment labels (app=web by default).
- Expose uses first container port if targetPort omitted; set explicitly.
Tag: hands-on

## Task: Generate Deployment YAML without creating
Command:
```bash
kubectl create deploy web --image=nginx --dry-run=client -o yaml
```
Verify:
```bash
kubectl create deploy web --image=nginx --dry-run=client -o yaml | head -n 5
```
Pitfalls:
- --record deprecated; omit.
- Remember to set selector when editing YAML manually.
Tag: memorize

## Task: Apply/replace manifests fast (use diff first)
Command:
```bash
kubectl diff -f file.yaml
kubectl apply -f file.yaml
kubectl replace --force -f file.yaml   # deletes then creates
```
Verify:
```bash
kubectl get -f file.yaml
```
Pitfalls:
- replace --force briefly deletes resource; avoid on critical Services.
- apply caches last-applied; use kubectl diff before.
Tag: hands-on

## Task: Delete a Pod immediately
Command:
```bash
kubectl delete pod nginx --grace-period=0 --force
```
Verify:
```bash
kubectl get pod nginx
```
Pitfalls:
- Force delete may leave volumes mounted; check node for cleanup.
- If finalizers present, patch to remove first.
Tag: hands-on

## Task: Inspect live resources and rollout quickly
Command:
```bash
kubectl describe pod nginx
kubectl logs nginx
kubectl logs nginx --previous
kubectl rollout status deploy/web
kubectl rollout undo deploy/web
```
Verify:
```bash
kubectl get rs -l app=web -o wide
```
Pitfalls:
- Use --previous only when container restarted.
- rollout status blocks until complete; ctrl+c if stuck.
Tag: hands-on

## Task: Exec into a Pod shell quickly
Command:
```bash
kubectl exec -it nginx -- sh
```
Verify:
```bash
kubectl exec nginx -- echo ok
```
Pitfalls:
- alpine uses /bin/sh; bash may not exist.
- Ensure pod Running before exec.
Tag: hands-on

## Task: Label/annotate resources fast
Command:
```bash
kubectl label pod nginx tier=frontend --overwrite
kubectl annotate pod nginx owner=cka --overwrite
```
Verify:
```bash
kubectl get pod nginx --show-labels -o custom-columns=NAME:.metadata.name,ANN:.metadata.annotations.owner
```
Pitfalls:
- Forgetting --overwrite fails on existing values.
- Quotes required if annotation contains slashes.
Tag: memorize

## Task: Use jsonpath with newline for first Pod name
Command:
```bash
kubectl get pods -o jsonpath='{.items[0].metadata.name}{"\n"}'
```
Verify:
```bash
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'
```
Pitfalls:
- Always add {"\n"} to avoid glued output.
- Order is not deterministic without sort.
Tag: memorize

## Task: Patch Deployment image via strategic merge
Command:
```bash
kubectl patch deploy web -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","image":"nginx:1.26"}]}}}}'
```
Verify:
```bash
kubectl get deploy web -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```
Pitfalls:
- Ensure container name matches existing one.
- Patch preserves other fields; use apply for larger changes.
Tag: hands-on

## Task: Patch Service type to NodePort
Command:
```bash
kubectl patch svc web -p '{"spec":{"type":"NodePort"}}'
```
Verify:
```bash
kubectl get svc web -o wide
```
Pitfalls:
- Existing ClusterIP preserved; nodePort auto-assigned unless set.
- May disrupt clients expecting ClusterIP only.
Tag: hands-on

## Task: Sort and filter events quickly
Command:
```bash
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get events --field-selector type=Warning --sort-by=.metadata.creationTimestamp
```
Verify:
```bash
kubectl get events --field-selector involvedObject.name=<obj>
```
Pitfalls:
- Events rotate fast; capture early.
- API priority/flow control may limit large queries.
Tag: memorize

## Task: Discover correct YAML fields under time pressure using kubectl explain
Commands (MEMORIZE):
```bash
kubectl explain pod.spec --recursive
kubectl explain deployment.spec.strategy
kubectl explain service.spec.ports
```
Typical use:
- Unsure where a field belongs
- Fixing invalid YAML quickly
Rule:
If unsure → explain, do NOT guess
Tag: memorize

## Task: Create resource imperatively, modify YAML, then apply
Workflow (MEMORIZE):
```bash
kubectl create deploy web --image=nginx --dry-run=client -o yaml > web.yaml
vi web.yaml
kubectl apply -f web.yaml
```
Why this matters:
- Fastest way to build correct YAML
- Common exam expectation
Verify:
```bash
kubectl get deploy web
```
Tag: memorize

## Task: Ensure you are operating in the correct namespace
Commands (MEMORIZE):
```bash
kubectl config view --minify | grep namespace
kubectl get all -n <ns>
kubectl get events -n <ns> --sort-by=.lastTimestamp
```
Rule:
If something “doesn’t exist”, check namespace first.
Tag: memorize

## Task: Preview changes before applying YAML
Command:
```bash
kubectl diff -f file.yaml
```
Use when:
- Editing existing Deployments
- Fixing production-like resources
Exam tip:
Diff catches selector mismatches before failure.
Tag: optional

# 2. YAML SKELETONS (B6)

## Task: Deployment skeleton with selector matchLabels
Manifest:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        image: nginx:1.25
        ports:
        - containerPort: 80
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: app
        image: nginx:1.25
        ports:
        - containerPort: 80
EOF
```
Verify:
```bash
kubectl get deploy app
```
Pitfalls:
- selector must match template labels or apply fails.
- Strategy defaults to RollingUpdate.
Tag: memorize

## Task: DaemonSet skeleton
Manifest:
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-syslog
spec:
  selector:
    matchLabels:
      app: ds-syslog
  template:
    metadata:
      labels:
        app: ds-syslog
    spec:
      containers:
      - name: logger
        image: busybox:1.36
        command: ["sh","-c","sleep 3600"]
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-syslog
spec:
  selector:
    matchLabels:
      app: ds-syslog
  template:
    metadata:
      labels:
        app: ds-syslog
    spec:
      containers:
      - name: logger
        image: busybox:1.36
        command: ["sh","-c","sleep 3600"]
EOF
```
Verify:
```bash
kubectl get pods -l app=ds-syslog -o wide
```
Pitfalls:
- No replicas field; schedules one per Ready node (taints may block).
- Use tolerations to run on masters if tainted.
Tag: memorize

## Task: Job skeleton
Manifest:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: simple-job
spec:
  completions: 1
  backoffLimit: 3
  template:
    spec:
      restartPolicy: OnFailure
      containers:
      - name: job
        image: busybox:1.36
        command: ["sh","-c","echo hi"]
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: batch/v1
kind: Job
metadata:
  name: simple-job
spec:
  completions: 1
  backoffLimit: 3
  template:
    spec:
      restartPolicy: OnFailure
      containers:
      - name: job
        image: busybox:1.36
        command: ["sh","-c","echo hi"]
EOF
```
Verify:
```bash
kubectl get job simple-job -o wide
```
Pitfalls:
- restartPolicy must be Never/OnFailure for jobs.
- Use ttlSecondsAfterFinished to auto-clean if desired.
Tag: memorize

## Task: CronJob skeleton (batch/v1)
Manifest:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: simple-cj
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: job
            image: busybox:1.36
            command: ["sh","-c","date"]
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: batch/v1
kind: CronJob
metadata:
  name: simple-cj
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: job
            image: busybox:1.36
            command: ["sh","-c","date"]
EOF
```
Verify:
```bash
kubectl get cronjob simple-cj
```
Pitfalls:
- schedule uses standard cron format.
- Ensure jobs finish before next run to avoid overlap if not desired.
Tag: memorize

# 3. WORKLOADS & SCHEDULING (A2 + A2b)

## Task: Run a Pod with env vars and custom command
Command:
```bash
kubectl run envpod --image=busybox:1.36 --restart=Never \
  --env=APP=prod --command -- sh -c 'echo $APP && sleep 3600'
```
Verify:
```bash
kubectl logs envpod | head -n 1
```
Pitfalls:
- --command required when overriding entrypoint.
- Use explicit image tag to avoid pull latency.
Tag: hands-on

## Task: Apply Pod with multi-container and probes (YAML)
Manifest:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mc-probe
spec:
  containers:
  - name: app
    image: nginx
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
    readinessProbe:
      httpGet:
        path: /
        port: 80
      periodSeconds: 5
  - name: sidecar
    image: busybox:1.36
    command: ["sh","-c","sleep 3600"]
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: mc-probe
spec:
  containers:
  - name: app
    image: nginx
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
    readinessProbe:
      httpGet:
        path: /
        port: 80
      periodSeconds: 5
  - name: sidecar
    image: busybox:1.36
    command: ["sh","-c","sleep 3600"]
EOF
```
Verify:
```bash
kubectl get pod mc-probe -o wide
kubectl describe pod mc-probe | sed -n '/Events:/,$p'
```
Pitfalls:
- selector not needed for standalone Pod; avoid kubectl set probe (deprecated).
- initialDelaySeconds prevents early restarts.
Tag: hands-on

## Task: Update Deployment image safely
Command:
```bash
kubectl set image deploy/web nginx=nginx:1.25
kubectl rollout status deploy/web
```
Verify:
```bash
kubectl rollout history deploy/web
```
Pitfalls:
- Ensure Deployment selector matches pod labels when editing YAML.
- Watch for imagePullBackOff during rollout.
Tag: hands-on

## Task: Create a Job and view logs
Manifest:
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: pi
        image: perl
        command: ["perl","-Mbignum=bpi","-wle","print bpi(10)"]
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: pi
        image: perl
        command: ["perl","-Mbignum=bpi","-wle","print bpi(10)"]
EOF
```
Verify:
```bash
kubectl get jobs pi -o wide
kubectl logs job/pi
```
Pitfalls:
- restartPolicy must be Never/OnFailure.
- backoffLimit defaults to 6; adjust if needed.
Tag: hands-on

## Task: Create a CronJob (batch/v1) running every minute
Manifest:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cj
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: date
            image: busybox:1.36
            command: ["sh","-c","date; sleep 10"]
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cj
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: date
            image: busybox:1.36
            command: ["sh","-c","date; sleep 10"]
EOF
```
Verify:
```bash
kubectl get cronjob cj
kubectl get jobs -l job-name -o wide
```
Pitfalls:
- Use batch/v1; older apiVersions are invalid.
- Set startingDeadlineSeconds to avoid backlogged runs.
Tag: hands-on

## Task: Schedule Pod to labeled node via nodeSelector
Manifest:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: selector-pod
spec:
  nodeSelector:
    disktype: ssd
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh","-c","sleep 3600"]
```
Apply:
```bash
kubectl label node <node> disktype=ssd
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: selector-pod
spec:
  nodeSelector:
    disktype: ssd
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh","-c","sleep 3600"]
EOF
```
Verify:
```bash
kubectl get pod selector-pod -o wide
```
Pitfalls:
- Pod Pending if label missing; add before create.
- nodeSelector is exact match only.
Tag: hands-on

## Task: Use nodeAffinity requiredDuringScheduling
Manifest:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values: [ssd]
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh","-c","sleep 3600"]
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values: [ssd]
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh","-c","sleep 3600"]
EOF
```
Verify:
```bash
kubectl get pod affinity-pod -o wide
kubectl describe pod affinity-pod | sed -n '/Events:/,$p'
```
Pitfalls:
- Pending with FailedScheduling if no nodes match.
- Use topologyKey for podAntiAffinity; not nodeAffinity.
Tag: hands-on

## Task: Allow Pod on tainted node with toleration
Manifest:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tolerate
spec:
  tolerations:
  - key: key
    operator: Equal
    value: value
    effect: NoSchedule
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh","-c","sleep 3600"]
```
Apply:
```bash
kubectl taint nodes <node> key=value:NoSchedule
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: tolerate
spec:
  tolerations:
  - key: key
    operator: Equal
    value: value
    effect: NoSchedule
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh","-c","sleep 3600"]
EOF
```
Verify:
```bash
kubectl get pod tolerate -o wide
```
Pitfalls:
- Remove taint with kubectl taint nodes <node> key:NoSchedule- after test.
- Effect must match taint; otherwise still blocked.
Tag: hands-on

## Task: Create PriorityClass and Pod using it
Manifest:
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100000
preemptionPolicy: PreemptLowerPriority
---
apiVersion: v1
kind: Pod
metadata:
  name: prio-pod
spec:
  priorityClassName: high-priority
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh","-c","sleep 3600"]
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100000
preemptionPolicy: PreemptLowerPriority
---
apiVersion: v1
kind: Pod
metadata:
  name: prio-pod
spec:
  priorityClassName: high-priority
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh","-c","sleep 3600"]
EOF
```
Verify:
```bash
kubectl get pod prio-pod -o jsonpath='{.spec.priority}{"\n"}'
```
Pitfalls:
- Preemption may evict lower priority pods; be careful in shared clusters.
- Only one globalDefault PriorityClass allowed.
Tag: hands-on

## Task: Create PodDisruptionBudget for a Deployment
Manifest:
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: web
EOF
```
Verify:
```bash
kubectl get pdb web-pdb
kubectl describe pdb web-pdb
```
Pitfalls:
- Ensure Deployment has enough replicas to satisfy minAvailable.
- PDBs block voluntary evictions only (drain/evict), not pod failures.
Tag: hands-on

## Task: List init containers and termination reasons
Command:
```bash
kubectl get pod <pod> -o jsonpath='{.spec.initContainers[*].name}{"\n"}'
kubectl get pod <pod> -o jsonpath='{range .status.initContainerStatuses[*]}{.name}{"="}{.state.terminated.reason}{"\n"}{end}'
```
Verify:
```bash
kubectl describe pod <pod> | sed -n '/Init Containers:/,/Containers:/p'
```
Pitfalls:
- Add {"\n"} to jsonpath to avoid mashed output.
- initContainers array absent if none defined.
Tag: hands-on

## Task: Check resource requests/limits per container
Command:
```bash
kubectl get pod <pod> -o jsonpath='{range .spec.containers[*]}{.name}{" req="}{.resources.requests}{" lim="}{.resources.limits}{"\n"}{end}'
```
Verify:
```bash
kubectl describe pod <pod> | grep -A4 Limits
```
Pitfalls:
- Empty map means defaults/LimitRange may apply.
- Requests gate scheduling; limits control runtime OOMKill (137).
Tag: hands-on

# 4. TROUBLESHOOTING (A7)

## Task: Troubleshoot CrashLoopBackOff and fix root cause
Fast triage:
```bash
kubectl describe pod <pod> -n <ns> | sed -n '/Events:/,$p'
kubectl logs <pod> -n <ns> --previous
```
CrashLoopBackOff checklist (MEMORIZE):
1. logs --previous
2. command / args wrong?
3. probe killing container?
4. missing ConfigMap/Secret?
Common causes → fix:

* Cause: bad command/probe → Fix: kubectl edit/patch deployment to correct command or probe.
  Verify:
  ```bash
  kubectl rollout status deploy/<deploy> -n <ns>
  ```
* Cause: missing config/secret → Fix: mount correct ConfigMap/Secret.

Pitfalls:
- Use --previous to see failing attempt.
- CrashLoopBackOff delays restart; be patient before verifying.
Tag: hands-on

## Task: Troubleshoot ImagePullBackOff
Fast triage:
```bash
kubectl describe pod <pod> -n <ns> | grep -i image -A2
```
Common causes → fix:

* Cause: bad image → Fix:
  ```bash
  kubectl set image deploy/<d> <c>=<image>:<tag> -n <ns>
  ```
  Verify:
  ```bash
  kubectl rollout status deploy/<d> -n <ns>
  ```
* Cause: private registry → Fix: create imagePullSecret and set in pod spec.

Pitfalls:
- ImagePolicyWebhook can block latest tag; specify digest/tag.
- Secret must be in same namespace and referenced.
Tag: hands-on

## Task: Pod <pod> is Pending. Identify the scheduling reason and make it Running without adding nodes.
Step-by-step:
```bash
kubectl describe pod <pod> -n <ns>
kubectl get nodes --show-labels
kubectl describe node <node>
```
Decision checklist (MEMORIZE):
1. Insufficient CPU/Mem → lower requests
2. Taint present → add toleration
3. NodeSelector/Affinity mismatch → fix labels
4. TopologySpreadConstraints unsatisfied → relax or fix labels
Verify:
```bash
kubectl get pod <pod> -n <ns>
```
Tag: hands-on

## Task: Service <svc> has no endpoints. Restore traffic.
Fast path (MEMORIZE):
SELECTOR → LABELS → READINESS → TARGETPORT
Commands:
```bash
kubectl get svc <svc> -n <ns> -o yaml
kubectl get endpoints <svc> -n <ns>
kubectl get pods -l <selector> -n <ns> -o wide
kubectl describe pod <pod>
```
Fix:
```bash
kubectl patch svc <svc> -p '{"spec":{"selector":{"app":"web"}}}'
```
Verify:
```bash
kubectl get endpoints <svc>
```
Tag: hands-on

## Task: Troubleshoot DNS failures end-to-end
Fast triage:
```bash
kubectl run -it --rm dnsdiag --image=busybox:1.36 --restart=Never -- nslookup kubernetes.default
kubectl get svc -n kube-system kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns --tail=20
```
Common causes → fix:

* Cause: kube-proxy broken → Fix: check kube-proxy pods/logs; restart DS pod.
  Verify:
  ```bash
  kubectl -n kube-system get ds kube-proxy -o wide
  ```
* Cause: NetworkPolicy blocks DNS → Fix: allow udp/53 to kube-dns.

Pitfalls:
- If CoreDNS Service ClusterIP missing, recreate from manifest.
- Ensure resolv.conf inside pods uses cluster DNS IP.
Tag: hands-on

## Task: Troubleshoot Node NotReady
Fast triage:
```bash
kubectl describe node <node>
sudo journalctl -u kubelet -n 50
```
Node NotReady checklist (MEMORIZE):
DESCRIBE NODE
→ KUBELET LOGS
→ CONTAINER RUNTIME
→ DISK/MEM PRESSURE
Common causes → fix:

* Cause: kubelet cert expired → Fix: sudo kubeadm cert renew kubelet.conf && sudo systemctl restart kubelet.
  Verify:
  ```bash
  kubectl get nodes
  ```
* Cause: disk/memory pressure → Fix: free space, prune images; restart kubelet.

Pitfalls:
- Network partition can show Ready,SchedulingDisabled; check routes.
- Check container runtime status (systemctl status containerd/crio).
Tag: hands-on

## Task: Troubleshoot PVC Pending (dynamic provisioning)
Fast triage:
```bash
kubectl describe pvc <pvc>
kubectl get events --field-selector involvedObject.name=<pvc> --sort-by=.lastTimestamp
kubectl get pods -A | grep csi
```
PVC Pending checklist (MEMORIZE):
1. storageClassName exists?
2. default StorageClass?
3. accessMode supported?
4. CSI pods running?
Common causes → fix:

* Cause: no default SC → Fix: set annotation on SC or specify storageClassName.
  Verify:
  ```bash
  kubectl patch storageclass <sc> -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
  ```
* Cause: driver pods down → Fix: restart CSI controller/daemonset.

Pitfalls:
- Namespace mismatch between pod and PVC leads to Pending pod not PVC.
- ReadWriteMany request on RWO-only provisioner stays Pending.
Tag: hands-on

## Task: CSI troubleshooting basics
Fast triage:
```bash
kubectl get csidrivers,csinodes,volumeattachments
kubectl get pods -A | grep -E 'csi|storage'
kubectl describe pvc <pvc>
```
Common causes → fix:

* Cause: node plugin down → Fix: check DaemonSet pods in kube-system; restart node plugin pod.
  Verify:
  ```bash
  kubectl get pods -A | grep csi-node
  ```
* Cause: attach timeout → Fix: inspect volumeattachment events.

Pitfalls:
- VolumeAttachments require RBAC; run as admin.
- Some drivers log to kubelet; check journal if events sparse.
Tag: hands-on

# 5. NETWORKING (A3 + B3)

## Task: Expose Deployment via NodePort and verify endpoint
Command:
```bash
kubectl expose deploy web --type=NodePort --port=80 --target-port=80 --name=web-nodeport
```
Verify:
```bash
kubectl get svc web-nodeport -o wide
kubectl get endpoints web-nodeport -o wide
```
Pitfalls:
- NodePorts default 30000-32767; specify --node-port to avoid collisions.
- Pods must be Ready to appear in endpoints.
Tag: hands-on

## Task: Create LoadBalancer Service (generic) and watch external IP
Command:
```bash
kubectl expose deploy web --type=LoadBalancer --port=80 --target-port=80 --name=web-lb
kubectl get svc web-lb -w
```
Verify:
```bash
kubectl get svc web-lb -o jsonpath='{.status.loadBalancer.ingress[*].ip}{"\n"}'
```
Pitfalls:
- On bare metal, External IP may stay <pending>; use nodePort fallback.
- healthCheckNodePort may be required by some providers.
Tag: hands-on

## Task: Create Ingress routing host/path to Service
Manifest:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ing
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 80
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ing
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 80
EOF
```
Verify:
```bash
kubectl get ingress web-ing
kubectl describe ingress web-ing | sed -n '/Rules:/,$p'
```
Pitfalls:
- Requires ingress controller; check events for class errors.
- pathType is mandatory (Prefix/Exact).
Tag: hands-on

## Task: Debug in-cluster DNS quickly
Fast triage:
```bash
kubectl run -n default -it --rm dnscheck --image=busybox:1.36 --restart=Never -- nslookup kubernetes.default
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns --tail=20
```
Common causes → fix:

* Cause: CoreDNS CrashLoop → Fix: describe/logs, check ConfigMap kube-dns.
  Verify:
  ```bash
  kubectl get pods -n kube-system -l k8s-app=kube-dns -o wide
  ```
* Cause: NetworkPolicy blocking DNS → Fix: allow udp/53 to kube-dns.

Pitfalls:
- metrics-server outages do not affect DNS; avoid misdiagnosis.
- BusyBox dig may be missing; use nslookup.
Tag: hands-on

## Task: Create headless Service for StatefulSet DNS
Manifest:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-headless
spec:
  clusterIP: None
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: web-headless
spec:
  clusterIP: None
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
EOF
```
Verify:
```bash
kubectl get endpoints web-headless -o wide
```
Pitfalls:
- Without selector, no endpoints generated.
- StatefulSet requires headless svc for stable DNS.
Tag: hands-on

## Task: Default-deny ingress NetworkPolicy and allow specific namespace
Manifest:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes: [Ingress]
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-prod-ingress
spec:
  podSelector:
    matchLabels:
      app: web
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          access: allowed
```
Apply:
```bash
kubectl label namespace prod access=allowed
kubectl apply -n default -f - <<'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes: [Ingress]
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-prod-ingress
spec:
  podSelector:
    matchLabels:
      app: web
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          access: allowed
EOF
```
Verify:
```bash
kubectl get netpol -n default
kubectl run -n prod -it --rm prodtest --image=busybox:1.36 --restart=Never -- wget -qO- web.default.svc.cluster.local
```
Pitfalls:
- Policies are additive; ensure allow rules exist for required traffic.
- CNI must support NetworkPolicy (Calico/Cilium/Weave).
Tag: hands-on

## Task: Allow egress DNS after default-deny egress
Manifest:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
spec:
  podSelector: {}
  policyTypes: [Egress]
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
spec:
  podSelector: {}
  policyTypes: [Egress]
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
```
Apply:
```bash
kubectl apply -n default -f - <<'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
spec:
  podSelector: {}
  policyTypes: [Egress]
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
spec:
  podSelector: {}
  policyTypes: [Egress]
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
EOF
```
Verify:
```bash
kubectl run -it --rm dnstest --image=busybox:1.36 --restart=Never -- nslookup kubernetes.default
```
Pitfalls:
- Ensure CoreDNS labels match k8s-app=kube-dns.
- If TCP DNS needed, add TCP port 53.
Tag: hands-on

# 6. STORAGE (A4 + B4)

## Task: Create PVC with specific StorageClass via YAML
Manifest:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-claim
spec:
  accessModes: ["ReadWriteOnce"]
  storageClassName: standard
  resources:
    requests:
      storage: 1Gi
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-claim
spec:
  accessModes: ["ReadWriteOnce"]
  storageClassName: standard
  resources:
    requests:
      storage: 1Gi
EOF
```
Verify:
```bash
kubectl describe pvc data-claim
```
Pitfalls:
- Avoid kubectl create pvc shortcuts; YAML ensures SC/accessModes set.
- Pending indicates missing PV or provisioner.
Tag: hands-on

## Task: Create PVC ReadWriteMany
Manifest:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-claim
spec:
  accessModes: ["ReadWriteMany"]
  resources:
    requests:
      storage: 1Gi
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-claim
spec:
  accessModes: ["ReadWriteMany"]
  resources:
    requests:
      storage: 1Gi
EOF
```
Verify:
```bash
kubectl get pvc shared-claim -o wide
```
Pitfalls:
- Many provisioners do not support RWX; expect Pending on unsupported storage.
- Specify storageClassName if default lacks RWX.
Tag: hands-on

## Task: Mount PVC into Pod and verify IO
Manifest:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh","-c","echo ok > /data/test && cat /data/test && sleep 3600"]
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: data-claim
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh","-c","echo ok > /data/test && cat /data/test && sleep 3600"]
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: data-claim
EOF
```
Verify:
```bash
kubectl logs pvc-pod | head -n 1
```
Pitfalls:
- Pod and PVC must be in same namespace.
- Pending PVC keeps pod Pending; describe pod events.
Tag: hands-on

## Task: Create PV with Retain reclaim policy
Manifest:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-retain
spec:
  capacity:
    storage: 1Gi
  accessModes: ["ReadWriteOnce"]
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/pv-retain
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-retain
spec:
  capacity:
    storage: 1Gi
  accessModes: ["ReadWriteOnce"]
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/pv-retain
EOF
```
Verify:
```bash
kubectl get pv pv-retain -o wide
```
Pitfalls:
- hostPath PVs bind to specific node; pods must schedule there.
- Recycle is deprecated; use Retain/Delete only.
Tag: hands-on

## Task: Resize PVC (if StorageClass allows expansion)
Command:
```bash
kubectl patch pvc data-claim -p '{"spec":{"resources":{"requests":{"storage":"2Gi"}}}}'
```
Verify:
```bash
kubectl get pvc data-claim
```
Pitfalls:
- StorageClass must set allowVolumeExpansion: true.
- Filesystem expansion may require pod restart depending on driver.
Tag: hands-on

## Task: Identify default StorageClass and provisioner
Command:
```bash
kubectl get sc
kubectl get sc -o jsonpath='{range .items[*]}{.metadata.name}{":"}{.metadata.annotations.storageclass\.kubernetes\.io/is-default-class}{"\n"}{end}'
```
Verify:
```bash
kubectl describe sc <sc-name>
```
Pitfalls:
- Only one default should be true; unset others if duplicates.
- Provisioner name hints CSI driver to inspect.
Tag: memorize

## Task: StorageClass allowVolumeExpansion note
Command:
```bash
kubectl get sc -o jsonpath='{range .items[*]}{.metadata.name}{" expand="}{.allowVolumeExpansion}{"\n"}{end}'
```
Verify:
```bash
kubectl describe sc <sc>
```
Pitfalls:
- Expansion only works for supported drivers.
- Must patch PVC to larger size; cannot shrink.
Tag: memorize

## Task: Create StatefulSet with volumeClaimTemplates
Manifest:
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: web-headless
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: web-headless
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
EOF
```
Verify:
```bash
kubectl get pvc -l app=web
```
Pitfalls:
- Headless Service required for stable network IDs.
- VolumeClaimTemplates immutable after creation.
Tag: hands-on

# 7. RBAC & AUTH (A6)

## Task: Create Role and RoleBinding for ServiceAccount
Command:
```bash
kubectl create role pod-reader --verb=get,list,watch --resource=pods -n dev
kubectl create serviceaccount app-sa -n dev
kubectl create rolebinding read-pods --role=pod-reader --serviceaccount=dev:app-sa -n dev
```
Verify:
```bash
kubectl auth can-i list pods --as=system:serviceaccount:dev:app-sa -n dev
```
Pitfalls:
- Namespace must match Role/RoleBinding.
- Use resourceNames for narrowed permissions if needed.
Tag: hands-on

## Task: Create ClusterRoleBinding to SA
Command:
```bash
kubectl create clusterrole ns-reader --verb=get,list --resource=namespaces
kubectl create clusterrolebinding ns-read --clusterrole=ns-reader --serviceaccount=dev:app-sa
```
Verify:
```bash
kubectl auth can-i list namespaces --as=system:serviceaccount:dev:app-sa
```
Pitfalls:
- ClusterRoleBinding is cluster-scoped; no -n flag.
- Avoid granting cluster-admin unless required.
Tag: hands-on

## Task: Build kubeconfig for a new user with client certs
Command:
```bash
openssl genrsa -out dev.key 2048
openssl req -new -key dev.key -out dev.csr -subj "/CN=dev/O=dev"
sudo openssl x509 -req -in dev.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out dev.crt -days 365
kubectl config set-credentials dev --client-certificate=dev.crt --client-key=dev.key --embed-certs=true
kubectl config set-context dev --cluster=kubernetes --user=dev --namespace=dev
kubectl config use-context dev
```
Verify:
```bash
kubectl --context dev auth can-i list pods
```
Pitfalls:
- Ensure kubeconfig cluster entry points to correct apiserver and CA.
- Bind permissions via Role/ClusterRole; without binding, access is denied.
Tag: hands-on

## Task: Run Pod using a ServiceAccount
Manifest:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sa-pod
spec:
  serviceAccountName: app-sa
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh","-c","sleep 3600"]
```
Apply:
```bash
kubectl apply -n dev -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: sa-pod
spec:
  serviceAccountName: app-sa
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh","-c","sleep 3600"]
EOF
```
Verify:
```bash
kubectl get pod sa-pod -n dev -o jsonpath='{.spec.serviceAccountName}{"\n"}'
```
Pitfalls:
- ServiceAccount must exist in same namespace.
- automountServiceAccountToken can be disabled; set explicitly if needed.
Tag: hands-on

## Task: Approve or deny CSRs
Command:
```bash
kubectl get csr
kubectl describe csr <name>
kubectl certificate approve <name>
# or deny
kubectl certificate deny <name>
```
Verify:
```bash
kubectl get csr <name> -o wide
```
Pitfalls:
- CSR signs user or kubelet cert; verify CN/Organization before approval.
- After approval, kubelet may need restart to pick up cert.
Tag: hands-on

# 8. CLUSTER ADMIN (A5 + B5)

## Task: Cordon, drain, uncordon a node safely
Command:
```bash
kubectl cordon <node>
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
kubectl uncordon <node>
```
Verify:
```bash
kubectl get nodes -o wide
```
Pitfalls:
- PDBs can block drain; use --disable-eviction only if necessary.
- Remove taints before expecting pods to reschedule there.
Tag: hands-on

## Task: Take etcd snapshot and validate
Command:
```bash
ETCDCTL_API=3 etcdctl snapshot save /var/lib/etcd/snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```
Verify:
```bash
ETCDCTL_API=3 etcdctl snapshot status /var/lib/etcd/snapshot.db
```
Pitfalls:
- Run on control-plane hosting etcd static pod.
- Ensure file permissions allow kube-apiserver to read after restore.
Tag: hands-on

## Task: Restore etcd snapshot and rewire static pod
Fast triage:
```bash
ETCDCTL_API=3 etcdctl snapshot restore /var/lib/etcd/snapshot.db --data-dir=/var/lib/etcd-restore
```
Common causes → fix:

* Cause: static pod still points to old dir → Fix: edit /etc/kubernetes/manifests/etcd.yaml volume hostPath and --data-dir to /var/lib/etcd-restore.
  Verify:
  ```bash
  kubectl get pods -n kube-system -w | grep etcd
  ```

Pitfalls:
- Remove old member data-dir if conflicts.
- Kubelet auto-restarts static pod after manifest edit.
Tag: hands-on

## Task: Initialize control plane with kubeadm and set kubeconfig
Command:
```bash
kubeadm init --pod-network-cidr=192.168.0.0/16
mkdir -p $HOME/.kube
cp /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```
Verify:
```bash
kubectl get nodes
```
Pitfalls:
- Apply CNI manifest after init; nodes stay NotReady until then.
- For root, $HOME may be /root; adjust paths for other users.
Tag: hands-on

## Task: Join worker node with kubeadm
Command:
```bash
kubeadm join <cp-ip>:6443 --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```
Verify:
```bash
kubectl get nodes -o wide
```
Pitfalls:
- Token expires; regenerate with kubeadm token create --print-join-command.
- Ensure required ports open (6443, 10250).
Tag: hands-on

## Task: Approve pending node client CSRs (bootstrap)
Command:
```bash
kubectl get csr
kubectl certificate approve $(kubectl get csr --no-headers | awk '/Pending/{print $1}')
```
Pitfalls:
- Node authorizer still blocks if node name mismatch; check kubelet `--hostname-override`.
- For rotated certs stuck in Pending, delete CSR and let kubelet recreate.
Tag: hands-on

## Task: Upgrade control plane then kubelet/kubectl
Command:
```bash
kubeadm upgrade plan
kubeadm upgrade apply v1.X.Y
apt-get install -y kubelet=1.X.Y-00 kubectl=1.X.Y-00
systemctl restart kubelet
```
Verify:
```bash
kubectl get nodes -o wide
```
Pitfalls:
- Upgrade control plane first; respect version skew (kubelet <= apiserver <= kubeadm).
- Drain nodes before upgrading kubelet when possible.
Tag: hands-on

## Task: Enable encryption at rest for Secrets
Command:
```bash
sudo cat >/etc/kubernetes/encryption-config.yaml <<'EOF'
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
- resources: ["secrets"]
  providers:
  - aescbc:
      keys:
      - name: key1
        secret: $(head -c 32 /dev/urandom | base64)
  - identity: {}
EOF
sudo sed -i 's#--encryption-provider-config=.*#--encryption-provider-config=/etc/kubernetes/encryption-config.yaml#' /etc/kubernetes/manifests/kube-apiserver.yaml
```
Verify:
```bash
kubectl get secrets -A >/tmp/secrets.txt
sudo hexdump -C /var/lib/etcd/member/snap/db | grep -m1 -i $(echo -n default-token | xxd -p | head -c 8)
```
Pitfalls:
- kube-apiserver manifest edit restarts static pod; expect brief API blip.
- Run `kubectl get secrets -A --all-namespaces | xargs -I{} kubectl get {} -o yaml >/dev/null` after enabling to rewrite data with encryption.
Tag: hands-on

## Task: Renew control-plane certs
Command:
```bash
kubeadm cert renew all
systemctl restart kubelet
```
Verify:
```bash
kubectl get nodes && kubectl get pods -n kube-system
```
Pitfalls:
- Backup /etc/kubernetes/pki before renewal in real clusters.
- Restart apiserver may briefly disrupt kubectl.
Tag: hands-on

## Task: Generate kubeadm join command (bootstrap token)
Command:
```bash
kubeadm token create --print-join-command
```
Verify:
```bash
kubeadm token list
```
Pitfalls:
- Use --ttl 0 for non-expiring token when allowed.
- Hash from /etc/kubernetes/pki/ca.crt via openssl x509 -pubkey ... if missing.
Tag: memorize

## Task: Configure kubectl for non-root user
Command:
```bash
sudo mkdir -p /home/user/.kube
sudo cp /etc/kubernetes/admin.conf /home/user/.kube/config
sudo chown user:user /home/user/.kube/config
```
Verify:
```bash
sudo -u user kubectl get nodes
```
Pitfalls:
- Ensure user shell honors $KUBECONFIG if not default path.
- File permissions must allow read by user.
Tag: hands-on

## Task: Locate and edit static pod manifests
Command:
```bash
grep staticPodPath /var/lib/kubelet/config.yaml || echo /etc/kubernetes/manifests
echo "edit files then kubelet auto-restarts static pods"
```
Verify:
```bash
kubectl get pods -n kube-system -w | grep -E 'kube-apiserver|kube-scheduler|kube-controller-manager|etcd'
```
Pitfalls:
- Editing manifests can briefly stop API; use care on single-node control plane.
- Ensure YAML indentation remains valid; kubelet loops on invalid files.
Tag: hands-on

## Task: Diagnose container runtime failure (containerd/crio)
Command:
```bash
sudo systemctl status containerd || sudo systemctl status crio
sudo crictl ps -a
```
Verify:
```bash
sudo systemctl restart containerd && crictl ps
```
Pitfalls:
- crictl requires runtime endpoint; configure /etc/crictl.yaml if missing.
- Runtime failure keeps kubelet NotReady.
Tag: hands-on

## Task: API server health probe locally
Command:
```bash
kubectl get --raw "/readyz"
```
Verify:
```bash
kubectl get --raw "/livez"
```
Pitfalls:
- Use /readyz and /livez; inspect static pod logs via node runtime if API is flaky.
- Use admin.conf or correct kubeconfig.
Tag: memorize

# 9. AUTOSCALING & QUOTAS

## Task: Horizontal Pod Autoscaler for Deployment
Command:
```bash
kubectl autoscale deploy web --min=2 --max=5 --cpu-percent=70
```
Verify:
```bash
kubectl describe hpa web
```
Pitfalls:
- Requires metrics-server; HPA shows Unknown without metrics.
- HPA scales on requests, not limits.
Tag: hands-on

## Task: HPA YAML skeleton (cpu utilization)
Manifest:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```
Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
EOF
```
Verify:
```bash
kubectl get hpa web-hpa
```
Pitfalls:
- apiVersion autoscaling/v2 required for advanced metrics.
- Ensure Deployment has resource requests set.
Tag: hands-on

## Task: Monitor resource usage quickly
Command:
```bash
kubectl top node
kubectl top pod -A
```
Verify:
```bash
kubectl top pod web-xxxxx
```
Pitfalls:
- metrics-server must be running; otherwise “metrics not available”.
- Values are near real-time; not historical.
Tag: hands-on

## Task: Create ResourceQuota and LimitRange in namespace
Manifest:
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-basic
spec:
  hard:
    pods: "10"
    requests.cpu: "2"
    requests.memory: 2Gi
    limits.cpu: "4"
    limits.memory: 4Gi
---
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-basic
spec:
  limits:
  - default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 200m
      memory: 256Mi
    type: Container
```
Apply:
```bash
kubectl apply -n dev -f - <<'EOF'
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-basic
spec:
  hard:
    pods: "10"
    requests.cpu: "2"
    requests.memory: 2Gi
    limits.cpu: "4"
    limits.memory: 4Gi
---
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-basic
spec:
  limits:
  - default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 200m
      memory: 256Mi
    type: Container
EOF
```
Verify:
```bash
kubectl describe resourcequota rq-basic -n dev
kubectl describe limitrange lr-basic -n dev
```
Pitfalls:
- Quota breaches block creation; check events for “exceeded quota”.
- LimitRange defaults apply only when requests/limits omitted.
Tag: hands-on

# 10. TOOLING & DEBUG EXTRAS

## Task: Install/upgrade Helm chart with custom values
Command:
```bash
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm repo update
helm upgrade --install metrics-server metrics-server/metrics-server -n kube-system \
  --set args[0]='--kubelet-insecure-tls'
```
Verify:
```bash
helm list -A | grep metrics-server
kubectl get deploy -n kube-system metrics-server
```
Pitfalls:
- --install ensures creation if release missing.
- Use -f values.yaml for larger overrides.
Tag: hands-on

## Task: Render Helm template without installing
Command:
```bash
helm template myrelease metrics-server/metrics-server --namespace kube-system > rendered.yaml
```
Verify:
```bash
head -n 5 rendered.yaml
```
Pitfalls:
- Template uses default values if none supplied; pass -f for overrides.
- Keep apiVersions current after render.
Tag: hands-on

## Task: Apply base + overlay using Kustomize
Command:
```bash
kubectl apply -k overlays/prod
```
Verify:
```bash
kubectl kustomize overlays/prod | head -n 5
```
Pitfalls:
- Ensure kustomization.yaml exists in directory.
- overlays inherit from base; confirm namespace patches.
Tag: hands-on

## Task: Use kubectl port-forward and proxy for quick validation (OPTIONAL)
Command:
```bash
kubectl port-forward svc/web 8080:80 &
# open new shell
kubectl proxy --port=8001 &
```
Verify:
```bash
curl -s http://127.0.0.1:8080
curl -s http://127.0.0.1:8001/api
```
Pitfalls:
- port-forward blocks terminal; use background (&) carefully.
- proxy exposes API on localhost only by default.
Tag: optional

## Task: kubectl debug with ephemeral container (OPTIONAL)
Command:
```bash
kubectl debug pod/<pod> -it --image=busybox:1.36 --target=<container>
```
Verify:
```bash
kubectl get pod <pod> -o jsonpath='{.spec.ephemeralContainers[*].name}{"\n"}'
```
Pitfalls:
- Requires EnableEphemeralContainers feature (usually on >=1.25).
- Ephemeral containers share namespaces but no ports.
Tag: hands-on

# LOW-ROI / LAST / SKIM ONLY

## Task: Gateway API quick triage
Fast triage:
```bash
kubectl get gatewayclasses,gateway,httproutes -A
kubectl describe httproute <route> -n <ns>
kubectl describe gateway <gw> -n <ns>
```
Common causes → fix:

* Cause: parentRefs mismatch → Fix: set httproute.spec.parentRefs to existing Gateway name/namespace.
  Verify:
  ```bash
  kubectl get httproute <route> -n <ns> -o yaml | grep parentRefs -A5
  ```
* Cause: backend Service wrong port → Fix: update backendRefs.port.

Pitfalls:
- Controller-specific; ensure gatewayclass has active controller.
- Check events on Gateway/HTTPRoute for attachment errors.
Tags: low-roi last skim-only hands-on
