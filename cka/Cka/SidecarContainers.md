# Sidecar Containers

## Content

### ❓ What does Figure 2.8 illustrate about multi-container Pods and service mesh sidecars?
![Figure 2.8 - Multi-container service mesh Pod](media/figure2-8.png)

Figure 2.8 shows a multi-container Pod with a main application container and a service mesh sidecar. Sidecar is jargon for a helper container that runs in the same Pod as the main app container and provides additional services. In Figure 2.8, the service mesh sidecar encrypts network traffic and provides telemetry.

### ❓ What is the primary function of a sidecar container in a Kubernetes Pod and what are five examples of functionality they can provide?
The primary function of a sidecar container is to add functionality to an application without having to add it directly to the application container. Examples of functionality that sidecar containers can provide include:

1. Scraping logs
2. Monitoring & syncing remote content
3. Brokering connections
4. Munging data
5. Encrypting network traffic

### ❓ What does Figure 4.6 illustrate about the relationship between application containers and service mesh sidecars in a multi-container Pod and why is it critical for service mesh sidecars to start before the main application container?
Figure 4.6 shows a multi-container Pod with an application container and a service mesh sidecar intercepting and encrypting all network traffic. It's critical for the service mesh sidecar to start before the main application container and remain running for the entire Pod lifecycle because if the sidecar isn't running, the application container cannot use the network. The sidecar handles network traffic interception and encryption, so the application container depends on it for network connectivity.

![Service Mesh Sidecar Diagram](media/figure4-6.png)

### ❓ What were the limitations of implementing sidecars as regular containers in older Kubernetes versions and how has native sidecar support evolved from version 1.28 to 1.32?
In older versions of Kubernetes, there was no concept of sidecar containers, and we had to implement them as regular containers. This was problematic as there was no reliable way to start sidecars before app containers, keep them running alongside app containers, or stop them after app containers.

Fortunately, Kubernetes v1.28 introduced native sidecars as an alpha feature and progressed them to beta status in v1.29. As of v1.32, sidecar containers are still a beta feature, and you should use them with caution. However, they're enabled by default and used by many notable projects, including Argo CD and Istio. We should expect them to reach the GA (stable) milestone very soon.

### ❓ What three guarantees does Kubernetes provide for native sidecar containers when they are defined with restartPolicy set to Always?
When you define sidecars as init containers with the restartPolicy set to Always, Kubernetes guarantees they will:

1. Start before the main application container
2. Keep running alongside the main application container
3. Terminate after the main application container

Aside from the above, they follow the other rules of init containers, such as startup order, and you can attach probes to manage and monitor their lifecycles.

### ❓ What is the primary function of a sidecar container in a Kubernetes Pod and how do sidecar containers benefit from running in the same Pod as application containers?
The job of a sidecar container is to augment an application container by providing a secondary service such as log scraping or synchronizing with a remote repository. Kubernetes runs Sidecar containers in the same Pod as application containers so they can share resources such as volumes.

### ❓ What is the primary reason Kubernetes runs Sidecar containers in the same Pod as application containers and what three lifecycle guarantees does Kubernetes provide for sidecar containers in relation to app containers?
Kubernetes runs Sidecar containers in the same Pod as application containers so they can share resources such as volumes. Kubernetes also guarantees that sidecars will start before app containers, run as long as app containers run, and terminate after app containers.

### ❓ What specific restart policy setting distinguishes a sidecar container from a regular init container?
The restart policy setting that distinguishes a sidecar container from a regular init container is "Always". This restart policy is what sets sidecars apart from regular init containers and ensures they'll run alongside the app container. The ctr-sync sidecar has the restartPolicy: Always setting, which only applies to the container and overrides any restart policy you might set at the Pod level.

### ❓ Explain the full YAML manifest structure including how the sidecar container is configured, what specific environment variables are configured for the git-sync sidecar container and what purpose each serves, and how the volume mounts in both containers enable communication between the sidecar and application containers.
The following YAML file defines the multi-container Pod with a single sidecar container called ctr-sync and a single app container called ctr-web:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: git-sync
  labels:
    app: sidecar
spec:
  initContainers:                     
  - name: ctr-sync                    ---┐  
    restartPolicy: Always                |      <<---- Setting to "Always" makes this a sidecar
    image: k8s.gcr.io/git-sync:v3.1.6    | 
    volumeMounts:                        | 
    - name: html                         | S
      mountPath: /tmp/git                | i
    env:                                 | d
    - name: GIT_SYNC_REPO                | e
      value: https://github.com...       | c
    - name: GIT_SYNC_BRANCH              | a
      value: master                      | r
    - name: GIT_SYNC_DEPTH               | 
      value: "1"                         | 
    - name: GIT_SYNC_DEST                | 
      value: "html"                   ---┘ 
  containers:
  - name: ctr-web                     ---┐ 
    image: nginx                         | A
    volumeMounts:                        | p 
    - name: html                         | p
      mountPath: /usr/share/nginx/    ---┘
  volumes:
  - name: html
    emptyDir: {}
```

The git-sync sidecar container has the following environment variables configured:
- `GIT_SYNC_REPO`: Specifies the GitHub repository URL to synchronize from
- `GIT_SYNC_BRANCH`: Defines which branch to synchronize (master)
- `GIT_SYNC_DEPTH`: Sets the depth of the git clone ("1" for shallow clone)
- `GIT_SYNC_DEST`: Specifies the destination directory within the volume ("html")

Both containers use volume mounts to enable communication:
- The sidecar container (ctr-sync) mounts the 'html' volume at `/tmp/git`
- The application container (ctr-web) mounts the same 'html' volume at `/usr/share/nginx/`
- This shared volume allows the sidecar to synchronize content from GitHub and the app container to serve that content

### ❓ What is the specific function of the ctr-sync sidecar container in relation to the GitHub repository and how does the ctr-web application container utilize the content synchronized by the sidecar container?
The ctr-sync sidecar container watches a GitHub repo and synchronizes any changes into a shared volume called html. The ctr-web app container watches this shared volume and serves a web page from its contents.

### ❓ What are the three main actions you'll perform in the hands-on example to demonstrate sidecar functionality?
In the example you're about to follow, you'll start the app, update the remote GitHub repo, and prove that the sidecar container synchronizes the updates.

### ❓ What are the minimum system requirements to successfully complete this sidecar container example?
You'll need Kubernetes v1.29 or later and a GitHub account to follow this example. GitHub accounts are free.

### ❓ What are the sequential steps required to complete the multi-container Pod example with a sidecar container and what specific actions are required to fork the GitHub repository for this exercise?
The sequential steps required to complete the multi-container Pod example with a sidecar container are:

1. **Fork the GitHub repo**: You'll need a GitHub account (they're free). Point your browser to the provided URL, click the Fork dropdown button, choose the + Create a new fork option, fill in the required details, and click the green Create fork button. This will take you to your newly forked repo where you can copy its URL. Be sure to copy the URL of your forked repo.

2. **Update the YAML file with the URL of your forked repo**: Return to your local machine, edit the file in the directory, paste the copied URL into the field, and save your changes.

3. **Deploy the app**: Run the following command to deploy the application from the file:

```bash
$ kubectl apply -f initsidecar.yml
pod/git-sync created
service/svc-sidecar created
```

It will deploy the Pod with the app and sidecar containers, as well as a Service that you'll use to connect to the app.

4. **Connect to the app and see it display This is version 1.0**: Once the Pod is running, you can get the Service IP/DNS and paste it into a new browser tab to see the web page displaying "This is version 1.0".

5. **Make a change to your fork of the GitHub repo**: You must complete this step against your forked repo. Go to your forked repo and edit the file. Change the line to something different, then save and commit your changes.

6. **Verify your changes appear on the app's web page**: Refresh the app's web page to see your updates.

### ❓ What does the following kubectl apply command create when deploying the sidecar application?
The `kubectl apply -f initsidecar.yml` command creates:

```bash
$ kubectl apply -f initsidecar.yml
pod/git-sync created
service/svc-sidecar created
```

This command deploys a Pod named "git-sync" and a Service named "svc-sidecar". The Pod contains both the app and sidecar containers, and the Service provides network access to connect to the application.

### ❓ How does the sidecar container pattern demonstrated in this exercise ensure proper initialization before the main application container starts and what steps are required to verify that changes made to the forked GitHub repository are reflected in the deployed application?
The sidecar container pattern ensures proper initialization before the main application container starts through Kubernetes' init container mechanism. The timestamps from the `kubectl describe` output show that Kubernetes started the ctr-sync sidecar container before it started the ctr-web app container. The sidecar started before the app container and is still running, providing the initialization and synchronization functionality needed by the main application.

To verify that changes made to the forked GitHub repository are reflected in the deployed application:

1. Make a change to your fork of the GitHub repo (you must complete this step against your forked repo)
2. Go to your forked repo and edit the file
3. Change the line to something different, then save and commit your changes
4. Refresh the app's web page to see your updates

The sidecar container pattern ensures that changes committed to the repository will be synchronized and reflected in the application.

### ❓ What was the observed behavior regarding the startup order and current state of the sidecar container compared to the app container?
The sidecar started before the app container and is still running.

### ❓ What two methods were used to prove the functionality of the sidecar container setup?
You proved this using commands, but you also proved it by changing the contents of your forked GitHub repo and witnessing your changes appear in the application.

