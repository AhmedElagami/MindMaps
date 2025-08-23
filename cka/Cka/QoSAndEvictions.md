# QoS and Evictions

## Content

- Pods fall into Guaranteed, Burstable, or BestEffort classes based on requests and limits.
- Under resource pressure, kubelet evicts BestEffort first, then Burstable.
- Eviction signals include memoryPressure and diskPressure.
