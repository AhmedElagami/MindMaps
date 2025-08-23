# DNS Resolution

## Content

### ❓ What is the mechanism by which a headless Service becomes the governing Service for a StatefulSet, what DNS records does it create for StatefulSet Pods, and what is the developer's responsibility regarding DNS querying?
A headless Service becomes the StatefulSet's governing Service by being listed in the StatefulSet under the `serviceName` field. This designation makes the headless Service "in charge" of the StatefulSet's DNS subdomain.

When a headless Service is combined with a StatefulSet in this way, the Service creates DNS SRV and DNS A records for every Pod matching the Service's label selector. Other Pods and applications can then query DNS to get the names and IPs of all the StatefulSet's Pods.

Developers must code their applications to query DNS like this. The applications need to be designed to take advantage of the way StatefulSets work by querying DNS for the SRV records to discover individual Pods and then send queries directly to those Pods instead of via a Service's ClusterIP.

