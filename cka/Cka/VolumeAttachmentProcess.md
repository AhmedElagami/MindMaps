# Volume Attachment Process

## Content

### ❓ What does the kubectl describe pod command output confirm about volume mounting for the newly created Pod?
The kubectl describe pod command output confirms that the newly created Pod automatically connected to the correct volume. The output shows:

```bash
$ kubectl describe pod tkb-sts-2 | grep ClaimName
ClaimName:  webroot-tkb-sts-2
```

This demonstrates that the StatefulSet controller successfully mounted the existing webroot-tkb-sts-2 PVC to the new tkb-sts-2 Pod, preserving the data association.

