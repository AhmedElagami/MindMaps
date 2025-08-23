# Taints and Tolerations

## Content

- Taints repel Pods from nodes unless a matching toleration exists.
- Effects: `NoSchedule`, `PreferNoSchedule`, `NoExecute`.
- Tolerations let specific Pods schedule onto tainted nodes.

```yaml
spec:
  tolerations:
  - key: "gpu"
    operator: "Exists"
    effect: "NoSchedule"
```
