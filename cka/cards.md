# A1. kubectl CORE MECHANICS (HIGHEST ROI)

## Create a single Pod named nginx using kubectl

kubectl run nginx --image=nginx --restart=Never

## Create a Deployment named web with image nginx

kubectl create deploy web --image=nginx

## Expose Deployment web on port 80 as ClusterIP

kubectl expose deploy web --port=80 --target-port=80 --type=ClusterIP

## Generate Deployment YAML without creating it

kubectl create deploy web --image=nginx --dry-run=client -o yaml

## Apply a manifest file

kubectl apply -f file.yaml

## Force replace a resource from file

kubectl replace --force -f file.yaml

## Immediately delete a Pod

kubectl delete pod nginx --grace-period=0 --force

## Edit a live resource

kubectl edit deploy web

## Export resource YAML to file

kubectl get pod nginx -o yaml > pod.yaml

## Get Pods in a namespace

kubectl get pods -n kube-system

## Get Pods across all namespaces

kubectl get pods --all-namespaces

## Select resources by label

kubectl get pods -l app=web

## Show wide output for Nodes

kubectl get nodes -o wide

## Extract Pod name using jsonpath

kubectl get pods -o jsonpath='{.items[0].metadata.name}'

## Filter Pods by node name

kubectl get pods --field-selector spec.nodeName=node1

## Scale a Deployment to 3 replicas

kubectl scale deploy web --replicas=3

## Watch rollout status of a Deployment

kubectl rollout status deploy web

## Roll back a Deployment

kubectl rollout undo deploy web

## Describe a Pod

kubectl describe pod nginx

## Show logs of a Pod

kubectl logs nginx

## Show logs from previous container instance

kubectl logs -p nginx

## Execute a shell in a Pod

kubectl exec -it nginx -- sh

## Check RBAC permission for an action

kubectl auth can-i create pods

## Impersonate a user for auth check

kubectl auth can-i delete pods --as=user1

## Impersonate user and group

kubectl auth can-i get pods --as=user1 --as-group=devs
# A2. WORKLOADS (PODS, DEPLOYMENTS, JOBS)

## Create a single-container Pod with kubectl

```bash
kubectl run nginx --image=nginx
```

## Create a multi-container Pod (2 containers)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mc
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
  - name: sidecar
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
```

## Set Pod restart policy Never

```bash
kubectl run test --image=busybox --restart=Never -- echo ok
```

## Set Pod restart policy OnFailure

```bash
kubectl run jobpod --image=busybox --restart=OnFailure -- false
```

## Override container command

```bash
kubectl run cmd --image=busybox --command -- sleep 3600
```

## Pass container args

```bash
kubectl run args --image=busybox -- sleep 3600
```

## Set environment variable in Pod

```bash
kubectl run env --image=busybox --env=APP=prod -- sleep 3600
```

## Add liveness probe (http)

```bash
kubectl set probe pod nginx --liveness --get-url=http://:80/
```

## Add readiness probe (http)

```bash
kubectl set probe pod nginx --readiness --get-url=http://:80/
```

## Add startup probe (http)

```bash
kubectl set probe pod nginx --startup --get-url=http://:80/
```

## Create ConfigMap from literal

```bash
kubectl create cm app-cm --from-literal=key=val
```

## Use ConfigMap as env in Pod

```bash
kubectl set env pod nginx --from=configmap/app-cm
```

## Mount ConfigMap as volume

```bash
kubectl set volume pod nginx --add --name=cm --type=configmap --configmap-name=app-cm --mount-path=/cm
```

## Create Secret from literal

```bash
kubectl create secret generic app-sec --from-literal=pwd=123
```

## Use Secret as env in Pod

```bash
kubectl set env pod nginx --from=secret/app-sec
```

## Mount Secret as volume

```bash
kubectl set volume pod nginx --add --name=sec --type=secret --secret-name=app-sec --mount-path=/sec
```

## Update Deployment image

```bash
kubectl set image deploy/nginx nginx=nginx:1.25
```

## Check Deployment rollout status

```bash
kubectl rollout status deploy/nginx
```

## Rollback Deployment

```bash
kubectl rollout undo deploy/nginx
```

## Create Job

```bash
kubectl create job pi --image=busybox -- echo ok
```

## Create CronJob

```bash
kubectl create cronjob cj --schedule="*/5 * * * *" --image=busybox -- echo ok
```

## Show non-running Pods (CrashLoopBackOff/Pending)

```bash
kubectl get pods --field-selector=status.phase!=Running
```

## Inspect Pod logs causing CrashLoopBackOff

```bash
kubectl logs podname --previous
```

## Show ImagePullBackOff Pods

```bash
kubectl get pods | grep ImagePullBackOff
```

## Describe Pod for image pull errors

```bash
kubectl describe pod podname
```

# A2b. SCHEDULING & PLACEMENT ESSENTIALS

## Schedule Pod to labeled node (nodeSelector)

```yaml
spec:
  nodeSelector:
    disktype: ssd
```

## Allow Pod on tainted node (tolerations)

```yaml
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

## Assign PriorityClass to Pod

```yaml
spec:
  priorityClassName: high-priority
```

# A3. NETWORKING (CORE ONLY)
## Create ClusterIP Service

```bash
kubectl expose pod nginx --port=80 --type=ClusterIP
```

## Create NodePort Service

```bash
kubectl expose pod nginx --port=80 --type=NodePort
```

## Set Service selector

```bash
kubectl patch svc nginx -p '{"spec":{"selector":{"app":"nginx"}}}'
```

## Check Service endpoints

```bash
kubectl get endpoints nginx
```

## Detect endpoint mismatch

```bash
kubectl describe svc nginx
```

## DNS lookup inside cluster

```bash
kubectl exec pod -- nslookup svcname
```

## DNS lookup with dig

```bash
kubectl exec pod -- dig svcname
```

## Create basic Ingress (host → service)

```bash
kubectl create ingress ing --rule=host/path=svc:80
```

## Create default deny NetworkPolicy

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

## Allow traffic from namespace selector

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ns
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: team
```

## Allow traffic from pod selector

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-pod
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: client
```

# A4. STORAGE (CORVE)

## Create a PersistentVolumeClaim (PVC) with size and access mode

```bash
kubectl create pvc mypvc --storage=1Gi --access-modes=ReadWriteOnce
```

## Create a PVC using a specific StorageClass

```bash
kubectl create pvc mypvc --storage=1Gi --access-modes=ReadWriteOnce --storage-class=fast
```

## Create a PVC with ReadWriteMany access mode

```bash
kubectl create pvc mypvc --storage=1Gi --access-modes=ReadWriteMany
```

## Mount an existing PVC into a Pod

```bash
kubectl run mypod --image=nginx --restart=Never --overrides='{"spec":{"volumes":[{"name":"v","persistentVolumeClaim":{"claimName":"mypvc"}}],"containers":[{"name":"nginx","image":"nginx","volumeMounts":[{"name":"v","mountPath":"/data"}]}]}}'
```

## Create a PersistentVolume with Retain reclaim policy

```yaml
apiVersion: v1
kind: PersistentVolume
spec:
  capacity:
    storage: 1Gi
  accessModes: [ReadWriteOnce]
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /data/pv
```


# A5. CLUSTER ADMIN (ABSOLUTE MUST)

## List all nodes in the cluster

```bash
kubectl get nodes
```

## Describe a node

```bash
kubectl describe node <node-name>
```

## Cordon a node

```bash
kubectl cordon <node-name>
```

## Drain a node safely for maintenance

```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```

## Uncordon a node

```bash
kubectl uncordon <node-name>
```

## Static Pod manifest location on node

```bash
ls /etc/kubernetes/manifests
```

## Detect static pods running on a node

```bash
kubectl get pods -n kube-system -o wide | grep <node-name>
```

## Kubelet configuration file location

```bash
ls /var/lib/kubelet/config.yaml
```

## View kubelet logs

```bash
journalctl -u kubelet
```

## Take an etcd snapshot

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key
```

## Restore etcd snapshot and rewire static pod

```bash
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db --data-dir=/var/lib/etcd-from-backup
```

Edit `/etc/kubernetes/manifests/etcd.yaml` volume hostPath and `--data-dir` to `/var/lib/etcd-from-backup`, then restart kubelet to reload the static pod.

## Initialize control plane with kubeadm

```bash
kubeadm init --pod-network-cidr=192.168.0.0/16
```

Verify nodes and apply CNI after init.

## Configure kubeconfig for root after init

```bash
mkdir -p $HOME/.kube
cp /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

## Join a worker node with kubeadm

```bash
kubeadm join <cp-ip>:6443 --token <token> \
--discovery-token-ca-cert-hash sha256:<hash>
```

Regenerate with `kubeadm token create --print-join-command` if needed.

## Upgrade control plane with kubeadm

```bash
kubeadm upgrade plan
kubeadm upgrade apply v1.28.x
```

Upgrade control plane first, then workers.

## Upgrade kubelet and kubectl on a node

```bash
apt-get install -y kubelet=1.28.x kubectl=1.28.x
systemctl restart kubelet
```

Verify node version skew stays within ±1 minor.

## Renew control plane certificates

```bash
kubeadm cert renew all
systemctl restart kubelet
```

## Create a new bootstrap token

```bash
kubeadm token create --print-join-command
```

# A6. RBAC (CORE)
## Role (namespaced) create

```sh
kubectl create role <role> --verb=get,list,watch --resource=pods -n <ns>
```

## ClusterRole create

```sh
kubectl create clusterrole <clusterrole> --verb=get,list,watch --resource=pods
```

## RoleBinding bind Role to ServiceAccount

```sh
kubectl create rolebinding <rb> --role=<role> --serviceaccount=<ns>:<sa> -n <ns>
```

## ClusterRoleBinding bind ClusterRole to ServiceAccount

```sh
kubectl create clusterrolebinding <crb> --clusterrole=<clusterrole> --serviceaccount=<ns>:<sa>
```

## `kubectl auth can-i` as ServiceAccount in namespace

```sh
kubectl auth can-i <verb> <resource> --as=system:serviceaccount:<ns>:<sa> -n <ns>
```
---

# A7. CORE TROUBLESHOOTING
## Pod won’t start: see events quickly

```sh
kubectl describe pod <pod> -n <ns>
```

## Pod crash looping: show last termination logs

```sh
kubectl logs <pod> -n <ns> --previous
```

## Pod crash looping: show current logs

```sh
kubectl logs <pod> -n <ns>
```

## Pod won’t start: show container statuses (wide)

```sh
kubectl get pod <pod> -n <ns> -o wide
```

## Node NotReady: describe node for reason/events

```sh
kubectl describe node <node>
```

## Node NotReady: list nodes with status

```sh
kubectl get nodes -o wide
```

## Service unreachable: list endpoints for service

```sh
kubectl get endpoints <svc> -n <ns>
```

## Service unreachable: verify service selector/ports fast

```sh
kubectl get svc <svc> -n <ns> -o wide
```

## Service unreachable: verify matching pods by selector

```sh
kubectl get pods -n <ns> -l <key>=<val> -o wide
```

## DNS failure: test DNS from a temp busybox pod

```sh
kubectl run -n <ns> dns --rm -it --image=busybox:1.36 --restart=Never -- nslookup kubernetes.default
```

## DNS failure: check CoreDNS pods

```sh
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

## DNS failure: view CoreDNS logs

```sh
kubectl logs -n kube-system -l k8s-app=kube-dns
```

## Volume mount failure: show pod events (mount errors)

```sh
kubectl describe pod <pod> -n <ns> | sed -n '/Events:/,$p'
```

# B1. ADVANCED kubectl SPEED

## Patch a Deployment image (strategic merge)

```bash
kubectl patch deploy <deploy> -p '{"spec":{"template":{"spec":{"containers":[{"name":"<c>","image":"<img>:<tag>"}]}}}}'
```

## Patch a Service type to NodePort

```bash
kubectl patch svc <svc> -p '{"spec":{"type":"NodePort"}}'
```

## Patch using JSON merge patch type explicitly

```bash
kubectl patch deploy <deploy> --type=merge -p '{"spec":{"replicas":<n>}}'
```

## Patch using JSON patch (RFC6902) to replace replicas

```bash
kubectl patch deploy <deploy> --type=json -p='[{"op":"replace","path":"/spec/replicas","value":<n>}]'
```

## Add a label to a Pod

```bash
kubectl label pod <pod> <k>=<v>
```

## Overwrite an existing label on a Pod

```bash
kubectl label pod <pod> <k>=<v> --overwrite
```

## Remove a label from a Pod

```bash
kubectl label pod <pod> <k>-
```

## Add an annotation to a Pod

```bash
kubectl annotate pod <pod> <k>=<v>
```

## Overwrite an existing annotation on a Pod

```bash
kubectl annotate pod <pod> <k>=<v> --overwrite
```

## Remove an annotation from a Pod

```bash
kubectl annotate pod <pod> <k>-
```

## Update a container image in a Deployment

```bash
kubectl set image deploy/<deploy> <c>=<img>:<tag>
```

## Update all container images in a Deployment by wildcard

```bash
kubectl set image deploy/<deploy> '*=<img>:<tag>'
```

## Set an env var on a Deployment

```bash
kubectl set env deploy/<deploy> <k>=<v>
```

## Remove an env var from a Deployment

```bash
kubectl set env deploy/<deploy> <k>-
```

## Sort events by creation time

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

## Show only Warning events sorted by time

```bash
kubectl get events --field-selector type=Warning --sort-by=.metadata.creationTimestamp
```

# B2. ADVANCED WORKLOAD DETAILS

## List init containers for a Pod

```bash
kubectl get pod <pod> -o jsonpath='{.spec.initContainers[*].name}{"\n"}'
```

## Show init container termination reasons

```bash
kubectl get pod <pod> -o jsonpath='{range .status.initContainerStatuses[*]}{.name}{"="}{.state.terminated.reason}{"\n"}{end}'
```

## Get logs from a specific container in a Pod (sidecar)

```bash
kubectl logs <pod> -c <sidecar>
```

## Tail sidecar logs

```bash
kubectl logs <pod> -c <sidecar> --tail=50
```

## Follow sidecar logs

```bash
kubectl logs <pod> -c <sidecar> -f
```

## Show resource requests for all containers in a Pod

```bash
kubectl get pod <pod> -o jsonpath='{range .spec.containers[*]}{.name}{" req="}{.resources.requests}{"\n"}{end}'
```

## Show resource limits for all containers in a Pod

```bash
kubectl get pod <pod> -o jsonpath='{range .spec.containers[*]}{.name}{" lim="}{.resources.limits}{"\n"}{end}'
```

## Check if a Pod was OOMKilled

```bash
kubectl get pod <pod> -o jsonpath='{range .status.containerStatuses[*]}{.name}{"="}{.lastState.terminated.reason}{"\n"}{end}'
```

## Show last terminated exit codes (OOM often 137)

```bash
kubectl get pod <pod> -o jsonpath='{range .status.containerStatuses[*]}{.name}{"="}{.lastState.terminated.exitCode}{"\n"}{end}'
```

## List PodDisruptionBudgets

```bash
kubectl get pdb
```

## Describe a PodDisruptionBudget

```bash
kubectl describe pdb <pdb>
```

# B3. NETWORKING (EDGE)

## Create a headless Service

```bash
kubectl create svc clusterip <svc> --tcp=80:80 --clusterip=None
```

## Create a multi-port Service (two TCP ports)

```bash
kubectl create svc clusterip <svc> --tcp=80:8080,443:8443
```

## Create an Ingress with a path rule (Prefix)

```bash
kubectl create ingress <ing> --rule="<host>/<path>=<svc>:<port>"
```

## Create an Ingress with a default backend

```bash
kubectl create ingress <ing> --default-backend=<svc>:<port>
```

## Apply a default-deny egress NetworkPolicy for a namespace

```bash
kubectl create networkpolicy default-deny-egress --pod-selector={} --policy-types=Egress
```

## Allow egress DNS (UDP/53) to kube-dns in kube-system

```bash
kubectl create networkpolicy allow-dns-egress --pod-selector={} --policy-types=Egress --egress-to-namespace-selector='kubernetes.io/metadata.name=kube-system' --egress-to-pod-selector='k8s-app=kube-dns' --egress-port=udp:53
```

# B4. STORAGE (EDGE)

## List emptyDir volume type in a Pod

```bash
kubectl explain pod.spec.volumes.emptyDir
```

## List hostPath volume type in a Pod

```bash
kubectl explain pod.spec.volumes.hostPath
```

## Resize an existing PVC

```bash
kubectl patch pvc mypvc -p '{"spec":{"resources":{"requests":{"storage":"2Gi"}}}}'
```

## Check if a PVC resize was applied

```bash
kubectl get pvc mypvc
```

## Identify StatefulSet volumeClaimTemplates

```bash
kubectl get sts mysts -o jsonpath='{.spec.volumeClaimTemplates[*].metadata.name}'
```

---

# B5. CLUSTER ADMIN (EDGE)

## Locate control plane static Pod manifests

```bash
ls /etc/kubernetes/manifests
```

## View kube-apiserver flags

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

## Locate kube-scheduler manifest

```bash
ls /etc/kubernetes/manifests | grep scheduler
```

## Locate kube-controller-manager manifest

```bash
ls /etc/kubernetes/manifests | grep controller
```

## Recall cluster upgrade order

Control plane → worker nodes

## Locate Kubernetes certificates directory

```bash
ls /etc/kubernetes/pki
```

## List containers using crictl

```bash
crictl ps
```

## View container logs with crictl

```bash
crictl logs <container-id>
```

---

# B6. RBAC (EDGE)

## Identify aggregated ClusterRoles

```bash
kubectl get clusterrole -o jsonpath='{range .items[*]}{.metadata.name}{" "}{.aggregationRule}{"\n"}{end}'
```

## Create RoleBinding with multiple subjects

```bash
kubectl create rolebinding rb --role=role1 --user=u1 --user=u2
```

## Check permissions causing Forbidden error

```bash
kubectl auth can-i <verb> <resource> --as <user>
```

## Check permissions in a namespace

```bash
kubectl auth can-i <verb> <resource> -n <ns>
```

# B7. ADVANCED TROUBLESHOOTING

## Identify why a pod is Pending (scheduler reason)

```bash
kubectl describe pod <pod> | grep -A5 Events
```

## Check if a pod is Pending due to insufficient resources

```bash
kubectl describe pod <pod> | grep -i insufficient
```

## List nodes with allocatable resources

```bash
kubectl describe nodes | grep -A5 Allocatable
```

## Verify imagePullSecret exists in namespace

```bash
kubectl get secret <secret> -n <ns>
```

## Confirm pod is using an imagePullSecret

```bash
kubectl get pod <pod> -o jsonpath='{.spec.imagePullSecrets}'
```

## Check image pull error on a pod

```bash
kubectl describe pod <pod> | grep -i image
```

## List NetworkPolicies in a namespace

```bash
kubectl get netpol -n <ns>
```

## Check if pod traffic is blocked by NetworkPolicy (events)

```bash
kubectl describe pod <pod> | grep -i policy
```

## Identify CNI issue via pod sandbox error

```bash
kubectl describe pod <pod> | grep -i cni
```

## Detect CNI failure via node status

```bash
kubectl describe node <node> | grep -i network
```

## Identify kube-proxy issue via service not routing

```bash
kubectl get pods -n kube-system | grep kube-proxy
```

## Check kube-proxy pod logs

```bash
kubectl logs -n kube-system <kube-proxy-pod>
```

# B8. YAML SKELETONS (MEMORIZE)

## Pod with probes (multi-container ready)

```yaml
apiVersion: v1
kind: Pod
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
  - name: sidecar
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
```

## Deployment with rolling update strategy

```yaml
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
  template:
    spec:
      containers:
      - name: app
        image: nginx:1.25
```

## StatefulSet volumeClaimTemplates

```yaml
spec:
  serviceName: app
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ReadWriteOnce]
      resources:
        requests:
          storage: 1Gi
```

## NetworkPolicy deny all ingress

```yaml
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

## Namespaced Role and RoleBinding

```yaml
kind: Role
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list"]
---
kind: RoleBinding
subjects:
- kind: ServiceAccount
  name: app-sa
roleRef:
  kind: Role
  name: read-pods
```

# B9. EXAM HYGIENE / TIME SAVERS

## Create kubectl alias k

```bash
alias k=kubectl
```

## Enable kubectl bash autocompletion

```bash
source <(kubectl completion bash)
```

## vim search and replace in file

```vim
:%s/old/new/g
```

## vim visual block select

```vim
Ctrl+v
```

## vim indent selected lines right

```vim
>
```

## vim indent selected lines left

```vim
<
```

## tmux split pane vertically

```bash
Ctrl+b %
```

## tmux split pane horizontally

```bash
Ctrl+b "
```

## Validate YAML before apply (dry-run)

```bash
kubectl apply -f file.yaml --dry-run=client
```

## Validate YAML with explain

```bash
kubectl explain pod.spec.containers
```

## Avoid namespace mistake by setting default

```bash
kubectl config set-context --current --namespace=<ns>
```

## Common flag fix: correct output flag

```bash
kubectl get pods -o wide
```

## Common flag fix: correct namespace flag

```bash
kubectl get pods -n <ns>
```

## Task: Generate a kubeadm join command for adding a new worker node.
in code snippets:  
Command or YAML:
```bash
kubeadm token create --print-join-command
```

Verify:
```bash
kubeadm token list
```

Tip:
Use --ttl 0 for a non-expiring token during the exam.

## Task: Create a new kubeadm bootstrap token and join a node using it.
in code snippets:  
Command or YAML:
```bash
kubeadm token create --print-join-command > /tmp/join.sh && sh /tmp/join.sh
```

Verify:
```bash
kubectl get nodes -o wide | grep <new-node-name>
```

Tip:
Run join as root on the target node after copying the command.

## Task: Upload control-plane certificates for adding an additional control-plane node (kubeadm).
in code snippets:  
Command or YAML:
```bash
kubeadm init phase upload-certs --upload-certs
```

Verify:
```bash
kubeadm token list | grep certs
```

Tip:
Capture the certificate key output for the join command.

## Task: Join a second control-plane node to an existing cluster using kubeadm.
in code snippets:  
Command or YAML:
```bash
kubeadm join <api-server>:6443 --control-plane --token <token> --discovery-token-ca-cert-hash sha256:<hash> --certificate-key <cert-key>
```

Verify:
```bash
kubectl get nodes -o wide
```

Tip:
Ensure ports 6443/10250 reachable from new control-plane node.

## Task: Reset a node that was previously joined and re-join it cleanly (kubeadm reset workflow).
in code snippets:  
Command or YAML:
```bash
sudo kubeadm reset -f && sudo rm -rf /etc/cni/net.d && sudo systemctl restart kubelet && sudo kubeadm join <api-server>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

Verify:
```bash
kubectl get nodes -o wide | grep <node>
```

Tip:
Delete stale CNI state (/var/lib/cni) if kubelet fails to start.

## Task: Find where control plane static pod manifests live and identify which node they run on.
in code snippets:  
Command or YAML:
```bash
grep staticPodPath /var/lib/kubelet/config.yaml
```

Verify:
```bash
kubectl get pods -n kube-system -o wide | grep -E 'kube-apiserver|kube-scheduler|kube-controller-manager|etcd'
```

Tip:
staticPodPath default is /etc/kubernetes/manifests on control-plane nodes.

## Task: Edit a static pod manifest (kube-apiserver/scheduler/controller-manager/etcd) and confirm it restarts.
in code snippets:  
Command or YAML:
```bash
sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

Verify:
```bash
kubectl get pods -n kube-system -w | grep kube-apiserver
```

Tip:
Save file and wait; kubelet auto-recreates the static pod.

## Task: Find kubelet’s config file and change one safe setting (e.g., clusterDNS/clusterDomain) then verify.
in code snippets:  
Command or YAML:
```bash
sudo sed -i 's/clusterDomain: .*/clusterDomain: cluster.local/' /var/lib/kubelet/config.yaml
sudo systemctl restart kubelet
```

Verify:
```bash
sudo systemctl status kubelet --no-pager
```

Tip:
Check systemd drop-ins for kubelet.service to confirm KUBELET_CONFIG_ARGS path.

## Task: Identify the container runtime used by kubelet and verify it’s running on a node.
in code snippets:  
Command or YAML:
```bash
grep containerRuntimeEndpoint /var/lib/kubelet/config.yaml
```

Verify:
```bash
sudo systemctl status containerd || sudo systemctl status crio
```

Tip:
Use crictl info if runtime endpoint is unclear.

## Task: Configure kubectl access for a non-root user using admin.conf (copy/permissions).
in code snippets:  
Command or YAML:
```bash
sudo mkdir -p /home/user/.kube && sudo cp /etc/kubernetes/admin.conf /home/user/.kube/config && sudo chown user:user /home/user/.kube/config
```

Verify:
```bash
sudo -u user kubectl get nodes
```

Tip:
Set KUBECONFIG env if using a non-default path.

## Task: Verify API server health from the control-plane node (local endpoint / component status equivalent).
in code snippets:  
Command or YAML:
```bash
kubectl get --raw "/readyz"
```

Verify:
```bash
kubectl get componentstatuses
```

Tip:
Use --server https://127.0.0.1:6443 with admin.conf if kubeconfig not set.

## Task: Renew control-plane certs and verify components come back healthy.
in code snippets:  
Command or YAML:
```bash
sudo kubeadm certs renew all && sudo systemctl restart kubelet
```

Verify:
```bash
kubectl get nodes && kubectl get pods -n kube-system
```

Tip:
Check /etc/kubernetes/pki for updated timestamps.

## Task: Perform a control-plane upgrade with kubeadm and verify component versions afterward.
in code snippets:  
Command or YAML:
```bash
sudo kubeadm upgrade apply v1.<X>.Y
sudo apt-get install -y kubelet=1.<X>.Y-00 kubectl=1.<X>.Y-00
sudo systemctl restart kubelet
```

Verify:
```bash
kubectl get nodes -o wide
```

Tip:
Drain other control-plane nodes only if instructed; always back up etcd first.

## Task: Upgrade a worker node (kubelet/kubectl) safely and verify it re-registers Ready.
in code snippets:  
Command or YAML:
```bash
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
sudo apt-get install -y kubelet=1.<X>.Y-00 kubectl=1.<X>.Y-00
sudo systemctl restart kubelet
kubectl uncordon <node>
```

Verify:
```bash
kubectl get nodes
```

Tip:
Never upgrade kubelet beyond kube-apiserver version.

## Task: Identify version skew issues and correct kubelet/kubectl versions to within supported skew.
in code snippets:  
Command or YAML:
```bash
kubectl version --short
```

Verify:
```bash
apt-cache madison kubelet | head
```

Tip:
Kubelet must be <= control-plane version and no more than one minor behind.

## Task: Backup etcd (snapshot) using correct certs and endpoints.
in code snippets:  
Command or YAML:
```bash
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key snapshot save /var/lib/etcd/snapshot.db
```

Verify:
```bash
ETCDCTL_API=3 etcdctl snapshot status /var/lib/etcd/snapshot.db
```

Tip:
Run from control-plane node hosting etcd static pod.

## Task: Restore etcd from snapshot and rewire the etcd static pod to the restored data-dir, then verify cluster state.
in code snippets:  
Command or YAML:
```bash
ETCDCTL_API=3 etcdctl snapshot restore /var/lib/etcd/snapshot.db --data-dir=/var/lib/etcd-restore
sudo sed -i 's#--data-dir=.*#--data-dir=/var/lib/etcd-restore#' /etc/kubernetes/manifests/etcd.yaml
```

Verify:
```bash
kubectl get pods -n kube-system
```

Tip:
Remove old member data-dir to avoid conflicts before restore.

## Task: Create a DaemonSet that runs on every node and verify one pod per node.
in code snippets:  
Command or YAML:
```bash
kubectl apply -f - <<'EOF'
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-syslog
  namespace: default
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
        image: busybox
        command: ["sh", "-c", "sleep 3600"]
EOF
```

Verify:
```bash
kubectl get pods -l app=ds-syslog -o wide
```

Tip:
DaemonSets ignore unschedulable taints unless NoSchedule exists.

## Task: Create a StatefulSet with volumeClaimTemplates and verify PVCs are created per replica.
in code snippets:  
Command or YAML:
```bash
kubectl apply -f - <<'EOF'
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: web
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

Tip:
Ensure a headless Service exists for StatefulSet DNS.

## Task: Perform a rolling update on a Deployment and verify revision history/rollout status.
in code snippets:  
Command or YAML:
```bash
kubectl set image deploy/web nginx=nginx:1.25 && kubectl rollout status deploy/web
```

Verify:
```bash
kubectl rollout history deploy/web
```

Tip:
Use --record is deprecated; history keeps revisions automatically.

## Task: Roll back a Deployment to a previous revision and verify pods use the old image.
in code snippets:  
Command or YAML:
```bash
kubectl rollout undo deploy/web --to-revision=1
```

Verify:
```bash
kubectl get pods -l app=web -o jsonpath='{.items[*].spec.containers[*].image}'
```

Tip:
Check revision numbers with rollout history before undo.

## Task: Pause a Deployment rollout, apply changes, then resume and verify rollout behavior.
in code snippets:  
Command or YAML:
```bash
kubectl rollout pause deploy/web
kubectl set image deploy/web nginx=nginx:1.26
kubectl rollout resume deploy/web
```

Verify:
```bash
kubectl rollout status deploy/web
```

Tip:
Paused deployments accept spec changes without creating new ReplicaSets until resumed.

## Task: Create a Job that retries on failure and verify completion and pod logs.
in code snippets:  
Command or YAML:
```bash
kubectl apply -f - <<'EOF'
apiVersion: batch/v1
kind: Job
metadata:
  name: retry-job
spec:
  backoffLimit: 3
  template:
    spec:
      restartPolicy: OnFailure
      containers:
      - name: retry
        image: busybox
        command: ["sh", "-c", "echo run; exit 0"]
EOF
```

Verify:
```bash
kubectl get jobs retry-job -o wide && kubectl logs job/retry-job
```

Tip:
backoffLimit controls retry count; use describe to see failures fast.

## Task: Create a CronJob with a schedule and verify it created Jobs on schedule.
in code snippets:  
Command or YAML:
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
          - name: ping
            image: busybox
            command: ["sh", "-c", "date"]
EOF
```

Verify:
```bash
kubectl get jobs -l job-name -w
```

Tip:
Set startingDeadlineSeconds to avoid missed runs backlog.

## Task: Create a pod with an initContainer and verify init runs before app container starts.
in code snippets:  
Command or YAML:
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  initContainers:
  - name: init
    image: busybox
    command: ["sh", "-c", "echo init > /work/flag"]
    volumeMounts:
    - name: work
      mountPath: /work
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "cat /work/flag && sleep 3600"]
    volumeMounts:
    - name: work
      mountPath: /work
  volumes:
  - name: work
    emptyDir: {}
EOF
```

Verify:
```bash
kubectl logs init-demo -c app --tail=1
```

Tip:
Init must complete before main containers start; check Pod status Init:0/1.

## Task: Add resource requests/limits to a Pod and verify scheduling behavior (requests gate scheduling).
in code snippets:  
Command or YAML:
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: res-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    resources:
      requests:
        cpu: "250m"
        memory: "256Mi"
      limits:
        cpu: "500m"
        memory: "512Mi"
EOF
```

Verify:
```bash
kubectl describe pod res-pod | grep -A3 Limits
```

Tip:
If Pending, check node allocatable vs. requests with kubectl describe pod.

## Task: Schedule a Pod using nodeSelector and verify it lands on the labeled node.
in code snippets:  
Command or YAML:
```bash
kubectl label node <node> disk=ssd
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: selector-pod
spec:
  nodeSelector:
    disk: ssd
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
EOF
```

Verify:
```bash
kubectl get pod selector-pod -o wide
```

Tip:
Labels must already exist on the target node.

## Task: Schedule a Pod using required nodeAffinity and verify placement.
in code snippets:  
Command or YAML:
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
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
EOF
```

Verify:
```bash
kubectl get pod affinity-pod -o wide
```

Tip:
If Pending, ensure nodes have matching label disktype=ssd.

## Task: Prevent co-scheduling using podAntiAffinity and verify pods spread across nodes.
in code snippets:  
Command or YAML:
```bash
kubectl apply -f - <<'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: anti
spec:
  replicas: 2
  selector:
    matchLabels:
      app: anti
  template:
    metadata:
      labels:
        app: anti
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values: [anti]
            topologyKey: kubernetes.io/hostname
      containers:
      - name: app
        image: busybox
        command: ["sh", "-c", "sleep 3600"]
EOF
```

Verify:
```bash
kubectl get pods -l app=anti -o wide
```

Tip:
Need at least two nodes Ready for required anti-affinity.

## Task: Taint a node and make a Pod tolerate it; verify it schedules only with toleration.
in code snippets:  
Command or YAML:
```bash
kubectl taint nodes <node> key=value:NoSchedule
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: tolerate
spec:
  tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule"
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
EOF
```

Verify:
```bash
kubectl get pod tolerate -o wide
```

Tip:
Remove taint with kubectl taint nodes <node> key:NoSchedule- when done.

## Task: Create a PriorityClass and run a Pod using it; verify priority is applied.
in code snippets:  
Command or YAML:
```bash
kubectl apply -f - <<'EOF'
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100000
globalDefault: false
description: "high"
---
apiVersion: v1
kind: Pod
metadata:
  name: prio-pod
spec:
  priorityClassName: high-priority
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
EOF
```

Verify:
```bash
kubectl get pod prio-pod -o jsonpath='{.spec.priority}'
```

Tip:
Higher value preempts lower if resources scarce.

## Task: Troubleshoot a Pending Pod due to taints/affinity/resources and fix it.
in code snippets:  
Command or YAML:
```bash
kubectl describe pod <pod>
```

Verify:
```bash
kubectl get pod <pod>
```

Tip:
Check Events for FailedScheduling; adjust tolerations/labels/resources accordingly.

## Task: Create a ClusterIP Service selecting a Deployment and verify Endpoints match pods.
in code snippets:  
Command or YAML:
```bash
kubectl expose deploy web --port=80 --target-port=80 --type=ClusterIP
```

Verify:
```bash
kubectl get endpoints web -o wide
```

Tip:
Selector must match pod labels; check with kubectl get pods --show-labels.

## Task: Fix a Service that has no endpoints (selector mismatch) and verify endpoints appear.
in code snippets:  
Command or YAML:
```bash
kubectl patch svc web -p '{\"spec\":{\"selector\":{\"app\":\"web\"}}}'
```

Verify:
```bash
kubectl get endpoints web
```

Tip:
Align Service selector to Deployment labels; patch is fastest.

## Task: Create a NodePort Service and verify connectivity from a node (curl to NodeIP:NodePort).
in code snippets:  
Command or YAML:
```bash
kubectl expose deploy web --type=NodePort --port=80 --target-port=80 --name=web-nodeport
```

Verify:
```bash
NODEPORT=$(kubectl get svc web-nodeport -o jsonpath='{.spec.ports[0].nodePort}')
curl -s http://$(kubectl get node -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}'):${NODEPORT}
```

Tip:
Use node InternalIP; avoid master if tainted without toleration.

## Task: Create a headless Service and verify DNS returns pod A records (StatefulSet use-case).
in code snippets:  
Command or YAML:
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
EOF
```

Verify:
```bash
kubectl run -it --rm dnscheck --image=busybox:1.36 --restart=Never -- nslookup web-headless
```

Tip:
Headless service required for stable DNS entries with StatefulSet.

## Task: Create an Ingress with host + path routing to a Service and verify routing works.
in code snippets:  
Command or YAML:
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
kubectl get ingress web-ing && curl -H "Host: example.com" http://<ingress-ip>
```

Tip:
Ensure ingress controller is installed; check ingress-class annotation if needed.

## Task: Update an Ingress rule (host/path/backend) and verify it takes effect.
in code snippets:  
Command or YAML:
```bash
kubectl patch ingress web-ing --type='json' -p='[{\"op\":\"replace\",\"path\":\"/spec/rules/0/host\",\"value\":\"new.example.com\"}]'
```

Verify:
```bash
kubectl describe ingress web-ing
```

Tip:
Some controllers need reload time; watch events after patch.

## Task: Apply a default-deny ingress NetworkPolicy and verify traffic is blocked.
in code snippets:  
Command or YAML:
```bash
kubectl apply -f - <<'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
EOF
```

Verify:
```bash
kubectl run -it --rm test --image=busybox:1.36 --restart=Never -- wget -qO- web.default.svc.cluster.local || echo blocked
```

Tip:
Ensure CNI supports NetworkPolicy (Calico/Cilium/Weave).

## Task: Allow ingress only from a specific namespace label and verify traffic works only from that namespace.
in code snippets:  
Command or YAML:
```bash
kubectl label namespace prod access=allowed
kubectl apply -f - <<'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-prod-ingress
  namespace: default
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
kubectl run -n prod -it --rm prodtest --image=busybox:1.36 --restart=Never -- wget -qO- web.default.svc.cluster.local
```

Tip:
Pods in other namespaces should fail; quick check with a non-labeled namespace.

## Task: Apply default-deny egress and then allow DNS egress to CoreDNS; verify DNS works.
in code snippets:  
Command or YAML:
```bash
kubectl apply -f - <<'EOF'
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
EOF
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: default
spec:
  podSelector: {}
  policyTypes:
  - Egress
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

Tip:
Ensure CoreDNS labeled k8s-app=kube-dns; adjust selectors if different.

## Task: Troubleshoot DNS failures (CoreDNS pods/config/service) and restore name resolution.
in code snippets:  
Command or YAML:
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns -o wide
kubectl logs -n kube-system -l k8s-app=kube-dns
```

Verify:
```bash
kubectl run -it --rm dnstest --image=busybox:1.36 --restart=Never -- nslookup kubernetes.default
```

Tip:
Check ConfigMap kube-dns in kube-system for Corefile issues.

## Task: Troubleshoot “Service unreachable” (kube-proxy, endpoints, targetPort/port) and fix routing.
in code snippets:  
Command or YAML:
```bash
kubectl get svc web -o yaml | grep -E 'port:|targetPort'
kubectl get endpoints web
```

Verify:
```bash
kubectl run -it --rm curl --image=busybox:1.36 --restart=Never -- wget -qO- web
```

Tip:
Ensure kube-proxy is running: kubectl -n kube-system get ds kube-proxy -o wide.

## Task: Identify CNI plugin health on nodes and fix a pod sandbox/CNI error symptom chain.
in code snippets:  
Command or YAML:
```bash
kubectl get pods -A | grep cni
sudo systemctl status kubelet
```

Verify:
```bash
kubectl describe pod <pod> | grep -i sandbox
```

Tip:
Check /etc/cni/net.d configs; restart kubelet after fixing CNI files.

## Task: Create a PV with a specific reclaimPolicy and verify PV/PVC binding behavior.
in code snippets:  
Command or YAML:
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  capacity:
    storage: 1Gi
  accessModes: ["ReadWriteOnce"]
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/pv1
EOF
```

Verify:
```bash
kubectl get pv pv1
```

Tip:
Ensure hostPath exists on all nodes if scheduled workload may move.

## Task: Create a PVC using a specific StorageClass and verify it binds (or identify why it doesn’t).
in code snippets:  
Command or YAML:
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: claim1
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
kubectl describe pvc claim1
```

Tip:
If Pending, check available PVs or dynamic provisioner logs.

## Task: Troubleshoot a PVC stuck Pending (no matching PV / wrong SC / wrong accessModes) and fix it.
in code snippets:  
Command or YAML:
```bash
kubectl describe pvc <pvc>
```

Verify:
```bash
kubectl get pvc <pvc>
```

Tip:
Adjust storageClassName/accessModes or create matching PV quickly.

## Task: Mount a PVC into a Pod and verify read/write to the mount path.
in code snippets:  
Command or YAML:
```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo data > /data/test && cat /data/test && sleep 3600"]
    volumeMounts:
    - mountPath: /data
      name: data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: claim1
EOF
```

Verify:
```bash
kubectl logs pvc-pod | head -n 1
```

Tip:
Pod must be in same namespace as PVC.

## Task: Expand a PVC (if SC supports) and verify requested capacity is updated/applied.
in code snippets:  
Command or YAML:
```bash
kubectl patch pvc claim1 -p '{"spec":{"resources":{"requests":{"storage":"2Gi"}}}}'
```

Verify:
```bash
kubectl get pvc claim1
```

Tip:
StorageClass must allow expansion (allowVolumeExpansion: true).

## Task: Explain and verify PV lifecycle states (Available/Bound/Released) by deleting PVC and observing PV.
in code snippets:  
Command or YAML:
```bash
kubectl delete pvc claim1
```

Verify:
```bash
kubectl get pv pv1 -w
```

Tip:
Retain reclaimPolicy keeps data; Recycle deprecated.

## Task: Triage a CrashLoopBackOff: find the failing container, view previous logs, and fix the cause.
in code snippets:  
Command or YAML:
```bash
kubectl describe pod <pod>
kubectl logs <pod> --previous
```

Verify:
```bash
kubectl get pod <pod>
```

Tip:
Check liveness/readiness probes misconfigurations causing restarts.

## Task: Triage an ImagePullBackOff: identify exact image error and fix image/secret/policy.
in code snippets:  
Command or YAML:
```bash
kubectl describe pod <pod> | grep -i image
kubectl get secret -A | grep docker
```

Verify:
```bash
kubectl get pod <pod>
```

Tip:
Use kubectl set image deploy/<name> <container>=<image>:<tag> to correct quickly.

## Task: Triage a Pending pod: determine if it’s resources/taints/affinity and resolve.
in code snippets:  
Command or YAML:
```bash
kubectl describe pod <pod>
kubectl get nodes --show-labels
```

Verify:
```bash
kubectl get pod <pod>
```

Tip:
FailedScheduling events point to taints or insufficient CPU/memory.

## Task: Triage a Pod stuck Terminating: find blockers (finalizers/volume) and force delete safely.
in code snippets:  
Command or YAML:
```bash
kubectl get pod <pod> -o jsonpath='{.metadata.finalizers}'
kubectl delete pod <pod> --grace-period=0 --force
```

Verify:
```bash
kubectl get pod <pod>
```

Tip:
If finalizer present, patch to remove finalizers before force delete.

## Task: Debug a Node NotReady: use describe + kubelet logs and restore node to Ready.
in code snippets:  
Command or YAML:
```bash
kubectl describe node <node>
sudo journalctl -u kubelet -n 50
```

Verify:
```bash
kubectl get nodes
```

Tip:
Check network, certificate, and kubelet config quickly.

## Task: Debug a node with DiskPressure/MemoryPressure and restore scheduling health.
in code snippets:  
Command or YAML:
```bash
kubectl describe node <node> | grep -A2 Conditions
```

Verify:
```bash
kubectl get nodes
```

Tip:
Free space under /var/lib/kubelet and cleanup images with crictl rmi --prune.

## Task: Debug kubelet not starting on a node and restore node registration.
in code snippets:  
Command or YAML:
```bash
sudo systemctl status kubelet
sudo journalctl -u kubelet -n 50
```

Verify:
```bash
kubectl get nodes
```

Tip:
Check /var/lib/kubelet/config.yaml and certificate permissions.

## Task: Debug container runtime failure (containerd/cri) using systemctl + crictl and restore workloads.
in code snippets:  
Command or YAML:
```bash
sudo systemctl status containerd
sudo crictl ps -a
```

Verify:
```bash
kubectl get pods -A
```

Tip:
Restart runtime with sudo systemctl restart containerd after fixing config.

## Task: Debug a missing control plane component (static pod not running) and bring it back.
in code snippets:  
Command or YAML:
```bash
ls /etc/kubernetes/manifests
sudo crictl ps -a | grep kube-apiserver
```

Verify:
```bash
kubectl get pods -n kube-system
```

Tip:
Restore manifest yaml; kubelet recreates static pod automatically.

## Task: Debug API server unreachable from kubectl and restore cluster access (kubeconfig/context/certs).
in code snippets:  
Command or YAML:
```bash
kubectl config view
kubectl config use-context <ctx>
```

Verify:
```bash
kubectl get nodes
```

Tip:
If cert expired, renew with kubeadm certs renew apiserver and restart kubelet.

## Task: Debug scheduler/controller-manager issues by checking static pod logs/status and fixing manifest flags.
in code snippets:  
Command or YAML:
```bash
sudo crictl logs $(sudo crictl ps | grep kube-scheduler | awk '{print $1}')
sudo vi /etc/kubernetes/manifests/kube-scheduler.yaml
```

Verify:
```bash
kubectl get pods -n kube-system -l component=kube-scheduler
```

Tip:
Check leader-elect flags and kubeconfig paths in manifest.

## Task: Debug etcd issues (static pod/logs/certs) and restore API functionality.
in code snippets:  
Command or YAML:
```bash
sudo crictl logs $(sudo crictl ps | grep etcd | awk '{print $1}')
sudo ls -l /etc/kubernetes/pki/etcd
```

Verify:
```bash
kubectl get componentstatuses
```

Tip:
Cert/key mismatches cause etcd startup failure; compare against manifest paths.

## Task: Debug CoreDNS resolution failures end-to-end and confirm resolution inside a test pod.
in code snippets:  
Command or YAML:
```bash
kubectl get svc -n kube-system kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

Verify:
```bash
kubectl run -it --rm dnscheck --image=busybox:1.36 --restart=Never -- nslookup kubernetes.default
```

Tip:
Check kube-proxy iptables/ipvs if CoreDNS Service ClusterIP unreachable.

## Task: Debug Service routing failures: endpoints, kube-proxy, iptables/ipvs symptoms, and fix.
in code snippets:  
Command or YAML:
```bash
kubectl get endpoints <svc>
kubectl -n kube-system logs ds/kube-proxy --tail=20
```

Verify:
```bash
kubectl run -it --rm curl --image=busybox:1.36 --restart=Never -- wget -qO- <svc>
```

Tip:
If iptables mode, check iptables -t nat -L -n | grep <svc>.

## Task: Debug NetworkPolicy blocking traffic: confirm policies selecting pods and adjust to allow required flow.
in code snippets:  
Command or YAML:
```bash
kubectl get networkpolicy -A
kubectl describe networkpolicy <np> -n <ns>
```

Verify:
```bash
kubectl run -it --rm tester --image=busybox:1.36 --restart=Never -- curl -s http://<svc> || echo blocked
```

Tip:
Add explicit allow rules; empty ingress/egress with policy present denies traffic.
