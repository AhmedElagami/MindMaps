# Topology Spread Constraints

## Content

- Evenly distribute Pods across zones or nodes.
- Use `topologySpreadConstraints` with `maxSkew` and `topologyKey`.

```yaml
spec:
  topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: kubernetes.io/hostname
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: web
```
