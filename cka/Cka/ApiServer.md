# API Server

## Content

### ❓ Describe the complete process that occurs when deploying an application through the API server, from YAML submission to container scheduling.
When deploying or updating an app through the API server, the process follows these steps:

1. **Describe the application**: Describe the application in a YAML configuration file that specifies which images to use, how many replicas, which network ports, and more.

2. **Post to API server**: Post the configuration file to the API server where it's authenticated and authorized.

3. **Authentication and authorization**: The request is authenticated and authorized before being processed.

4. **Persist to cluster store**: The application definition is persisted in the cluster store as a record of intent.

5. **Controller reconciliation**: A controller notices the observed state doesn't match the new desired state and begins reconciliation, which involves:
   - Scheduling new Pods
   - Pulling images
   - Starting containers
   - Attaching them to networks
   - Starting application processes

6. **Scheduler assignment**: The scheduler watches the API server for new work tasks and assigns them to healthy worker nodes based on capability checks, filtering, and ranking algorithms.

7. **Kubelet execution**: The kubelet on worker nodes watches for new tasks, instructs the runtime to execute them, and reports task status back to the API server.

