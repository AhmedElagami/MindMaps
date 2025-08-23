# Pod Disruption Budgets

## Content

- Define the minimum number of Pods that must remain available during voluntary disruptions.
- `maxUnavailable` or `minAvailable` fields control disruption limits.

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      app: web
```
