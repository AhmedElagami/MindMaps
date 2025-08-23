# Kubectl Fundamentals

## Contexts and Namespaces
- `kubectl config use-context`
- `kubectl config set-context --namespace`

## Resource Management
- `kubectl get`, `kubectl describe`
- `kubectl apply --dry-run=server -f file.yml`

## Interaction Commands
- `kubectl exec`, `kubectl cp`, `kubectl port-forward`

## Labels and Annotations
- `kubectl label`, `kubectl annotate`

## Output Formatting
- `-o wide`, `-o yaml`, `-o jsonpath='{.status.podIP}'`

## Imperative Commands
- `kubectl create`, `kubectl run`

## Programming Models
Kubernetes prefers declarative configuration (`kubectl apply`) but supports imperative commands for quick changes.
