# RAFT Consensus Algorithm

## Content

### ❓ What operational impact occurs when etcd enters read-only mode due to a split-brain, and how does the RAFT consensus algorithm prevent data corruption?
If a split-brain occurs, etcd goes into read-only mode preventing updates to the cluster. User applications will continue working, but Kubernetes won't be able to scale or update them. As with all distributed databases, consistency of writes is vital. For example, multiple writes from different sources to the same can cause corruption. etcd uses the RAFT consensus algorithm to prevent this from happening.

