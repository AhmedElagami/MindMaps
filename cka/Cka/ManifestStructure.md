# Manifest Structure

## Content

### ❓ Explain what the following full YAML manifest does when deployed as a Kubernetes Service, including the purpose of each section and port mapping, and what command is used to deploy it declaratively.
The YAML manifest deploys a LoadBalancer Service in Kubernetes. Here's the complete manifest with explanations:

```bash
kind: Service
apiVersion: v1
metadata:
  name: svc-lb
spec:
  type: LoadBalancer
  ports:
  - port: 9000           <<---- Load balancer port
    targetPort: 8080     <<---- Application port inside container
  selector:
    chapter: services
```

**Purpose of each section:**
- The first two lines (`kind: Service` and `apiVersion: v1`) tell Kubernetes to deploy a Service object based on the v1 API schema.
- The `metadata` block names this Service `svc-lb` and registers the name with the internal cluster DNS. You can also define custom labels and annotations here.
- The `spec` section defines all the front-end and back-end details. This example tells Kubernetes to deploy a LoadBalancer Service that listens on port 9000 on the front end and sends traffic to Pods with the `chapter: services` label on port 8080.

**Port mapping:**
- `port: 9000` - This is the port the load balancer exposes externally
- `targetPort: 8080` - This is the application port inside the containers where traffic is forwarded

To deploy this Service declaratively, use the following command:

```bash
$ kubectl apply -f lb.yml
service/svc-lb created
```

The output `service/svc-lb created` confirms successful creation of the Service.

### ❓ What functions does the metadata block perform for a Kubernetes Service, and how does the spec section define the relationship between front-end Service configuration and back-end Pod targeting?
The `metadata` block tells Kubernetes to name this Service `svc-lb` and register the name with the internal cluster DNS. You can also define custom labels and annotations here.

The `spec` section defines all the front-end and back-end details. This example tells Kubernetes to deploy a LoadBalancer Service that listens on port 9000 on the front end and sends traffic to Pods with the `chapter: services` label on port 8080.

### ❓ What Kubernetes resources are created when executing the full kubectl apply -f app.yml command?
When executing `kubectl apply -f app.yml`, the following Kubernetes resources are created:

```bash
$ kubectl apply -f app.yml
deployment.apps/wasm-spin created
service/svc-wasm created
ingress.networking.k8s.io/ing-wasm created
```

### ❓ What does the following YAML configuration define in terms of Kubernetes objects and their namespaces and how does it enable running identical services in different namespaces?
The YAML configuration defines a multi-namespace deployment setup that enables running identical services in different namespaces. It creates:

1. Two Namespaces: `dev` and `prod`
2. Two Deployments: Both named `enterprise` but in different namespaces (dev and prod)
3. Two Services: Both named `ent` but in different namespaces (dev and prod)
4. One standalone Pod: `jump` Pod deployed only to the dev namespace

The structure enables identical configurations by leveraging Kubernetes namespaces to partition the cluster address space. Object names must be unique within a namespace but not across namespaces, allowing identical names for deployments and services in different namespaces.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
---
apiVersion: v1
kind: Namespace
metadata:
  name: prod
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: enterprise
  namespace: dev
  spec:
    replicas: 2
    template:
      spec:
        containers:
        - image: nigelpoulton/k8sbook:text-dev
          name: enterprise-ctr
          ports:
          - containerPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: enterprise
  namespace: prod
  spec:
    replicas: 2
    template:
      spec:
        containers:
        - image: nigelpoulton/k8sbook:text-prod
          name: enterprise-ctr
          ports:
          - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: ent
  namespace: dev
  spec:
    selector:
      app: enterprise
    ports:
      - port: 8080
    type: ClusterIP
---
apiVersion: v1
kind: Service
metadata:
  name: ent
  namespace: prod
  spec:
    selector:
      app: enterprise
    ports:
      - port: 8080
    type: ClusterIP
---
apiVersion: v1
kind: Pod
metadata:
  name: jump
  namespace: dev
  spec:
    terminationGracePeriodSeconds: 5
    containers:
    - name: jump
      image: ubuntu
      tty: true
      stdin: true
```

### ❓ What is the purpose of running the kubectl apply command with the -f sd-example.yml file, and what resources does it create based on the output shown?
The `kubectl apply -f sd-example.yml` command is used to deploy Kubernetes resources defined in the sd-example.yml file. Based on the output shown, this command creates the following resources:

```bash
$ kubectl apply -f sd-example.yml
namespace/dev created
namespace/prod created
deployment.apps/enterprise created
deployment.apps/enterprise created
service/ent created
service/ent created
pod/jump-pod created
```

It creates two namespaces (dev and prod), two deployments of the enterprise application (one in each namespace), two services named 'ent' (one in each namespace), and a standalone pod called 'jump-pod' in the dev namespace.

