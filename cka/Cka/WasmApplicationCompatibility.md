# Wasm Application Compatibility

## Content

### ❓ What significant milestones did WebAssembly achieve by 2025 according to the Wasm Primer section, and explain Solomon Hykes' famous tweet about WebAssembly and its significance for Docker?
By 2025, WebAssembly achieved several significant milestones:
- It became an official W3C standard
- It's in all the major browsers
- It's the go-to solution for web games and web apps that require high performance without sacrificing security and portability

Solomon Hykes, Docker Founder, famously tweeted: 
"If Wasm+WASI existed in 2008, we wouldn't have needed to create Docker. That's how important it is. WebAssembly on the server is the future of computing. A standardized system interface was the missing link. Let's hope WASI is up to the task!"

He quickly followed up with another tweet saying he expected a future where Linux containers and Wasm containers work side-by-side and Docker works with them all. Solomon's predicted future is already here - Docker has excellent support for Wasm, and you can run Linux containers side-by-side with Wasm containers in the same Kubernetes Pod.

### ❓ What is WASI and why is it crucial for WebAssembly outside the browser?
WASI (WebAssembly System Interface) is the WebAssembly System Interface and allows sandboxed Wasm apps to securely access external services such as:
- key-value stores
- networks  
- the host environment
- and more

WASI is absolutely vital to the success of Wasm outside the browser. At the time of writing, WASI Preview 2 is released and is a huge step forward. You'll sometimes see WASI Preview 2 written as WASI 0.2 and wasip2.

### ❓ List the four key advantages of Wasm apps over traditional Linux containers as summarized in the recap.
Wasm apps are smaller, faster, more portable, and more secure than traditional Linux containers.

### ❓ What are five specific scenarios or applications for which Wasm is currently considered a great choice?
Currently, Wasm is a great choice for: event handlers, anything needing super-fast startup times, IoT, edge computing, and building extensions and plugins.

### ❓ What does the following full command sequence do to prepare the Git repository for the hands-on exercise and explain the purpose and outcome of the rustup target add wasm32-wasip1 command?
The Git repository setup commands prepare the local environment for the hands-on exercise by cloning the book's GitHub repository and checking out the specific branch needed for the exercises:

```bash
$ git clone https://github.com/nigelpoulton/TKB.git
<Snip>

$ cd TKB

$ git fetch origin

$ git checkout -b 2025 origin/2025
```

This sequence: 1) Clones the TKB repository from GitHub, 2) Changes into the TKB directory, 3) Fetches the latest updates from the origin remote, and 4) Creates and switches to a new local branch called '2025' that tracks the remote 'origin/2025' branch.

The rustup target add wasm32-wasip1 command installs the necessary Rust compilation target for WebAssembly:

```bash
$ rustup target add wasm32-wasip1
info: downloading component 'rust-std' for 'wasm32-wasip1'
info: installing component 'rust-std' for 'wasm32-wasip1'
```

This command adds the wasm32-wasip1 target to the Rust toolchain, which allows Rust to compile code to WebAssembly binaries using the WASI (WebAssembly System Interface) preview1 specification. The output shows it downloads and installs the standard library component for this target architecture, enabling Rust developers to compile their applications to portable Wasm binaries that can run in Wasm runtimes.

### ❓ What is the purpose of installing the wasm32-wasip1 target in Rust for Wasm development and what command sequence is used to add it?
The purpose of installing the wasm32-wasip1 target in Rust is so that Rust can compile code to Wasm binaries. This target enables Rust to generate WebAssembly binaries that can run in WebAssembly runtimes.

The command to install this target is:

```bash
$ rustup target add wasm32-wasip1
info: downloading component 'rust-std' for 'wasm32-wasip1'
info: installing component 'rust-std' for 'wasm32-wasip1'
```

### ❓ What is Spin and what components does it include for working with Wasm applications, and how do you verify its installation?
Spin is a popular Wasm framework that includes a Wasm runtime and tools to build and work with Wasm apps. It provides both the runtime environment for executing WebAssembly applications and the development tools needed to create and manage them.

To verify the Spin installation, you run the version check command:

```bash
$ spin --version
spin 3.1.2 (3d37bd8 2025-01-13)
```

This command confirms that Spin is installed and provides the version information (3.1.2), the commit hash (3d37bd8), and the build date (2025-01-13).

### ❓ What is the purpose of the 'spin new' command with the http-rust template and what does the full command sequence with prompts configure?
The 'spin new' command with the http-rust template scaffolds a simple Spin app that responds to web requests. It creates the basic structure and configuration for a Rust-based HTTP application that can be compiled to WebAssembly.

The full command sequence is:

```bash
$ spin new tkb-wasm -t http-rust
Description []: My first Wasm app
HTTP path [/...]: /tkb
```

This command:
- Creates a new Spin app called "tkb-wasm" using the http-rust template
- Sets the description to "My first Wasm app" (prompt response)
- Configures the HTTP path to "/tkb" (prompt response)
- TKB is short for The Kubernetes Book

### ❓ What does the 'spin build' command do behind the scenes and what toolchain does it utilize, and what compilation process occurs?
The 'spin build' command compiles the app as a Wasm binary. Behind the scenes, spin runs a more complicated cargo command from the Rust toolchain. Specifically, it executes:

```bash
$ spin build
Building component tkb-wasm with `cargo build --target wasm32-wasip1 --release`
    Updating crates.io index
    <Snip>
Finished building all Spin components
```

The build process:
- Uses the Rust cargo toolchain with the wasm32-wasip1 target
- Builds in release mode for optimized performance
- Updates the crates.io index to ensure dependencies are current
- Compiles the Rust code to a WebAssembly binary
- Places the compiled binary in the target/wasm32-wasip1/release/ directory

### ❓ What is the purpose of packaging a Wasm app as an OCI container image for Kubernetes deployment and what is the significance of using 'FROM scratch' in the Dockerfile?
The purpose of packaging a Wasm app as an OCI container image is to share it on an OCI registry and run it on Kubernetes. This allows the Wasm application to be distributed and deployed using standard container tools and platforms.

The significance of using 'FROM scratch' in the Dockerfile is that it tells Docker to package your Wasm app inside an empty scratch image instead of a typical Linux base image. This keeps the image small and helps build a minimal container at runtime. Wasm apps don't need a container with a Linux filesystem and other constructs, but platforms such as Docker and Kubernetes use tools that expect to work with basic container constructs. Packaging your Wasm app in a scratch image accomplishes this. At runtime, the Wasm app and Wasm runtime will execute inside a minimal container that is basically just namespaces and cgroups (no filesystem etc.).

### ❓ Explain the full Dockerfile structure and what each COPY instruction accomplishes for packaging the Wasm application.
The Dockerfile structure for packaging the Wasm application is:

```bash
FROM scratch
COPY /target/wasm32-wasip1/release/tkb_wasm.wasm .
COPY spin.toml .
```

Each instruction accomplishes:

1. `FROM scratch` - Uses an empty base image instead of a Linux base image, creating a minimal container

2. `COPY /target/wasm32-wasip1/release/tkb_wasm.wasm .` - Copies the compiled Wasm binary from the local build directory into the container's root folder

3. `COPY spin.toml .` - Copies the Spin configuration file into the container's root folder, which tells the Spin runtime how to execute the Wasm app

The Dockerfile packages both the Wasm binary and its configuration file into a minimal OCI container image that can be deployed to Kubernetes.

### ❓ What is WebAssembly (Wasm) and how does it function as a compilation target compared to traditional compilation targets?
WebAssembly (Wasm) is a binary instruction set that programming languages use as a compilation target. Instead of compiling to traditional targets like Linux on ARM or other platform-specific architectures, you compile your code to Wasm. This means Wasm serves as an intermediate representation that can be executed by any Wasm runtime, providing platform independence.

### ❓ What are the four key advantages of Wasm applications over traditional Linux containers, and what limitation do Wasm apps currently have?
Wasm applications have four key advantages over traditional Linux containers:

1. **Smaller**: Compiled Wasm apps are tiny binaries
2. **Faster**: Wasm apps execute more quickly
3. **More portable**: They can run anywhere with a Wasm runtime
4. **More secure**: Wasm provides enhanced security features

However, at the time of writing, Wasm apps have one significant limitation: they cannot do everything that Linux containers can, meaning there are certain capabilities and functionalities that are still exclusive to traditional containers.

### ❓ Describe the high-level process for taking Wasm applications from development to deployment on Kubernetes, including the tools and registries involved.
The high-level process for taking Wasm applications from development to deployment on Kubernetes involves these steps:

1. **Write applications** in existing programming languages
2. **Compile them as Wasm binaries**
3. **Use tools** (though specific tool names are not provided in the content) to build them into OCI images
4. **Push them to OCI registries** (standard container registries that support the OCI format)
5. **Wrap them in Kubernetes Pods** and run them on Kubernetes just like regular containers

This process allows Wasm applications to leverage the existing Kubernetes ecosystem and container infrastructure.

### ❓ What two specific resources are recommended for further information about Wasm implementation on Kubernetes?
For further information about Wasm implementation on Kubernetes, two specific resources are recommended:

1. **The SpinKube project**: A project that facilitates running Wasm applications on Kubernetes
2. **The containerd shim lifecycle community proposal**: Documentation or specification related to how containerd shims (including Wasm shims) should be implemented and managed

These resources provide deeper technical details and implementation guidance for working with Wasm on Kubernetes platforms.

