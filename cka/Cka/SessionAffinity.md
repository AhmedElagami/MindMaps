# Session Affinity

## Content

### ❓ Explain how Session Affinity controls session stickiness in Kubernetes Services, including the default behavior and when to use the ClientIP option.
Session Affinity allows you to control session stickiness — whether or not client connections always go to the same Pod. The default is None and forwards multiple connections from the same client to different Pods. You should try the ClientIP option if your app stores state in Pods and requires session stickiness. However, this is an anti-pattern as microservices apps should be designed for process disposability where clients can connect to any Pod.

