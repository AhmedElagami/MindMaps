# ReplicaSet Naming Convention

## Content

### ❓ How is the ReplicaSet name generated from the Deployment name, what does it represent, and what specific part of the Deployment manifest is used to generate the crypto-hash?
The ReplicaSet name matches the Deployment's name with a hash added to the end. This hash is a crypto-hash of the Pod template section of the Deployment manifest (everything below the template section). The hash represents the specific configuration of the Pod template, and making changes to the Pod template section initiates a rollout and creates a new ReplicaSet with a hash of the updated Pod template.

