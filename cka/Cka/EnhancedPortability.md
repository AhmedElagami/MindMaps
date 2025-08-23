# Enhanced Portability

## Content

### ❓ Why is the statement 'containers are portable' considered a misconception, what two primary reasons are given for this belief, and what fundamental characteristic makes them not truly portable?
It's a common misconception that containers are portable. They're not! We only think containers are portable because they're smaller than VMs and easier to copy between hosts and registries. However, this isn't true portability. In fact, containers are architecture-dependent, meaning they're not portable.

### ❓ List two specific examples provided of architecture or platform dependencies that prevent container portability.
For example, you cannot run: an ARM container on an AMD processor or a Windows container on a Linux system.

### ❓ Despite build tools making it easier, what is the persistent limitation of each individual container image and what problem do organizations often face as a result?
However, each container still only works on one platform or architecture, and many organizations end up with image sprawl.

### ❓ How does WebAssembly fundamentally solve the portability problem that containers face and what is the only requirement for a system to run a pre-compiled Wasm binary?
It does this by implementing its own bytecode format that requires a runtime to execute. You build an app once as a Wasm binary, which you can then run on any system with a Wasm runtime.

### ❓ Besides standard data center and cloud architectures, where else can Wasm runtimes be found, and what additional size advantage do Wasm apps have over Linux containers for these environments?
Wasm runtimes even exist for exotic architectures that you find on IoT and edge devices. Speaking of IoT devices, Wasm apps are typically a lot smaller than Linux containers, meaning you can run them in resource-constrained environments, such as the edge and IoT, where you can't run containers.

