# EndpointSlice Partitioning

## Content

### ❓ What determines when a Kubernetes Service will have more than one EndpointSlice and what is the threshold for this automatic partitioning?
A Kubernetes Service will have more than one EndpointSlice when it maps to more than 100 Pods. This 100-Pod threshold triggers automatic partitioning where Kubernetes creates additional EndpointSlice objects to handle the large number of endpoints. The system automatically manages this partitioning to ensure efficient endpoint management and scalability.

