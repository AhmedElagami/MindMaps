# Ordered Deployment Management

## Content

### ❓ Explain the pattern observed in the StatefulSet rollout process as shown in the watch output and what it indicates about the controller's behavior.
The watch output shows the StatefulSet controller starting each replica sequentially with approximately 30-second intervals:

```bash
$ kubectl get sts --watch
NAME      READY   AGE
tkb-sts   0/3     14s
tkb-sts   1/3     30s
tkb-sts   2/3     60s
tkb-sts   3/3     90s
```

This pattern indicates that the StatefulSet controller starts each replica in turn and waits for them to be running and ready before starting the next replica, ensuring orderly and sequential deployment rather than parallel deployment.

