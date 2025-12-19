# A1. kubectl CORE MECHANICS (HIGHEST ROI)

## Task: Create a single Pod named nginx quickly in default ns
Tags: domain:workloads action:create format:kubectl
Command:
```bash
kubectl run nginx --image=nginx --restart=Never
```

Verify:
```bash
kubectl get pod nginx -o wide
```

Pitfalls:

* Forgetting --restart=Never creates Deployment instead.
* Pod may stay ContainerCreating if image pull issues.

## Task: Create a Deployment and expose it via ClusterIP Service
Tags: domain:workloads action:create format:kubectl
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

* Service selector must match Deployment labels (app=web by default).
* Expose uses first container port if targetPort omitted; set explicitly.

## Task: Generate Deployment YAML without creating
Tags: domain:workloads action:observe format:kubectl
Command:
```bash
kubectl create deploy web --image=nginx --dry-run=client -o yaml
```

Verify:
```bash
kubectl create deploy web --image=nginx --dry-run=client -o yaml | head -n 5
```

Pitfalls:

* --record deprecated; omit.
* Remember to set selector when editing YAML manually.

## Task: Apply/replace manifests fast
Tags: domain:workloads action:apply format:kubectl
Command:
```bash
kubectl apply -f file.yaml
kubectl replace --force -f file.yaml   # deletes then creates
```

Verify:
```bash
kubectl get -f file.yaml
```

Pitfalls:

* replace --force briefly deletes resource; avoid on critical Services.
* apply caches last-applied; use kubectl diff before.

## Task: Delete a Pod immediately
Tags: domain:workloads action:delete format:kubectl
Command:
```bash
kubectl delete pod nginx --grace-period=0 --force
```

Verify:
```bash
kubectl get pod nginx
```

Pitfalls:

* Force delete may leave volumes mounted; check node for cleanup.
* If finalizers present, patch to remove first.

## Task: Inspect live resources and rollout
Tags: domain:workloads action:observe format:kubectl
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

* Use --previous only when container restarted.
* rollout status blocks until complete; ctrl+c if stuck.

## Task: Exec into a Pod shell quickly
Tags: domain:workloads action:debug format:kubectl
Command:
```bash
kubectl exec -it nginx -- sh
```

Verify:
```bash
kubectl exec nginx -- echo ok
```

Pitfalls:

* alpine uses /bin/sh; bash may not exist.
* Ensure pod Running before exec.

## Task: Label/annotate resources fast
Tags: domain:workloads action:edit format:kubectl
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

* Forgetting --overwrite fails on existing values.
* Quotes required if annotation contains slashes.

## Task: Use jsonpath with newline for first Pod name
Tags: domain:workloads action:observe format:kubectl
Command:
```bash
kubectl get pods -o jsonpath='{.items[0].metadata.name}{"\n"}'
```

Verify:
```bash
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}'
```

Pitfalls:

* Always add {"\n"} to avoid glued output.
* Order is not deterministic without sort.

# A2. WORKLOADS (PODS, DEPLOYMENTS, JOBS)

## Task: Run a Pod with env vars and custom command
Tags: domain:workloads action:create format:kubectl
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

* --command required when overriding entrypoint.
* Use explicit image tag to avoid pull latency.

## Task: Apply Pod with multi-container and probes (YAML)
Tags: domain:workloads action:create format:yaml
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

* selector not needed for standalone Pod; avoid kubectl set probe (deprecated).
* initialDelaySeconds prevents early restarts.

## Task: Update Deployment image safely
Tags: domain:workloads action:edit format:kubectl
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

* Ensure Deployment selector matches pod labels when editing YAML.
* Watch for imagePullBackOff during rollout.

## Task: Create a Job and view logs
Tags: domain:workloads action:create format:yaml
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

* restartPolicy must be Never/OnFailure.
* backoffLimit defaults to 6; adjust if needed.

## Task: Create a CronJob (batch/v1) running every minute
Tags: domain:workloads action:create format:yaml
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

* Use batch/v1; older apiVersions are invalid.
* Set startingDeadlineSeconds to avoid backlogged runs.

# A2b. SCHEDULING & PLACEMENT ESSENTIALS

## Task: Schedule Pod to labeled node via nodeSelector
Tags: domain:scheduling action:create format:yaml
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

* Pod Pending if label missing; add before create.
* nodeSelector is exact match only.

## Task: Use nodeAffinity requiredDuringScheduling
Tags: domain:scheduling action:create format:yaml
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

* Pending with FailedScheduling if no nodes match.
* Use topologyKey for podAntiAffinity; not nodeAffinity.

## Task: Allow Pod on tainted node with toleration
Tags: domain:scheduling action:create format:yaml
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

* Remove taint with kubectl taint nodes <node> key:NoSchedule- after test.
* Effect must match taint; otherwise still blocked.

## Task: Create PriorityClass and Pod using it
Tags: domain:scheduling action:create format:yaml
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

* Preemption may evict lower priority pods; be careful in shared clusters.
* Only one globalDefault PriorityClass allowed.

## Task: Create PodDisruptionBudget for a Deployment
Tags: domain:scheduling action:create format:yaml
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

* Ensure Deployment has enough replicas to satisfy minAvailable.
* PDBs block voluntary evictions only (drain/evict), not pod failures.

# A3. NETWORKING (CORE ONLY)

## Task: Expose Deployment via NodePort and verify endpoint
Tags: domain:net action:create format:kubectl
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

* NodePorts default 30000-32767; specify --node-port to avoid collisions.
* Pods must be Ready to appear in endpoints.

## Task: Create LoadBalancer Service (generic) and watch external IP
Tags: domain:net action:create format:kubectl
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

* On bare metal, External IP may stay <pending>; use nodePort fallback.
* healthCheckNodePort may be required by some providers.

## Task: Create Ingress routing host/path to Service
Tags: domain:net action:create format:yaml
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

* Requires ingress controller; check events for class errors.
* pathType is mandatory (Prefix/Exact).

## Task: Check Service endpoints and fix selector mismatch
Tags: domain:net action:fix format:kubectl
Fast triage:
```bash
kubectl get svc web -o wide
kubectl get endpoints web -o wide
kubectl get pods -l app=web -o wide
```

Common causes → fix:

* Cause: selector wrong → Fix:
  ```bash
  kubectl patch svc web -p '{"spec":{"selector":{"app":"web"}}}'
  ```
  Verify:
  ```bash
  kubectl get endpoints web -o wide
  ```
* Cause: pods not Ready → Fix: inspect describe/events/logs for readiness.

Pitfalls:

* Exposing Pod without labels yields empty endpoints.
* targetPort must match containerPort or named port.

## Task: Debug in-cluster DNS quickly
Tags: domain:net action:debug format:kubectl
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

* metrics-server outages do not affect DNS; avoid misdiagnosis.
* BusyBox dig may be missing; use nslookup.

# A4. STORAGE (CORE)

## Task: Create PVC with specific StorageClass via YAML
Tags: domain:storage action:create format:yaml
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

* Avoid kubectl create pvc shortcuts; YAML ensures SC/accessModes set.
* Pending indicates missing PV or provisioner.

## Task: Create PVC ReadWriteMany
Tags: domain:storage action:create format:yaml
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

* Many provisioners do not support RWX; expect Pending on unsupported storage.
* Specify storageClassName if default lacks RWX.

## Task: Mount PVC into Pod and verify IO
Tags: domain:storage action:create format:yaml
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

* Pod and PVC must be in same namespace.
* Pending PVC keeps pod Pending; describe pod events.

## Task: Create PV with Retain reclaim policy
Tags: domain:storage action:create format:yaml
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

* hostPath PVs bind to specific node; pods must schedule there.
* Recycle is deprecated; use Retain/Delete only.

## Task: Resize PVC (if StorageClass allows expansion)
Tags: domain:storage action:edit format:kubectl
Command:
```bash
kubectl patch pvc data-claim -p '{"spec":{"resources":{"requests":{"storage":"2Gi"}}}}'
```

Verify:
```bash
kubectl get pvc data-claim
```

Pitfalls:

* StorageClass must set allowVolumeExpansion: true.
* Filesystem expansion may require pod restart depending on driver.

## Task: Identify default StorageClass and provisioner
Tags: domain:storage action:observe format:kubectl
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

* Only one default should be true; unset others if duplicates.
* Provisioner name hints CSI driver to inspect.

## Task: Debug PVC Pending (dynamic provisioning)
Tags: domain:storage action:debug format:kubectl
Fast triage:
```bash
kubectl describe pvc <pvc>
kubectl get events --field-selector involvedObject.name=<pvc> --sort-by=.lastTimestamp
kubectl get pods -A | grep csi
```

Common causes → fix:

* Cause: no default SC → Fix: set annotation on SC or specify storageClassName.
  Verify:
  ```bash
  kubectl patch storageclass <sc> -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
  ```
* Cause: driver pods down → Fix: restart CSI controller/daemonset.

Pitfalls:

* Namespace mismatch between pod and PVC leads to Pending pod not PVC.
* ReadWriteMany request on RWO-only provisioner stays Pending.

## Task: CSI troubleshooting basics
Tags: domain:storage action:debug format:kubectl
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

* VolumeAttachments require RBAC; run as admin.
* Some drivers log to kubelet; check journal if events sparse.

# A5. CLUSTER ADMIN (ABSOLUTE MUST)

## Task: Cordon, drain, uncordon a node safely
Tags: domain:cluster-arch action:edit format:kubectl
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

* PDBs can block drain; use --disable-eviction only if necessary.
* Remove taints before expecting pods to reschedule there.

## Task: Take etcd snapshot and validate
Tags: domain:cluster-arch action:create format:node-shell
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

* Run on control-plane hosting etcd static pod.
* Ensure file permissions allow kube-apiserver to read after restore.

## Task: Restore etcd snapshot and rewire static pod
Tags: domain:cluster-arch action:fix format:node-shell
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

* Remove old member data-dir if conflicts.
* Kubelet auto-restarts static pod after manifest edit.

## Task: Initialize control plane with kubeadm and set kubeconfig
Tags: domain:cluster-arch action:create format:node-shell
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

* Apply CNI manifest after init; nodes stay NotReady until then.
* For root, $HOME may be /root; adjust paths for other users.

## Task: Join worker node with kubeadm
Tags: domain:cluster-arch action:create format:node-shell
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

* Token expires; regenerate with kubeadm token create --print-join-command.
* Ensure required ports open (6443, 10250).

## Task: Upgrade control plane then kubelet/kubectl
Tags: domain:cluster-arch action:edit format:node-shell
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

* Upgrade control plane first; respect version skew (kubelet <= apiserver <= kubeadm).
* Drain nodes before upgrading kubelet when possible.

## Task: Renew control-plane certs
Tags: domain:cluster-arch action:fix format:node-shell
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

* Backup /etc/kubernetes/pki before renewal in real clusters.
* Restart apiserver may briefly disrupt kubectl.

## Task: Generate kubeadm join command (bootstrap token)
Tags: domain:cluster-arch action:create format:node-shell
Command:
```bash
kubeadm token create --print-join-command
```

Verify:
```bash
kubeadm token list
```

Pitfalls:

* Use --ttl 0 for non-expiring token when allowed.
* Hash from /etc/kubernetes/pki/ca.crt via openssl x509 -pubkey ... if missing.

## Task: Configure kubectl for non-root user
Tags: domain:cluster-arch action:fix format:node-shell
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

* Ensure user shell honors $KUBECONFIG if not default path.
* File permissions must allow read by user.

# A6. RBAC & AUTH BASICS

## Task: Create Role and RoleBinding for ServiceAccount
Tags: domain:rbac action:create format:kubectl
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

* Namespace must match Role/RoleBinding.
* Use resourceNames for narrowed permissions if needed.

## Task: Create ClusterRoleBinding to SA
Tags: domain:rbac action:create format:kubectl
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

* ClusterRoleBinding is cluster-scoped; no -n flag.
* Avoid granting cluster-admin unless required.

## Task: Run Pod using a ServiceAccount
Tags: domain:rbac action:create format:yaml
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

* ServiceAccount must exist in same namespace.
* automountServiceAccountToken can be disabled; set explicitly if needed.

## Task: Approve or deny CSRs
Tags: domain:rbac action:edit format:kubectl
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

* CSR signs user or kubelet cert; verify CN/Organization before approval.
* After approval, kubelet may need restart to pick up cert.

# A7. CORE TROUBLESHOOTING

## Task: Troubleshoot CrashLoopBackOff and fix root cause
Tags: domain:troubleshoot action:fix format:kubectl
Fast triage:
```bash
kubectl describe pod <pod> -n <ns> | sed -n '/Events:/,$p'
kubectl logs <pod> -n <ns> --previous
```

Common causes → fix:

* Cause: bad command/probe → Fix: kubectl edit/patch deployment to correct command or probe.
  Verify:
  ```bash
  kubectl rollout status deploy/<deploy> -n <ns>
  ```
* Cause: missing config/secret → Fix: mount correct ConfigMap/Secret.

Pitfalls:

* Use --previous to see failing attempt.
* CrashLoopBackOff delays restart; be patient before verifying.

## Task: Troubleshoot ImagePullBackOff
Tags: domain:troubleshoot action:fix format:kubectl
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

* ImagePolicyWebhook can block latest tag; specify digest/tag.
* Secret must be in same namespace and referenced.

## Task: Troubleshoot Pending Pod (scheduling)
Tags: domain:troubleshoot action:fix format:kubectl
Fast triage:
```bash
kubectl describe pod <pod> -n <ns> | grep -A5 Events
kubectl get nodes --show-labels
```

Common causes → fix:

* Cause: insufficient CPU/mem → Fix: reduce requests or free capacity; verify:
  ```bash
  kubectl get pod <pod> -n <ns>
  ```
* Cause: missing toleration/affinity mismatch → Fix: add toleration or adjust selectors.

Pitfalls:

* PDBs do not affect scheduling; look for taints instead.
* topologySpreadConstraints may also block scheduling.

## Task: Troubleshoot Service no endpoints
Tags: domain:troubleshoot action:fix format:kubectl
Fast triage:
```bash
kubectl get svc <svc> -n <ns> -o wide
kubectl get endpoints <svc> -n <ns>
kubectl get pods -n <ns> -l <key>=<val> -o wide
```

Common causes → fix:

* Cause: selector wrong → Fix: patch selector to match pod labels.
  Verify:
  ```bash
  kubectl get endpoints <svc> -n <ns>
  ```
* Cause: pods NotReady → Fix readiness probe, check describe/events.

Pitfalls:

* Headless Services have ClusterIP=None; expect multiple endpoints.
* targetPort must match containerPort name/number.

## Task: Troubleshoot DNS failures end-to-end
Tags: domain:troubleshoot action:fix format:kubectl
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

* If CoreDNS Service ClusterIP missing, recreate from manifest.
* Ensure resolv.conf inside pods uses cluster DNS IP.

## Task: Troubleshoot Node NotReady
Tags: domain:troubleshoot action:fix format:node-shell
Fast triage:
```bash
kubectl describe node <node>
sudo journalctl -u kubelet -n 50
```

Common causes → fix:

* Cause: kubelet cert expired → Fix: sudo kubeadm cert renew kubelet.conf && sudo systemctl restart kubelet.
  Verify:
  ```bash
  kubectl get nodes
  ```
* Cause: disk/memory pressure → Fix: free space, prune images; restart kubelet.

Pitfalls:

* Network partition can show Ready,SchedulingDisabled; check routes.
* Check container runtime status (systemctl status containerd/crio).

# A8. TOOLING, AUTOSCALING, CUSTOMIZATION

## Task: Install/upgrade Helm chart with custom values
Tags: domain:tooling action:create format:helm
Command:
```bash
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm upgrade --install metrics-server stable/metrics-server -n kube-system \
  --set args[0]='--kubelet-insecure-tls'
```

Verify:
```bash
helm list -A | grep metrics-server
kubectl get deploy -n kube-system metrics-server
```

Pitfalls:

* --install ensures creation if release missing.
* Use -f values.yaml for larger overrides.

## Task: Render Helm template without installing
Tags: domain:tooling action:observe format:helm
Command:
```bash
helm template myrelease stable/metrics-server --namespace kube-system > rendered.yaml
```

Verify:
```bash
head -n 5 rendered.yaml
```

Pitfalls:

* Template uses default values if none supplied; pass -f for overrides.
* Keep apiVersions current after render.

## Task: Apply base + overlay using Kustomize
Tags: domain:tooling action:create format:kustomize
Command:
```bash
kubectl apply -k overlays/prod
```

Verify:
```bash
kubectl kustomize overlays/prod | head -n 5
```

Pitfalls:

* Ensure kustomization.yaml exists in directory.
* overlays inherit from base; confirm namespace patches.

## Task: Horizontal Pod Autoscaler for Deployment
Tags: domain:workloads action:create format:kubectl
Command:
```bash
kubectl autoscale deploy web --min=2 --max=5 --cpu-percent=70
```

Verify:
```bash
kubectl describe hpa web
```

Pitfalls:

* Requires metrics-server; HPA shows Unknown without metrics.
* HPA scales on requests, not limits.

## Task: HPA YAML skeleton (cpu utilization)
Tags: domain:workloads action:create format:yaml
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

* apiVersion autoscaling/v2 required for advanced metrics.
* Ensure Deployment has resource requests set.

## Task: Monitor resource usage quickly
Tags: domain:tooling action:observe format:kubectl
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

* metrics-server must be running; otherwise “metrics not available”.
* Values are near real-time; not historical.

## Task: Create ResourceQuota and LimitRange in namespace
Tags: domain:workloads action:create format:yaml
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

* Quota breaches block creation; check events for “exceeded quota”.
* LimitRange defaults apply only when requests/limits omitted.

## Task: Gateway API quick triage
Tags: domain:net action:debug format:gateway-api
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

* Controller-specific; ensure gatewayclass has active controller.
* Check events on Gateway/HTTPRoute for attachment errors.

## Task: Use kubectl port-forward and proxy for quick validation (OPTIONAL)
Tags: domain:tooling action:debug format:kubectl
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

* port-forward blocks terminal; use background (&) carefully.
* proxy exposes API on localhost only by default.

## Task: kubectl debug with ephemeral container (OPTIONAL)
Tags: domain:troubleshoot action:debug format:kubectl
Command:
```bash
kubectl debug pod/<pod> -it --image=busybox:1.36 --target=<container>
```

Verify:
```bash
kubectl get pod <pod> -o jsonpath='{.spec.ephemeralContainers[*].name}{"\n"}'
```

Pitfalls:

* Requires EnableEphemeralContainers feature (usually on >=1.25).
* Ephemeral containers share namespaces but no ports.

# B1. ADVANCED kubectl SPEED

## Task: Patch Deployment image via strategic merge
Tags: domain:workloads action:patch format:kubectl
Command:
```bash
kubectl patch deploy web -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","image":"nginx:1.26"}]}}}}'
```

Verify:
```bash
kubectl get deploy web -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}'
```

Pitfalls:

* Ensure container name matches existing one.
* Patch preserves other fields; use apply for larger changes.

## Task: Patch Service type to NodePort
Tags: domain:net action:patch format:kubectl
Command:
```bash
kubectl patch svc web -p '{"spec":{"type":"NodePort"}}'
```

Verify:
```bash
kubectl get svc web -o wide
```

Pitfalls:

* Existing ClusterIP preserved; nodePort auto-assigned unless set.
* May disrupt clients expecting ClusterIP only.

## Task: Sort and filter events quickly
Tags: domain:tooling action:observe format:kubectl
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

* Events rotate fast; capture early.
* API priority/flow control may limit large queries.

# B2. ADVANCED WORKLOAD DETAILS

## Task: List init containers and termination reasons
Tags: domain:workloads action:observe format:kubectl
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

* Add {"\n"} to jsonpath to avoid mashed output.
* initContainers array absent if none defined.

## Task: Check resource requests/limits per container
Tags: domain:workloads action:observe format:kubectl
Command:
```bash
kubectl get pod <pod> -o jsonpath='{range .spec.containers[*]}{.name}{" req="}{.resources.requests}{" lim="}{.resources.limits}{"\n"}{end}'
```

Verify:
```bash
kubectl describe pod <pod> | grep -A4 Limits
```

Pitfalls:

* Empty map means defaults/LimitRange may apply.
* Requests gate scheduling; limits control runtime OOMKill (137).

# B3. NETWORKING (EDGE)

## Task: Create headless Service for StatefulSet DNS
Tags: domain:net action:create format:yaml
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

* Without selector, no endpoints generated.
* StatefulSet requires headless svc for stable DNS.

## Task: Default-deny ingress NetworkPolicy and allow specific namespace
Tags: domain:net action:create format:yaml
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

* Policies are additive; ensure allow rules exist for required traffic.
* CNI must support NetworkPolicy (Calico/Cilium/Weave).

## Task: Allow egress DNS after default-deny egress
Tags: domain:net action:create format:yaml
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

* Ensure CoreDNS labels match k8s-app=kube-dns.
* If TCP DNS needed, add TCP port 53.

# B4. STORAGE (EDGE)

## Task: StatefulSet with volumeClaimTemplates
Tags: domain:storage action:create format:yaml
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

* Headless Service required for stable network IDs.
* VolumeClaimTemplates immutable after creation.

## Task: StorageClass allowVolumeExpansion note
Tags: domain:storage action:observe format:kubectl
Command:
```bash
kubectl get sc -o jsonpath='{range .items[*]}{.metadata.name}{" expand="}{.allowVolumeExpansion}{"\n"}{end}'
```

Verify:
```bash
kubectl describe sc <sc>
```

Pitfalls:

* Expansion only works for supported drivers.
* Must patch PVC to larger size; cannot shrink.

# B5. CLUSTER ADMIN (EDGE)

## Task: Locate and edit static pod manifests
Tags: domain:cluster-arch action:edit format:node-shell
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

* Editing manifests can briefly stop API; use care on single-node control plane.
* Ensure YAML indentation remains valid; kubelet loops on invalid files.

## Task: Diagnose container runtime failure (containerd/crio)
Tags: domain:cluster-arch action:debug format:node-shell
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

* crictl requires runtime endpoint; configure /etc/crictl.yaml if missing.
* Runtime failure keeps kubelet NotReady.

## Task: API server health probe locally
Tags: domain:cluster-arch action:observe format:kubectl
Command:
```bash
kubectl get --raw "/readyz"
```

Verify:
```bash
kubectl get componentstatuses
```

Pitfalls:

* componentstatuses deprecated; rely on /readyz and /livez.
* Use admin.conf or correct kubeconfig.

# B6. YAML SKELETONS (MEMORIZE)

## Task: Deployment skeleton with selector matchLabels
Tags: domain:workloads action:create format:yaml
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

* selector must match template labels or apply fails.
* Strategy defaults to RollingUpdate.

## Task: DaemonSet skeleton
Tags: domain:workloads action:create format:yaml
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

* No replicas field; schedules one per Ready node (taints may block).
* Use tolerations to run on masters if tainted.

## Task: Job skeleton
Tags: domain:workloads action:create format:yaml
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

* restartPolicy must be Never/OnFailure for jobs.
* Use ttlSecondsAfterFinished to auto-clean if desired.

## Task: CronJob skeleton (batch/v1)
Tags: domain:workloads action:create format:yaml
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
          - name: cj
            image: busybox:1.36
            command: ["sh","-c","echo run"]
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
          - name: cj
            image: busybox:1.36
            command: ["sh","-c","echo run"]
EOF
```

Verify:
```bash
kubectl get cronjob simple-cj
```

Pitfalls:

* startingDeadlineSeconds prevents missed run backlog.
* concurrencyPolicy (Forbid/Replace) controls overlap.

## Task: Service skeleton (ClusterIP)
Tags: domain:net action:create format:yaml
Manifest:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-svc
spec:
  type: ClusterIP
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
  name: web-svc
spec:
  type: ClusterIP
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
EOF
```

Verify:
```bash
kubectl get svc web-svc -o wide
```

Pitfalls:

* Selector must match pod labels; headless requires clusterIP: None.
* targetPort can be name or number.

## Task: Ingress skeleton (networking.k8s.io/v1)
Tags: domain:net action:create format:yaml
Manifest:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ing-basic
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
```

Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ing-basic
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-svc
            port:
              number: 80
EOF
```

Verify:
```bash
kubectl get ingress web-ing-basic
```

Pitfalls:

* Requires controller; check events for class errors.
* TLS requires secret and tls block.

## Task: NetworkPolicy deny all ingress skeleton
Tags: domain:net action:create format:yaml
Manifest:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
EOF
```

Verify:
```bash
kubectl get netpol deny-all
```

Pitfalls:

* Blocks all ingress unless additional allow policies exist.
* CNI must implement NetworkPolicy.

## Task: RBAC Role + RoleBinding skeleton
Tags: domain:rbac action:create format:yaml
Manifest:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: read-pods
  namespace: dev
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: dev
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: dev
roleRef:
  kind: Role
  name: read-pods
  apiGroup: rbac.authorization.k8s.io
```

Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: read-pods
  namespace: dev
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: dev
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: dev
roleRef:
  kind: Role
  name: read-pods
  apiGroup: rbac.authorization.k8s.io
EOF
```

Verify:
```bash
kubectl auth can-i list pods --as=system:serviceaccount:dev:app-sa -n dev
```

Pitfalls:

* roleRef must match Role kind and apiGroup exactly.
* Binding namespace must match Role namespace.

## Task: PVC skeleton
Tags: domain:storage action:create format:yaml
Manifest:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-basic
spec:
  accessModes: ["ReadWriteOnce"]
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
  name: pvc-basic
spec:
  accessModes: ["ReadWriteOnce"]
  resources:
    requests:
      storage: 1Gi
EOF
```

Verify:
```bash
kubectl get pvc pvc-basic
```

Pitfalls:

* Specify storageClassName if default absent.
* RWX support depends on provisioner.

## Task: HPA skeleton (autoscaling/v2)
Tags: domain:workloads action:create format:yaml
Manifest:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 1
  maxReplicas: 4
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 75
```

Apply:
```bash
kubectl apply -f - <<'EOF'
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 1
  maxReplicas: 4
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 75
EOF
```

Verify:
```bash
kubectl get hpa app-hpa
```

Pitfalls:

* Requires metrics-server; otherwise metrics Unknown.
* Ensure Deployment has cpu requests.

# B7. EXAM HYGIENE / TIME SAVERS

## Task: Speed up kubectl usage
Tags: domain:tooling action:edit format:kubectl
Command:
```bash
alias k=kubectl
complete -F __start_kubectl k
export do='--dry-run=client -o yaml'
```

Verify:
```bash
k get pods --help >/dev/null
```

Pitfalls:

* Aliases persist only in current shell unless added to rc file.
* Autocomplete script depends on bash-completion installed.

## Task: Set default namespace to avoid mistakes
Tags: domain:tooling action:edit format:kubectl
Command:
```bash
kubectl config set-context --current --namespace=dev
```

Verify:
```bash
kubectl config view --minify | grep namespace
```

Pitfalls:

* Changes current context only; switch contexts resets namespace.
* Avoid hardcoding namespace in scripts if switching often.

## Task: Validate manifests quickly (dry-run/explain)
Tags: domain:tooling action:observe format:kubectl
Command:
```bash
kubectl apply -f file.yaml --dry-run=client
kubectl explain pod.spec.containers
```

Verify:
```bash
kubectl explain deployment.spec.selector
```

Pitfalls:

* explain shows server schema; ensure correct apiVersion.
* dry-run=client does not contact cluster; still catches syntax errors.
