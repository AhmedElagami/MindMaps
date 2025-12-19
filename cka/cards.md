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
