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

* --install ensures creation if release missing.
* Use -f values.yaml for larger overrides.

## Task: Render Helm template without installing
Tags: domain:tooling action:observe format:helm
Command:
```bash
helm template myrelease metrics-server/metrics-server --namespace kube-system > rendered.yaml
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
kubectl get --raw "/livez"
```

Pitfalls:

* Use /readyz and /livez; inspect static pod logs via node runtime if API is flaky.
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

# CKA v1.34 COVERAGE ADDITIONS (COMPLETE DO/VERIFY/FIX)

## Task: Create ConfigMap from literal and file (imperative)
Tags: domain:workloads action:create format:kubectl cka:competency:configmaps card:type:do time:2m
Command:
```bash
kubectl create configmap app-cm --from-literal=env=prod --from-file=app.conf=./app.conf \
  --dry-run=client -o yaml > cm.yaml
kubectl apply -f cm.yaml
```

Verify:
```bash
kubectl get cm app-cm -o jsonpath='{.data.env}{" "}{.data.app\.conf}{"\n"}'
```

Pitfalls:

* Escape dots in jsonpath when reading file keys.
* Namespaces must match workload consuming config.

## Task: Consume ConfigMap via envFrom
Tags: domain:workloads action:configure format:yaml cka:competency:configmaps card:type:do time:2m
Manifest:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm-envfrom
spec:
  containers:
  - name: app
    image: busybox:1.36
    envFrom:
    - configMapRef:
        name: app-cm
    command: ["sh","-c","env | grep env= && sleep 3600"]
```

Verify:
```bash
kubectl exec cm-envfrom -- printenv env
```

Pitfalls:

* Pod fails to start if referenced ConfigMap missing.
* envFrom loads all keys; use env with valueFrom for single key.

## Task: Consume ConfigMap as single env var
Tags: domain:workloads action:configure format:yaml cka:competency:configmaps card:type:verify time:30s
Command:
```bash
kubectl set env deploy/web APP_ENV_FROM_CM_CONFIG= --from=configmap/app-cm
kubectl get deploy web -o jsonpath='{.spec.template.spec.containers[0].env[?(@.name=="env")].valueFrom.configMapKeyRef.name}{"\n"}'
```

Verify:
```bash
kubectl exec deploy/web -- printenv env
```

Pitfalls:

* Key mismatch shows empty env var; confirm key names in ConfigMap data.

## Task: Mount ConfigMap as volume and verify file path
Tags: domain:workloads action:configure format:yaml cka:competency:configmaps card:type:verify time:2m
Command:
```bash
kubectl patch deploy web -p '{
  "spec":{"template":{"spec":{"volumes":[{"name":"cm-vol","configMap":{"name":"app-cm"}}],
  "containers":[{"name":"web","volumeMounts":[{"name":"cm-vol","mountPath":"/etc/app"}]}]}}}}'
kubectl rollout status deploy/web
```

Verify:
```bash
kubectl exec deploy/web -- ls -l /etc/app && kubectl exec deploy/web -- cat /etc/app/app.conf
```

Pitfalls:

* SubPath mounts need restart on update; whole volume propagates automatically.

## Task: Update ConfigMap and trigger rollout
Tags: domain:workloads action:edit format:kubectl cka:competency:configmaps card:type:do time:2m
Command:
```bash
kubectl create configmap app-cm --from-literal=env=staging -o yaml --dry-run=client | kubectl apply -f -
kubectl annotate deploy/web checksum/config="$(kubectl get cm app-cm -o json | sha256sum | awk '{print $1}')" --overwrite
kubectl rollout restart deploy/web
```

Verify:
```bash
kubectl get pods -l app=web -o jsonpath='{.items[*].metadata.annotations.checksum/config}{"\n"}'
kubectl logs deploy/web --tail=5
```

Pitfalls:

* Without rollout restart, pods may keep stale config if using envFrom.
* Rolling update needs available replicas; ensure readiness probes pass.

## Task: Debug app not picking up ConfigMap values
Tags: domain:workloads action:debug format:kubectl cka:competency:configmaps card:type:fix time:2m
Checklist:
```bash
kubectl get pod -n appns app-pod -o jsonpath='{.spec.volumes[*].configMap.name}{"\n"}'
kubectl exec app-pod -n appns -- ls /etc/app
kubectl rollout history deploy/app -n appns
kubectl get events -n appns --sort-by=.lastTimestamp | tail
```

Fix:

* If ConfigMap wrong namespace → recreate in appns.
* If env var empty → correct key name/valueFrom.
* If pods old → rollout restart deploy/app.

Pitfalls:

* SubPath mounts require pod recreate after ConfigMap update.

## Task: Create Secret from literal and file (dry-run YAML)
Tags: domain:workloads action:create format:kubectl cka:competency:secrets card:type:do time:2m
Command:
```bash
kubectl create secret generic app-secret --from-literal=password=s3cr3t \
  --from-file=token=./token.txt --dry-run=client -o yaml > secret.yaml
kubectl apply -f secret.yaml
```

Verify:
```bash
kubectl get secret app-secret -o jsonpath='{.data.password}' | base64 -d
```

Pitfalls:

* Secrets stored base64 encoded, not encrypted by default.
* Keep files out of version control.

## Task: Consume Secret via envFrom
Tags: domain:workloads action:configure format:yaml cka:competency:secrets card:type:do time:30s
Command:
```bash
kubectl set env deploy/web --from=secret/app-secret
kubectl rollout status deploy/web
```

Verify:
```bash
kubectl exec deploy/web -- printenv password
```

Pitfalls:

* Missing secret causes deployment to fail scheduling.

## Task: Use Secret key as single env var
Tags: domain:workloads action:configure format:yaml cka:competency:secrets card:type:verify time:30s
Manifest:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-key-env
spec:
  containers:
  - name: app
    image: busybox:1.36
    env:
    - name: APP_TOKEN
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: token
    command: ["sh","-c","echo $APP_TOKEN && sleep 3600"]
```

Verify:
```bash
kubectl exec secret-key-env -- printenv APP_TOKEN
```

Pitfalls:

* Wrong key name yields EmptyDir-style missing var; check describe pod warnings.

## Task: Mount Secret as volume and note permissions
Tags: domain:workloads action:configure format:yaml cka:competency:secrets card:type:verify time:30s
Command:
```bash
kubectl patch deploy web -p '{
  "spec":{"template":{"spec":{"volumes":[{"name":"secret-vol","secret":{"secretName":"app-secret"}}],
  "containers":[{"name":"web","volumeMounts":[{"name":"secret-vol","mountPath":"/etc/secret","readOnly":true}]}]}}}}'
kubectl rollout status deploy/web
```

Verify:
```bash
kubectl exec deploy/web -- ls -l /etc/secret
```

Pitfalls:

* Default file mode 0400; set defaultMode if app needs different permissions.

## Task: Create docker-registry Secret and attach imagePullSecrets
Tags: domain:workloads action:create format:kubectl cka:competency:secrets card:type:do lab:required time:2m
Command:
```bash
kubectl create secret docker-registry regcred --docker-server=index.docker.io \
  --docker-username=user --docker-password=pass --docker-email=user@example.com
kubectl patch serviceaccount default -p '{"imagePullSecrets":[{"name":"regcred"}]}'
```

Verify:
```bash
kubectl run pullcheck --image=private/repo:tag --restart=Never --dry-run=client -o yaml
```

Pitfalls:

* Scope: namespace-specific; create in each ns or set imagePullSecrets per pod.

## Task: Debug ImagePullBackOff due to pull secret
Tags: domain:workloads action:fix format:kubectl cka:competency:secrets card:type:fix time:2m
Checklist:
```bash
kubectl describe pod failing-pod
kubectl get secret regcred -o yaml
kubectl get sa default -o yaml
```

Fix:

* Create correct docker-registry secret; patch pod spec imagePullSecrets.
* Wrong server URL → recreate secret with proper --docker-server.

Pitfalls:

* Secret name typo common; use imagePullSecrets list order does not matter.

## Task: Configure topologySpreadConstraints and fix Pending skew
Tags: domain:workloads action:configure format:yaml cka:competency:scheduling card:type:do time:2m
Command:
```bash
kubectl patch deploy web -p '{
  "spec":{"template":{"spec":{"topologySpreadConstraints":[{
    "maxSkew":1,"topologyKey":"topology.kubernetes.io/zone","whenUnsatisfiable":"DoNotSchedule",
    "labelSelector":{"matchLabels":{"app":"web"}},"matchLabelKeys":["app"]}]}}}}'
```

Verify:
```bash
kubectl get pods -l app=web -o wide --sort-by=.spec.nodeName
kubectl describe pod <pending-pod> | grep -A3 Unschedulable
```

Pitfalls:

* Ensure nodes labeled with topologyKey; otherwise pods pend.
* whenUnsatisfiable=ScheduleAnyway ignores skew but may concentrate pods.

## Task: Fix scheduling failure by adjusting resource requests/limits
Tags: domain:workloads action:fix format:kubectl cka:competency:scheduling card:type:fix time:2m
Command:
```bash
kubectl describe pod heavy | grep -A3 -e "Insufficient"
kubectl patch deploy heavy -p '{
  "spec":{"template":{"spec":{"containers":[{"name":"app","resources":{
    "requests":{"cpu":"100m","memory":"128Mi"},"limits":{"cpu":"500m","memory":"256Mi"}}}]}}}}'
kubectl rollout status deploy/heavy
```

Verify:
```bash
kubectl get pods -l app=heavy -o wide
```

Pitfalls:

* Requests drive scheduling; limits do not help placement.
* Node allocatable may still be too small; consider scale-out.

## Task: Confirm LimitRange defaulting applies
Tags: domain:workloads action:verify format:kubectl cka:competency:scheduling card:type:verify time:30s
Command:
```bash
kubectl create ns limits-demo
kubectl get limitrange -n limits-demo -o wide
kubectl run lr-pod --image=busybox:1.36 -n limits-demo --restart=Never -- /bin/sleep 3600
kubectl describe pod lr-pod -n limits-demo | grep -A2 "Limits:" -m1
```

Pitfalls:

* Default LimitRange only applies when requests/limits omitted.
* Delete namespace after lab to clean up.

## Task: Observe pod preemption due to PriorityClass
Tags: domain:workloads action:verify format:kubectl cka:competency:scheduling card:type:verify time:2m
Command:
```bash
kubectl get priorityclass
kubectl describe pod preempted-pod | grep -i preempt
kubectl get events --field-selector reason=Preempting --sort-by=.lastTimestamp | tail
```

Pitfalls:

* Preemption happens only if higher-priority pod fits after evictions.
* System priorities start with system-; avoid overriding.

## Task: StatefulSet operations (scale/update/stable DNS)
Tags: domain:workloads action:configure format:kubectl cka:competency:statefulsets card:type:do time:2m
Command:
```bash
kubectl scale sts web-db --replicas=3
kubectl patch sts web-db -p '{"spec":{"updateStrategy":{"type":"RollingUpdate"}}}'
kubectl get pod -l app=web-db -o custom-columns=NAME:.metadata.name,PODIP:.status.podIP,HOST:.spec.subdomain
```

Verify:
```bash
kubectl exec web-db-0 -- nslookup web-db-0.web-db
```

Pitfalls:

* PVCs stick to ordinal; deleting pod keeps PVC.
* Parallel update strategy available only with allowance; default is OrderedReady.

## Task: DaemonSet update and rollback
Tags: domain:workloads action:fix format:kubectl cka:competency:daemonsets card:type:fix time:30s
Command:
```bash
kubectl rollout status ds/node-agent -n kube-system
kubectl set image ds/node-agent agent=repo/agent:v2 -n kube-system
kubectl rollout undo ds/node-agent -n kube-system
```

Verify:
```bash
kubectl get pods -n kube-system -l app=node-agent -o wide
```

Pitfalls:

* DaemonSets can use surge during update only with maxUnavailable control.
* Rollback requires revision history enabled.

## Task: Job and CronJob control knobs
Tags: domain:workloads action:configure format:yaml cka:competency:jobs card:type:verify time:2m
Command:
```bash
kubectl patch cronjob simple-cj -p '{"spec":{"concurrencyPolicy":"Forbid","startingDeadlineSeconds":120}}'
kubectl patch job simple-job -p '{"spec":{"ttlSecondsAfterFinished":60}}'
```

Verify:
```bash
kubectl get cronjob simple-cj -o jsonpath='{.spec.concurrencyPolicy}{" "}{.spec.startingDeadlineSeconds}{"\n"}'
kubectl get job simple-job -o jsonpath='{.spec.ttlSecondsAfterFinished}{"\n"}'
```

Pitfalls:

* Missed CronJobs beyond startingDeadlineSeconds are skipped.
* TTL controller must be enabled (default on new clusters).

## Task: Highly-available control-plane endpoint in kubeadm config
Tags: domain:cluster-arch action:observe format:node-shell cka:competency:ha-control-plane card:type:verify lab:required time:2m
Command:
```bash
sudo cat /etc/kubernetes/kubeadm-config.yaml | grep controlPlaneEndpoint -A2
```

Verify:
```bash
grep controlPlaneEndpoint /etc/kubernetes/kubeadm-config.yaml
```

Pitfalls:

* Missing endpoint leads to API servers advertising node IPs.

## Task: Join additional control-plane node
Tags: domain:cluster-arch action:create format:node-shell cka:competency:ha-control-plane card:type:do lab:required time:2m
Command:
```bash
kubeadm token create --print-join-command   # run on existing CP
kubeadm init phase upload-certs --upload-certs
# on new node:
sudo <join-command> --control-plane --certificate-key <key>
```

Verify:
```bash
kubectl get nodes -o wide | grep control-plane
```

Pitfalls:

* Cert key valid 2 hours; re-upload if expired.
* Ensure load balancer points to all API servers.

## Task: Distinguish stacked-etcd vs external-etcd
Tags: domain:cluster-arch action:observe format:node-shell cka:competency:ha-control-plane card:type:verify lab:required time:30s
Command:
```bash
ls /etc/kubernetes/manifests | grep etcd && sudo crictl ps | grep etcd
grep "\--etcd-servers" /etc/kubernetes/manifests/kube-apiserver.yaml
```

Pitfalls:

* External etcd: no etcd static pod on node; apiserver --etcd-servers points to LB.

## Task: API server behind LB health check
Tags: domain:cluster-arch action:verify format:kubectl cka:competency:ha-control-plane card:type:verify lab:required time:30s
Command:
```bash
kubectl get --raw /readyz
curl -k https://<controlplane-lb>:6443/readyz
```

Pitfalls:

* LB health must probe /readyz; misconfig causes flapping.

## Task: Troubleshoot NotReady control-plane node in HA
Tags: domain:cluster-arch action:fix format:node-shell cka:competency:ha-control-plane card:type:fix lab:required time:2m
Command:
```bash
kubectl describe node <cp-node>
sudo journalctl -u kubelet -n 50
sudo crictl logs $(sudo crictl ps --name kube-apiserver -q)
```

Pitfalls:

* etcd quorum may still be healthy; check etcd endpoints status.
* Kubelet cert rotation failures common; renew kubelet.conf.

## Task: Inspect scheduler/controller-manager static pods
Tags: domain:cluster-arch action:observe format:node-shell cka:competency:ha-control-plane card:type:verify lab:required time:30s
Command:
```bash
sudo ls /etc/kubernetes/manifests | grep kube-scheduler
sudo cat /etc/kubernetes/manifests/kube-controller-manager.yaml | head
```

Pitfalls:

* Edits require kubelet restart of static pods; be careful on single surviving node.

## Task: Rotate/renew control-plane certs impact
Tags: domain:cluster-arch action:observe format:node-shell cka:competency:ha-control-plane card:type:verify lab:required time:2m
Command:
```bash
sudo kubeadm cert check-expiration
sudo kubeadm cert renew all
sudo systemctl restart kubelet
```

Pitfalls:

* Renew on all control-plane nodes; restart components to pick new certs.
* Backup /etc/kubernetes/pki before renewal.

## Task: Recover after losing one control-plane node
Tags: domain:cluster-arch action:fix format:node-shell cka:competency:ha-control-plane card:type:fix lab:required time:2m
Command:
```bash
kubectl get nodes
kubectl get pods -n kube-system -o wide | grep -E 'apiserver|etcd'
```

Fix:

* Traffic continues via LB if quorum intact; drain/cordon failed node.
* Recreate node with kubeadm join --control-plane using latest cert key.

Pitfalls:

* etcd with odd members tolerates one loss; two losses break quorum.

## Task: Discover CRDs and new API resources
Tags: domain:cluster-arch action:observe format:kubectl cka:competency:crds-operators card:type:verify time:30s
Command:
```bash
kubectl get crd | head
kubectl api-resources | grep -i <keyword>
```

Pitfalls:

* api-resources hides disabled groups unless server-side enabled.

## Task: List CR instances across namespaces
Tags: domain:cluster-arch action:observe format:kubectl cka:competency:crds-operators card:type:verify time:30s
Command:
```bash
kubectl get <crd-plural> -A
```

Pitfalls:

* Some CRDs are namespace-scoped; others cluster-scoped.

## Task: Explain unknown CRD quickly
Tags: domain:cluster-arch action:observe format:kubectl cka:competency:crds-operators card:type:verify time:30s
Command:
```bash
kubectl explain <Kind> --recursive | less
```

Pitfalls:

* CRDs may not include full schema; rely on examples in operator docs.

## Task: Install operator via Helm
Tags: domain:cluster-arch action:create format:helm cka:competency:crds-operators card:type:do lab:required time:2m
Command:
```bash
helm repo add example-operator https://example.com/charts
helm upgrade --install example-operator example-operator/chart -n operators --create-namespace
```

Verify:
```bash
helm status example-operator -n operators
kubectl get crd | grep example
```

Pitfalls:

* Wait for CRDs to register before applying CRs.

## Task: Install operator via Kustomize
Tags: domain:cluster-arch action:create format:kustomize cka:competency:crds-operators card:type:do lab:required time:2m
Command:
```bash
kubectl apply -k config/default
```

Verify:
```bash
kubectl get pods -n operators
```

Pitfalls:

* Ensure kustomization.yaml sets namespace; overlays override images easily.

## Task: Verify operator installation health
Tags: domain:cluster-arch action:verify format:kubectl cka:competency:crds-operators card:type:verify time:30s
Command:
```bash
kubectl get crd | grep example
kubectl get deploy -n operators -l app=example-operator
kubectl get events -n operators | tail
```

Pitfalls:

* Admission webhooks may block CR creation if operator pods not ready.

## Task: Create minimal CustomResource from docs
Tags: domain:cluster-arch action:create format:yaml cka:competency:crds-operators card:type:do time:2m
Command:
```bash
kubectl apply -f - <<'EOF'
apiVersion: example.com/v1
kind: Example
metadata:
  name: example-sample
spec:
  size: 1
EOF
```

Verify:
```bash
kubectl get example example-sample -o yaml
```

Pitfalls:

* apiVersion must match installed CRD; check kubectl api-resources.

## Task: Debug CustomResource stuck in progressing state
Tags: domain:cluster-arch action:fix format:kubectl cka:competency:crds-operators card:type:fix time:2m
Command:
```bash
kubectl describe example example-sample
kubectl get events -A | grep example
kubectl logs -n operators deploy/example-operator
```

Pitfalls:

* Invalid spec often shown in status.conditions; fix and reapply.

## Task: Uninstall operator cleanly
Tags: domain:cluster-arch action:delete format:kubectl cka:competency:crds-operators card:type:fix lab:required time:2m
Command:
```bash
kubectl delete examples --all
helm uninstall example-operator -n operators || kubectl delete -k config/default
kubectl delete crd examples.example.com
```

Pitfalls:

* Deleting CRDs before CRs leaves orphan finalizers; remove finalizers if stuck.

## Task: Prepare infrastructure (ports, swap, sysctls)
Tags: domain:cluster-arch action:configure format:node-shell cka:competency:infra-prep card:type:do lab:required time:2m
Command:
```bash
sudo swapoff -a && sudo sed -i '/ swap / s/^/#/' /etc/fstab
sudo modprobe br_netfilter && echo 'net.bridge.bridge-nf-call-iptables=1' | sudo tee /etc/sysctl.d/k8s.conf
sudo sysctl --system
```

Verify:
```bash
sudo ss -ltn | grep -E '2379|2380|10250|10257|10259'
```

Pitfalls:

* Forgetting to persist sysctl causes CNI issues after reboot.

## Task: Ensure container runtime running and crictl configured
Tags: domain:cluster-arch action:verify format:node-shell cka:competency:infra-prep card:type:verify lab:required time:30s
Command:
```bash
sudo systemctl status containerd
sudo cat /etc/crictl.yaml
sudo crictl ps -a
```

Pitfalls:

* Wrong runtimeEndpoint breaks kubectl logs/exec; set to unix:///run/containerd/containerd.sock.

## Task: Extension interfaces quick checks (CNI/CSI/CRI)
Tags: domain:cluster-arch action:debug format:kubectl cka:competency:extensions card:type:verify time:2m
Command:
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl get pods -n kube-system -l k8s-app=kube-proxy
kubectl get csidrivers,csinodes,volumeattachments.storage.k8s.io
sudo systemctl status containerd || sudo systemctl status crio
```

Pitfalls:

* CNI manifests usually in /etc/cni/net.d; broken CNI → pods Pending or No route to host.

## Task: Helm/Kustomize debugging for cluster components
Tags: domain:tooling action:observe format:helm cka:competency:cluster-maintenance card:type:verify time:2m
Command:
```bash
helm status metrics-server -n kube-system
helm get values metrics-server -n kube-system
helm get manifest metrics-server -n kube-system | head
kubectl apply -k overlays/prod
kubectl kustomize overlays/prod | head
```

Pitfalls:

* Keep helm release namespace consistent; helm status fails otherwise.

## Task: Connectivity triage between pods
Tags: domain:net action:debug format:kubectl cka:competency:services-networking card:type:fix time:2m
Command:
```bash
kubectl run netshoot --rm -it --image=nicolaka/netshoot -- /bin/sh
# inside:
ping -c1 <pod-ip> || echo "routing?"
nslookup web-svc || echo "dns?"
curl -v web-svc.default.svc.cluster.local:80 || echo "svc?"
```

Pitfalls:

* Distinguish DNS failure (NXDOMAIN) vs timeout (routing/NP).

## Task: NetworkPolicy default deny ingress+egress then allow
Tags: domain:net action:create format:yaml cka:competency:services-networking card:type:do lab:required time:2m
Command:
```bash
kubectl apply -f - <<'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: deny-all}
spec:
  podSelector: {}
  policyTypes: [Ingress, Egress]
EOF
kubectl apply -f - <<'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: {name: allow-app}
spec:
  podSelector: {matchLabels: {app: web}}
  ingress:
  - from:
    - namespaceSelector: {matchLabels: {name: client}}
    - podSelector: {matchLabels: {role: client}}
    ports: [{port: 80}]
  egress:
  - to:
    - namespaceSelector: {matchLabels: {kubernetes.io/metadata.name: kube-system}}
    ports: [{port: 53, protocol: UDP}, {port: 53, protocol: TCP}]
EOF
```

Verify:
```bash
kubectl describe netpol deny-all
```

Pitfalls:

* Ensure CNI supports egress; otherwise policies silently ignored.

## Task: Debug NetworkPolicy blocking traffic
Tags: domain:net action:fix format:kubectl cka:competency:services-networking card:type:fix time:2m
Command:
```bash
kubectl run tester --rm -it --image=busybox:1.36 --restart=Never -- /bin/sh
# attempt curl/ping
kubectl get netpol -A
kubectl describe netpol allow-app
```

Pitfalls:

* policies are additive; missing egress rules block DNS.
* Some CNIs need podSelector+namespaceSelector combo to match pods.

## Task: Service named port and readiness-related empty endpoints
Tags: domain:net action:verify format:kubectl cka:competency:services-networking card:type:verify time:2m
Command:
```bash
kubectl patch svc web-svc -p '{"spec":{"ports":[{"port":80,"targetPort":"http","name":"web"}]}}'
kubectl get endpoints web-svc -o yaml
kubectl describe pod <pod> | grep -A2 Readiness
```

Pitfalls:

* Pods not Ready remove endpoints; check readinessProbe events.

## Task: Headless Service DNS check
Tags: domain:net action:verify format:kubectl cka:competency:services-networking card:type:verify time:30s
Command:
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Service
metadata: {name: web-headless}
spec:
  clusterIP: None
  selector: {app: web}
  ports: [{port: 80, targetPort: 80}]
EOF
kubectl run dnscheck --rm -it --image=busybox:1.36 --restart=Never -- nslookup web-headless.default.svc.cluster.local
```

Pitfalls:

* Each pod gets A/AAAA record; SRV records include port.

## Task: Identify ingress controller and class
Tags: domain:net action:observe format:kubectl cka:competency:services-networking card:type:verify time:30s
Command:
```bash
kubectl get ingressclass
kubectl get pods -A -l app.kubernetes.io/name=ingress-nginx
kubectl get events -A | grep Ingress
```

Pitfalls:

* Ingress without matching ingressClassName will not be processed.

## Task: Debug Ingress created but no routing
Tags: domain:net action:fix format:kubectl cka:competency:services-networking card:type:fix time:2m
Command:
```bash
kubectl describe ingress web-ing-basic
kubectl get events -n ingress-nginx | tail
kubectl logs -n ingress-nginx deploy/ingress-nginx-controller --tail=20
```

Pitfalls:

* Service selector mismatch leads to empty endpoints.
* Wrong ingressClassName leaves resource Unmanaged.

## Task: Gateway API status checks
Tags: domain:net action:verify format:kubectl cka:competency:services-networking card:type:verify time:2m
Command:
```bash
kubectl get httproute -o jsonpath='{.items[*].status.parents[*].conditions[*].type}{"\n"}'
kubectl describe httproute web-route | grep -A2 Accepted
```

Pitfalls:

* status.parents shows Accepted/Attached/ResolvedRefs; use to spot class mismatch.

## Task: Fix Gateway parentRef or backendRef mismatch
Tags: domain:net action:fix format:kubectl cka:competency:services-networking card:type:fix lab:required time:2m
Command:
```bash
kubectl get gateway -A
kubectl patch httproute web-route -p '{"spec":{"parentRefs":[{"name":"web-gw","namespace":"gateway-ns"}]}}'
kubectl patch httproute web-route -p '{"spec":{"rules":[{"backendRefs":[{"name":"web-svc","port":80}]}]}}'
```

Verify:
```bash
kubectl get httproute web-route -o jsonpath='{.status.parents[*].conditions[*].status}{"\n"}'
```

Pitfalls:

* Wrong namespace on parentRef blocks attachment.

## Task: CoreDNS deep checks (service IP, resolv.conf, ConfigMap)
Tags: domain:net action:verify format:kubectl cka:competency:services-networking card:type:verify time:2m
Command:
```bash
kubectl get svc -n kube-system kube-dns -o wide
kubectl exec deploy/web -- cat /etc/resolv.conf
kubectl get cm coredns -n kube-system -o yaml | head
```

Pitfalls:

* dnsPolicy: ClusterFirstWithHostNet required for hostNetwork pods.
* Wrong dnsConfig may break search paths.

## Task: Troubleshoot CoreDNS when DNS wrong
Tags: domain:net action:fix format:kubectl cka:competency:services-networking card:type:fix time:2m
Command:
```bash
kubectl get endpoints -n kube-system kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns --tail=20
kubectl exec deploy/web -- nslookup kubernetes.default
```

Pitfalls:

* If endpoints empty, CoreDNS pods not Ready; restart DS/Deployment.

## Task: StorageClass parameters and default class
Tags: domain:storage action:verify format:kubectl cka:competency:storage card:type:verify time:30s
Command:
```bash
kubectl get sc -o yaml | grep -A3 parameters
kubectl patch storageclass standard -p '{"metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

Pitfalls:

* Only one default class; remove annotation from others.

## Task: Reclaim policy behavior (Delete vs Retain)
Tags: domain:storage action:fix format:kubectl cka:competency:storage card:type:fix time:2m
Command:
```bash
kubectl get pv
kubectl patch pv pv-retain -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
kubectl delete pvc claim-retain
```

Verify:
```bash
kubectl get pv pv-retain -o jsonpath='{.status.phase}{" "}{.spec.claimRef}{"\n"}'
```

Pitfalls:

* Retain leaves PV Released; clear claimRef to recycle manually.

## Task: PV/PVC binding selectors
Tags: domain:storage action:verify format:yaml cka:competency:storage card:type:do time:2m
Command:
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: PersistentVolume
metadata: {name: pv-sel}
spec:
  capacity: {storage: 1Gi}
  accessModes: [ReadWriteOnce]
  storageClassName: manual
  hostPath: {path: /tmp/pv-sel}
  claimRef:
    name: sel-claim
    namespace: default
EOF
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata: {name: sel-claim}
spec:
  storageClassName: manual
  volumeName: pv-sel
  accessModes: [ReadWriteOnce]
  resources: {requests: {storage: 1Gi}}
EOF
```

Verify:
```bash
kubectl get pvc sel-claim -o wide
```

Pitfalls:

* storageClassName must match; volumeName hard binds to specific PV.

## Task: Volume type quick tasks (emptyDir, hostPath, config/secret)
Tags: domain:storage action:create format:yaml cka:competency:storage card:type:do time:2m
Command:
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata: {name: vol-mix}
spec:
  containers:
  - name: app
    image: busybox:1.36
    command: ["sh","-c","df -h /cache && ls /host && ls /cfg && sleep 3600"]
    volumeMounts:
    - {name: cache, mountPath: /cache}
    - {name: host, mountPath: /host}
    - {name: cfg, mountPath: /cfg}
  volumes:
  - {name: cache, emptyDir: {}}
  - {name: host, hostPath: {path: /tmp}}
  - {name: cfg, configMap: {name: app-cm}}
EOF
```

Pitfalls:

* hostPath tied to node; avoid for HA workloads.

## Task: Troubleshoot MountVolume/attach errors
Tags: domain:storage action:fix format:kubectl cka:competency:storage card:type:fix time:2m
Command:
```bash
kubectl describe pod failing-pod | grep -A4 -e "MountVolume" -e "AttachVolume"
kubectl get events --field-selector involvedObject.name=failing-pod --sort-by=.lastTimestamp
kubectl logs -n kube-system -l app=csi-controller --tail=20
```

Pitfalls:

* Node plugin daemonset issues often cause attach failures; restart CSI node pod.

## Task: Troubleshoot kube-apiserver static pod logs (API flaky)
Tags: domain:troubleshoot action:debug format:node-shell cka:competency:cluster-components card:type:fix lab:required time:2m
Command:
```bash
sudo crictl ps | grep kube-apiserver
sudo crictl logs $(sudo crictl ps --name kube-apiserver -q) | tail
```

Pitfalls:

* If container runtime down, check /var/log/pods/<uid>/kube-apiserver/*.log.

## Task: Debug scheduler/controller-manager crashloop
Tags: domain:troubleshoot action:debug format:node-shell cka:competency:cluster-components card:type:fix lab:required time:2m
Command:
```bash
sudo crictl logs $(sudo crictl ps --name kube-scheduler -q) | tail
sudo crictl logs $(sudo crictl ps --name kube-controller-manager -q) | tail
kubectl get events -n kube-system | grep -E 'scheduler|controller'
```

Pitfalls:

* Mis-signed kubeconfig causes TLS errors; regen with kubeadm init phase control-plane.

## Task: etcd health basics
Tags: domain:troubleshoot action:observe format:node-shell cka:competency:cluster-components card:type:verify lab:required time:2m
Command:
```bash
sudo ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/peer.crt \
  --key=/etc/kubernetes/pki/etcd/peer.key endpoint health
```

Pitfalls:

* Use peer certs for member-to-member; client certs for apiserver access.

## Task: kubelet troubleshooting (certs, runtime endpoint)
Tags: domain:troubleshoot action:fix format:node-shell cka:competency:cluster-components card:type:fix lab:required time:2m
Command:
```bash
sudo journalctl -u kubelet -n 50
sudo kubeadm cert renew kubelet.conf
sudo systemctl restart kubelet
```

Pitfalls:

* Check /var/lib/kubelet/config.yaml for containerRuntimeEndpoint.

## Task: kube-proxy issues affecting Service traffic
Tags: domain:troubleshoot action:debug format:kubectl cka:competency:cluster-components card:type:fix time:2m
Command:
```bash
kubectl get ds kube-proxy -n kube-system -o wide
kubectl logs -n kube-system -l k8s-app=kube-proxy --tail=20
kubectl get cm -n kube-system kube-proxy -o yaml | head
```

Pitfalls:

* Mode iptables/ipvs mismatch with kernel modules leads to no service VIPs.

## Task: Container logs for CoreDNS issues
Tags: domain:troubleshoot action:debug format:kubectl cka:competency:cluster-components card:type:verify time:30s
Command:
```bash
kubectl logs -n kube-system -l k8s-app=kube-dns --previous --tail=30
```

Pitfalls:

* CrashLoopBackOff indicates configmap parse errors.

## Task: Multi-container logs and previous attempts
Tags: domain:troubleshoot action:observe format:kubectl cka:competency:observability card:type:verify time:30s
Command:
```bash
kubectl logs pod/mc-probe -c app --tail=20
kubectl logs pod/mc-probe -c sidecar --previous --tail=20
kubectl get pod mc-probe -o jsonpath='{.status.containerStatuses[*].restartCount}{"\n"}'
```

Pitfalls:

* --previous only works if container restarted.

## Task: Use logs --since/--tail and handle empty logs
Tags: domain:troubleshoot action:observe format:kubectl cka:competency:observability card:type:verify time:30s
Command:
```bash
kubectl logs deploy/web --since=5m --tail=50
kubectl logs pod/sidecar-only -c sidecar || echo "check container name/termination state"
```

Pitfalls:

* Terminated containers without --previous show nothing.

## Task: Monitor metrics availability and debug missing metrics
Tags: domain:troubleshoot action:debug format:kubectl cka:competency:observability card:type:fix time:2m
Command:
```bash
kubectl top nodes || kubectl get pods -n kube-system | grep metrics-server
kubectl logs -n kube-system deploy/metrics-server --tail=20
```

Pitfalls:

* Metrics-server needs correct --kubelet-insecure-tls or CA; TLS errors block metrics.

## Task: Interpret OOMKilled vs CPU throttling
Tags: domain:troubleshoot action:observe format:kubectl cka:competency:observability card:type:verify time:2m
Command:
```bash
kubectl describe pod app | grep -A3 -E 'OOMKilled|ExitCode: 137'
kubectl top pod app
kubectl logs app --tail=5
```

Pitfalls:

* OOMKilled exit 137; fix by raising memory request/limit.
* CPU throttling shows high latency but pod stays Running.

## Task: Container output streams awareness
Tags: domain:troubleshoot action:observe format:kubectl cka:competency:observability card:type:do time:30s
Command:
```bash
kubectl logs pod/mc-probe -c app --timestamps
kubectl logs pod/mc-probe -c app --previous
```

Pitfalls:

* Init containers have separate logs; use -c <init-name>.
