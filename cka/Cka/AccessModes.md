# Access Modes

## Content

### ❓ What are the three volume access modes supported by Kubernetes and what are the key differences between them?
Kubernetes supports three volume access modes:

**RWO (ReadWriteOnce)** - Lets a single PVC bind to a volume in read-write (R/W) mode. Attempts to bind it from multiple PVCs will fail.

**RWM (ReadWriteMany)** - Lets multiple PVCs bind to a volume in read-write (R/W) mode. File and object storage usually support this mode, whereas block storage usually doesn't.

**ROM (ReadOnlyMany)** - Allows multiple PVCs to bind to a volume in read-only (R/O) mode.

It's also important to know that a PV can only be opened in one mode. For example, you cannot bind a single PV to one PVC in ROM mode and another PVC in RWM mode.

### ❓ Explain the specific binding limitations and use cases for RWO (ReadWriteOnce) access mode, what storage types typically support RWM (ReadWriteMany) mode and which typically do not, and describe the fundamental limitation regarding how a single PV can be accessed by multiple PVCs with different access modes.
**RWO (ReadWriteOnce) Binding Limitations:** RWO lets a single PVC bind to a volume in read-write (R/W) mode. Attempts to bind it from multiple PVCs will fail. This mode is suitable for single-pod applications that need exclusive read-write access to storage.

**RWM Storage Type Compatibility:** RWM lets multiple PVCs bind to a volume in read-write (R/W) mode. File and object storage usually support this mode, whereas block storage usually doesn't. This makes RWM ideal for shared file systems and collaborative applications.

**PV Access Mode Restriction:** A fundamental limitation is that a PV can only be opened in one mode. You cannot bind a single PV to one PVC in ROM mode and another PVC in RWM mode. Once a PV is bound with a specific access mode, all subsequent bindings must use the same mode.

