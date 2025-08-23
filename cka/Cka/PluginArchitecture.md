# Plugin Architecture

## Content

### ❓ What are the key differences between in-tree storage plugins and CSI plugins in terms of development and release management?
The key differences between in-tree storage plugins and CSI plugins are:

**In-tree plugins (before CSI):**
- Developed as part of the main Kubernetes code tree (in-tree)
- Forced to be open source
- Tied plugin updates and bug fixes to the Kubernetes release cycle
- This was problematic for plugin developers as well as the Kubernetes maintainers

**CSI plugins:**
- Don't need to be open source
- Can be released whenever required
- Abstract the internal Kubernetes machinery and allow developers to develop out-of-tree
- Use an industry-standard storage interface for container orchestrators such as Kubernetes

