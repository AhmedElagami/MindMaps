# Volume Reattachment

## Content

### ❓ What does the grep ClaimName command output reveal about the relationship between the replacement Pod and its original storage?
The grep ClaimName command output confirms that the replacement Pod is connected to the same original PVC:

```bash
$ kubectl describe pod tkb-sts-0 | grep ClaimName
    ClaimName:  webroot-tkb-sts-0
```

This reveals that the new Pod (tkb-sts-0) has been connected to the original webroot-tkb-sts-0 PVC, demonstrating that StatefulSets maintain persistent storage relationships even when Pods are replaced. The replacement Pod retains the same storage claim as the original failed Pod.

