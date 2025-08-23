# Deprecation Detection

## Content

### ❓ What is the 9-month window requirement for beta resources and what support period does Kubernetes provide for deprecated GA resources, including what condition must be met before deprecation?
Kubernetes has specific time-based commitments for beta and GA resources:

**Beta Resources:** Resources in beta have a 9-month window to either release a newer beta version or graduate to GA. This is to prevent resources from stagnating in beta (for example, the Ingress resource remained in beta for over 15 Kubernetes releases!).

**GA Resources:** When deprecated, Kubernetes continues to serve and support GA resources for 12 months or three releases, whichever is longest. After this period they are removed. However, Kubernetes will only deprecate an existing stable resource after a newer stable version is available (for example, it will only deprecate a v1 resource if the v2 of the same resource is already released).

### ❓ What condition must be met before Kubernetes will deprecate an existing stable resource and provide a specific example of when Kubernetes would deprecate a v1 resource?
Kubernetes will only deprecate an existing stable resource after a newer stable version is available. For example, it will only deprecate a v1 resource if the v2 of the same resource is already released.

### ❓ What are the three actions that recent versions of Kubernetes take when you deploy a deprecated resource?
Recent versions of Kubernetes do three things when you deploy a deprecated resource:

1. Return a deprecation warning message on the CLI
2. Add an annotation to the audit record for the request
3. Set a gauge metric

### ❓ How does Kubernetes provide immediate feedback when a deprecated API is used, and what type of annotation does Kubernetes add to indicate deprecated API usage in audit records, and what type of metric does Kubernetes set when deprecated resources are used?
Kubernetes provides immediate feedback when a deprecated API is used by returning deprecation warning messages on the CLI. For audit records, Kubernetes adds an annotation to the audit record for the request. For metrics, Kubernetes sets a gauge metric when deprecated resources are used.

### ❓ Besides CLI warnings, what two other mechanisms help identify deprecated API usage during Kubernetes upgrade planning?
Besides CLI warnings, the other two mechanisms that help identify deprecated API usage during Kubernetes upgrade planning are:

1. Audit logs - Kubernetes adds annotations to audit records for requests using deprecated APIs
2. Process logs - Kubernetes sets gauge metrics that can be queried

These last two can be useful when planning Kubernetes upgrades as they help you to know if you're using deprecated resources.

