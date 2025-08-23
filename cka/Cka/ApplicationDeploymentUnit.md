# Application Deployment Unit

## Content

### ❓ What is the purpose of the jump Pod in the service discovery example configuration?
The jump Pod is only deployed to the dev Namespace and serves as a testing tool for demonstrating service discovery. It allows administrators to execute commands and test connectivity to services within the same namespace and across different namespaces.

### ❓ What additional standalone pod exists in the dev namespace beyond the enterprise application components?
The dev Namespace also has a standalone Pod called 'jump' beyond the enterprise application components.

### ❓ Explain the step-by-step process of establishing an interactive session with the jump pod's container in the dev namespace using the provided kubectl exec command.
To establish an interactive session with the jump pod's container in the dev namespace, you use the following kubectl exec command:

```bash
$ kubectl exec -it jump --namespace dev -- bash 
root@jump:/#
```

This command:
1. Uses `kubectl exec` to execute a command in a container
2. The `-it` flags create an interactive terminal session
3. Specifies the pod name 'jump'
4. Uses `--namespace dev` to target the dev namespace
5. Executes the `bash` command to start a bash shell
6. Your terminal prompt changes to `root@jump:/#` indicating you're attached to the container

### ❓ What command sequence is used to install the curl utility in the container, and why is this utility necessary for the service discovery testing?
The curl utility is installed using the following command sequence:

```bash
# apt-get update && apt-get install curl -y
<snip>
```

Curl is necessary for service discovery testing because it allows you to make HTTP requests to services to verify connectivity and test that service discovery is working correctly. It's used to connect to the ent Service on port 8080 to confirm that the connection reaches the correct namespace instance.

