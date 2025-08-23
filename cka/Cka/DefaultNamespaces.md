# Default Namespaces

## Content

### ❓ What does the output of the following command reveal about the default Namespaces present in a Kubernetes cluster and what is the purpose of each of the four default Namespaces?
The `kubectl get namespaces` command reveals that every Kubernetes cluster has a set of pre-created Namespaces:

```bash
$ kubectl get namespaces
NAME             STATUS    AGE
default           Active   2d
kube-system       Active   2d
kube-public       Active   2d
kube-node-lease   Active   2d
```

Each default Namespace serves a specific purpose:
- The **default Namespace** is where new objects go if you don't specify a Namespace when creating them
- **kube-system** is where control plane components such as the internal DNS service and the metrics server run
- **kube-public** is for objects that need to be readable by anyone
- **kube-node-lease** is used for node heartbeats and managing node leases

### ❓ Interpret the output of the following command that lists all Namespaces and their status.
The following command lists all Namespaces and shows that both hydra and shield Namespaces are Active:

```bash
$ kubectl get ns
NAME         STATUS   AGE
<Snip>
hydra        Active   49s
shield       Active   3s
```

This output shows:
- Both Namespaces (hydra and shield) have STATUS: Active
- hydra has been active for 49 seconds
- shield has been active for 3 seconds
- The \<Snip\> indicates other existing Namespaces were omitted from the display

### ❓ What does Figure 10.7 illustrate about Kubernetes namespace configurations?
Figure 10.7 illustrates identical configurations running in different Kubernetes namespaces. It shows a single cluster divided into two namespaces called dev and prod, with both namespaces having identical instances of the same Service. This demonstrates how namespaces enable running parallel dev and prod configurations on the same cluster.

![Figure 10.7 - Identical configurations in different Namespaces](media/figure10-7.png)

### ❓ Analyze the output of the 'kubectl get all --namespace' commands to compare the deployment and service configurations between the dev and prod namespaces.
The output shows the deployment and service configurations for both dev and prod namespaces:

```bash
$ kubectl get all --namespace dev
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/enterprise   2/2     2            2           51s

NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/ent   ClusterIP   10.96.138.186   <none>        8080/TCP   51s
<Snip>

$ kubectl get all --namespace prod
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/enterprise   2/2     2            2           1m24s

NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/ent   ClusterIP   10.96.147.32   <none>        8080/TCP   1m25s
<snip>
```

Both namespaces have identical deployment configurations for the 'enterprise' app with 2/2 replicas ready. The services are both ClusterIP type services named 'ent' listening on port 8080, but they have different ClusterIP addresses (10.96.138.186 for dev and 10.96.147.32 for prod). The prod namespace resources are slightly older (1m24s/1m25s) compared to the dev namespace resources (51s).

### ❓ What are the two namespaces created in this service discovery example, and what application components does each namespace contain?
The two namespaces created are called 'dev' and 'prod'. Each namespace contains:
- An instance of the enterprise application (deployment)
- An instance of the 'ent' Service

The dev Namespace also has a standalone Pod called 'jump' beyond these enterprise application components.

### ❓ What command creates the psa-test Namespace and what is its purpose in the PSA demonstration?
The following command creates a new Namespace called psa-test:

```bash
$ kubectl create ns psa-test
```

The purpose of this Namespace is to demonstrate Pod Security Admission by applying security policies and testing Pod deployments against those policies.

