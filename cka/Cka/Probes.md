# Probes

## Content

- Liveness, readiness, and startup probes check container health.
- Executed via HTTP, TCP, or command checks.

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
```
