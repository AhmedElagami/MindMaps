# Pod Immutability

## Content

### ❓ Explain the concept of Pod immutability and what it means for operational practices in Kubernetes, including the proper procedure for updating Pods according to this principle.
Pods are immutable. This means you never change them once they're running. For example, if you need to change or update a Pod, you always replace it with a new one running the updates. You should never log on to a Pod and change it. This means any time we talk about "updating Pods", we always mean deleting the old one and replacing it with a new one. This can be a huge mindset change for some of us, but it fits nicely with modern tools and GitOps-style workflows.

### ❓ What does the term 'immutable' mean in the context of Pods, how does this differ from traditional server management practices, and what is the three-step process for making changes to a Pod?
In the context of Pods, 'immutable' means you cannot modify them after you've deployed them. This can be a huge mindset change if you're from a traditional background where you regularly patched live servers and logged on to them to make fixes and configuration changes.

If you need to change a Pod, you create a new one with the changes, delete the old one, and replace it with the new one. This three-step process (create new → delete old → replace) is fundamentally different from traditional server management where you would typically modify running systems directly.

### ❓ Why is making changes to live Pods considered an anti-pattern according to Kubernetes design principles?
Making changes to live Pods is considered an anti-pattern because Pods are designed as immutable objects. Kubernetes treats Pods as immutable entities that shouldn't be changed after deployment. This design principle ensures consistency, reliability, and predictability in the container orchestration system.

### ❓ What are the two levels of immutability that apply to Pods and how does Kubernetes handle each?
Immutability applies to Pods at two levels:

1. **Object immutability (the Pod)** - Kubernetes handles this by preventing changes to a running Pod's configuration. The system enforces that certain attributes of a live Pod cannot be modified.

2. **App immutability (containers)** - Kubernetes can't always prevent you from changing the app and filesystem inside of containers. You're responsible for ensuring containers and their apps are stateless and immutable.

While Kubernetes enforces object-level immutability at the system level, container-level immutability is the responsibility of the user and application design.

### ❓ What command should you run from your local terminal to edit a Pod's configuration, and which editors typically open for Mac/Linux versus Windows users?
You need to run the command `kubectl edit pod hello-pod` from your local terminal. For Mac and Linux users, it will typically open the session in the default editor (typically vi or vim), whereas for Windows, it's usually the default text editor.

### ❓ Explain what the following full kubectl edit command does and what specific container attributes are marked as immutable in the YAML output.
The command `kubectl edit pod hello-pod` opens the Pod's configuration in your default editor for modification. In the YAML output, several container attributes are marked as immutable and cannot be changed:

```bash
$ kubectl edit pod hello-pod

# Please edit the object below. Lines beginning with a '#' will be ignored...
apiVersion: v1
kind: Pod
metadata:
  <Snip>
  labels:
    version: v1
    zone: prod
  name: hello-pod                    
  namespace: default
  resourceVersion: "432621"
  uid: a131fb37-ceb4-4484-9e23-26c0b9e7b4f4
spec:
  containers:
  - image: nigelpoulton/k8sbook:1.0
    imagePullPolicy: IfNotPresent
    name: hello-ctr                  <<---- Try to change this
    ports:
    - containerPort: 8080            <<---- Try to change this
      protocol: TCP
    resources:
      limits:
        cpu: 500m                    <<---- Try to change this
        memory: 256Mi                <<---- Try to change this
      requests:
        cpu: 500m                    <<---- Try to change this
        memory: 256Mi                <<---- Try to change this
```

The immutable attributes marked for change include: container name (`hello-ctr`), container port (`8080`), and all resource specifications including CPU and memory limits and requests.

### ❓ What happens when you attempt to save changes to immutable Pod attributes after editing the configuration file?
When you edit the file, save your changes, and close your editor, you'll get a message telling you the changes are forbidden because the attributes are immutable.

### ❓ What key combination should you use to exit if you get stuck inside the editor session?
If you get stuck inside the editor session, you can probably exit by typing the following key combination and then pressing the appropriate key (though the specific combination is not fully specified in the provided content).

### ❓ Why are all update operations in Kubernetes actually replacement operations rather than true updates and what fundamental property of Pods makes this necessary?
All update operations in Kubernetes are actually replacement operations rather than true updates because Pods are immutable objects. This fundamental property of Pods means you never change or update them after you've deployed them. When you update a Pod, you're actually deleting it and replacing it with a new one. The immutability of Pods is a core design principle in Kubernetes that ensures consistency and predictability in the system.

