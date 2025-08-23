# State Transition Management

## Content

### ❓ What action triggers the creation of a new ReplicaSet with an updated hash value?
Making changes to the Pod template section of the Deployment manifest initiates a rollout and creates a new ReplicaSet with a hash of the updated Pod template. This action triggers the creation of a new ReplicaSet with an updated hash value.

### ❓ What is the purpose of the 'kubectl scale' command with the '--replicas' flag and explain the sequence of commands and their outputs when scaling a deployment from 10 to 5 replicas?
The `kubectl scale` command with the `--replicas` flag is used to imperatively scale a deployment to a specific number of replicas. This command allows you to manually adjust the number of running Pod instances without modifying the declarative YAML manifest.

When scaling a deployment from 10 to 5 replicas, the sequence of commands and outputs would be:

```bash
$ kubectl scale deploy hello-deploy --replicas 5
deployment.apps/hello-deploy scaled

$ kubectl get deploy hello-deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
hello-deploy   5/5     5            5           29m
```

The first command scales the deployment down to 5 replicas and confirms the operation with "deployment.apps/hello-deploy scaled". The second command verifies that the scaling operation was successful, showing that the deployment now has 5 out of 5 replicas ready, up-to-date, and available.

### ❓ How do scaling operations differ from rolling updates in terms of execution time according to the text?
Scaling operations differ from rolling updates in terms of execution time in that scaling operations are almost instantaneous, while rolling updates take significantly longer. The text states: "You may have noticed that scaling operations are almost instantaneous. This is not the case with rolling updates which you're about to see next." This highlights that scaling simply adjusts the number of existing Pod replicas, whereas rolling updates involve a more complex process of gradually replacing old Pods with new ones while maintaining application availability.

### ❓ What are the four equivalent terms used to describe the process of updating applications in Kubernetes?
The four equivalent terms used to describe the process of updating applications in Kubernetes are: rollout, release, zero-downtime update, and rolling update. These terms all mean the same thing and are used interchangeably to describe the process of updating applications without causing downtime.

### ❓ Analyze the following YAML snippet and identify which specific line needs to be updated to change the image version from 1.0 to 2.0.
In the YAML snippet, the specific line that needs to be updated to change the image version from 1.0 to 2.0 is the `image: nigelpoulton/k8sbook:2.0` line, which is marked with `<<---- Update this line to 2.0` in the comment.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-deploy
spec:
  replicas: 10
  <Snip>
  template:
    <Snip>
    spec:
      containers:
      - name: hello-pod
        image: nigelpoulton/k8sbook:2.0          <<---- Update this line to 2.0
        ports:
        - containerPort: 8080
```

This line specifies the container image and tag that the Pods will use. Changing the tag from the previous version (likely 1.0) to 2.0 will trigger the rollout of the new application version.

### ❓ What section of the YAML file contains all the settings that tell Kubernetes how to perform the update?
The `spec` section of the YAML file contains all the settings that tell Kubernetes how to perform the update.

### ❓ Which section of the Deployment YAML file contains the settings that govern how Kubernetes executes rollouts?
The section of the Deployment YAML file that contains the settings governing how Kubernetes executes rollouts is the `spec` section. This section includes all the configuration settings that tell Kubernetes how to perform the update, including rollout strategy, revision history limits, progress deadlines, and minimum ready seconds.

### ❓ Explain what each configuration parameter in the deployment strategy YAML snippet does during a rolling update, including revisionHistoryLimit, progressDeadlineSeconds, minReadySeconds, and the rollingUpdate constraints.
The deployment strategy YAML snippet contains several key configuration parameters that control how Kubernetes performs rolling updates:

```html
<Snip>
revisionHistoryLimit: 5          <<---- Keep the config from the five previous versions
progressDeadlineSeconds: 300     <<---- Give each new replica five minutes to start
minReadySeconds: 10              <<---- Wait 10 seconds after the previous replica has started
strategy:
  type: RollingUpdate            <<---- Incrementally replace replicas
  rollingUpdate:
    maxUnavailable: 1            <<---- Can take one replica away during update operation
    maxSurge: 1                  <<---- Can add one extra replica during update operation
<Snip>
```

- **revisionHistoryLimit: 5**: Tells Kubernetes to keep the configs from the previous five releases for easy rollbacks.
- **progressDeadlineSeconds: 300**: Tells Kubernetes to give each new Pod replica a five-minute window to start properly before assuming it's failed. It's OK if they start faster.
- **minReadySeconds: 10**: Throttles the rate at which Kubernetes replaces replicas. This configuration tells Kubernetes to wait 10 seconds between each replica. Longer waits give you a better chance of catching problems and preventing scenarios where you replace all replicas with broken ones. In the real world, you'll need to make this value large enough to trap common failures.
- **strategy.type: RollingUpdate**: Tells Kubernetes to update using the RollingUpdate strategy.
- **rollingUpdate.maxUnavailable: 1**: Never have more than one Pod below desired state.
- **rollingUpdate.maxSurge: 1**: Never have more than one Pod above desired state.

### ❓ What are the two key constraints specified in the rollingUpdate strategy regarding pod availability during deployment, and how do they work with a desired state of ten replicas to update two pods at a time?
The rollingUpdate strategy specifies two key constraints:

1. **Never have more than one Pod below desired state (maxUnavailable: 1)**
2. **Never have more than one Pod above desired state (maxSurge: 1)**

The desired state of this app is ten replicas. Therefore, maxSurge: 1 means Kubernetes can go up to 11 replicas during the rollout, and maxUnavailable: 1 allows it to go down to 9. The net result is a rollout that updates two Pods at a time (the delta between 9 and 11 is 2).

### ❓ Based on the configuration parameters, why would a rollout of ten replicas take approximately a minute or two to complete, and what does the kubectl rollout status command output reveal about progress?
The rollout replaces two Pods at a time with a ten-second wait after each (due to minReadySeconds: 10). This means it will take a minute or two to complete for ten replicas.

You can monitor the progress with the kubectl rollout status command, which reveals how many new replicas have been created and are ready:

```bash
$ kubectl rollout status deployment hello-deploy
Waiting for deployment "hello-deploy" rollout... 4 out of 10 new replicas...
Waiting for deployment "hello-deploy" rollout... 4 out of 10 new replicas...
Waiting for deployment "hello-deploy" rollout... 6 out of 10 new replicas...
^C
```

The output shows the progressive creation of new replicas (4 out of 10, then 6 out of 10), indicating the rolling update is proceeding according to the configured strategy.

### ❓ How can you observe the effect of maxUnavailable: 1 during an ongoing rollout, and what would the replica count show?
If you quit monitoring the progress while the rollout is still happening, you can run kubectl commands and see the effect of the update-related settings. For example, the following command shows that six replicas have already been updated, and you currently have nine. Nine is one less than the desired state of ten and is the result of the maxUnavailable: 1 value in the manifest. This demonstrates that maxUnavailable: 1 allows the deployment to have one replica below the desired state during the update process.

### ❓ Explain the purpose and effect of the 'kubectl rollout pause deploy hello-deploy' command shown in the code block.
The `kubectl rollout pause deploy hello-deploy` command is used to temporarily halt an ongoing deployment rollout. Its purpose is to pause the update process, allowing you to inspect the current state or perform other operations before continuing.

```bash
$ kubectl rollout pause deploy hello-deploy
deployment.apps/hello-deploy paused
```

When executed, this command:
- Stops the rolling update process immediately
- Freezes the current state of the deployment
- Leaves some Pods running the old version and some running the new version
- Allows you to examine the deployment's intermediate state using commands like `kubectl describe`

### ❓ What does the 'kubectl rollout resume deploy hello-deploy' command accomplish in the context of a paused deployment update?
The `kubectl rollout resume deploy hello-deploy` command resumes a previously paused deployment rollout, allowing the update process to continue from where it was paused.

```bash
$ kubectl rollout resume deploy hello-deploy
deployment.apps/hello-deploy resumed
```

This command:
- Restarts the rolling update process that was previously paused
- Continues replacing old Pods with new ones according to the deployment strategy
- Follows the same update rules (maxUnavailable, maxSurge, etc.) as the original rollout
- Allows the deployment to progress toward completion

### ❓ What does the 'kubectl rollout undo deployment hello-deploy --to-revision=1' command do, why is it considered imperative, and what precautions should be taken?
The command 'kubectl rollout undo deployment hello-deploy --to-revision=1' reverts the application to revision 1. This is an imperative command and not recommended because it makes changes directly to the cluster without updating the source YAML files. However, it's convenient for quick rollbacks. The important precaution is to remember to update your source YAML files to reflect the changes made by the imperative command.

```bash
$ kubectl rollout undo deployment hello-deploy --to-revision=1
deployment.apps "hello-deploy" rolled back
```

Rollbacks follow the same logic and rules as updates/rollouts - they terminate Pods with the current image and replace them with Pods running the new image. In the case of a rollback, the 'new' image is actually an older one.

### ❓ Why might a rollback operation appear instantaneous but actually follow standard rollout rules, and how can you verify and track its progress?
Although a rollback operation might look instantaneous, it isn't. Rollbacks follow the same rules as rollouts, meaning they proceed through the same procedural steps of terminating old Pods and creating new ones. You can verify this and track the progress with the 'kubectl get deploy' command, which shows the current state of the Deployment during the rollback process.

### ❓ Analyze the following rollout status progression. What does each stage reveal about the deployment process, and why might the command have been interrupted with ^C?
The rollout status progression `$ kubectl rollout status deployment hello-deployWaiting for deployment "hello-deploy"... 6 out of 10 new replicas have been updated...Waiting for deployment "hello-deploy"... 7 out of 10 new replicas have been updated...Waiting for deployment "hello-deploy"... 8 out of 10 new replicas have been updated...Waiting for deployment "hello-deploy"... 1 old replicas are pending termination...Waiting for deployment "hello-deploy"... 9 of 10 updated replicas are available...^C` reveals:

- The deployment is performing a rolling update, gradually replacing old Pods with new ones
- It progresses through stages: 6→7→8 new replicas updated, showing controlled, incremental replacement
- One old replica remains pending termination while new ones are being brought online
- 9 out of 10 updated replicas become available, indicating most of the update is complete
- The command was interrupted with ^C (Ctrl+C) likely because the user decided to stop waiting for the final completion, as the deployment was taking significant time to complete the update process

### ❓ Describe the manual process for scaling Deployments using the declarative approach.
You can manually scale Deployments by editing the Deployment YAML and re-posting it to the cluster.

### ❓ What alternative to manual scaling does Kubernetes provide for Deployments, and what factor determines the scaling?
Kubernetes has autoscalers that automatically scale Deployments based on demand.

### ❓ Explain how rolling updates are implemented in Kubernetes Deployments, including the specific operations performed on Pods.
Rolling updates happen by deleting old Pods and replacing them with new ones in a controlled, organized manner.

