# Labels for Selection

## Content

### ❓ What three types of resource limits should be configured to prevent essential system resources from being starved and potential DoS attacks?
You should configure appropriate limits for the following: Memory, CPU, and Storage. Limits like these can help prevent essential system resources from being starved, therefore preventing potential DoS.

### ❓ What is the high-level purpose of a ConfigMap in Kubernetes and how does it integrate with applications?
At a high level, a ConfigMap is an object for storing configuration data that you can easily inject into containers at run time. You can configure them to work with environment variables and volumes so that they work seamlessly with applications.

### ❓ What are the technical characteristics of ConfigMap keys and values, what types of content can ConfigMap values store, and what is their size limitation?
Keys are an arbitrary name that can include alphanumerics, dashes, dots, and underscores. Values can store anything, including full configuration files with multiple lines and carriage returns. They're limited to 1MiB (1,048,576 bytes) in size.

