# Faster Performance

## Content

### ❓ Compare the typical startup times for VMs, containers, and Wasm applications as outlined in the performance section and why are Wasm cold starts described as not feeling like traditional cold starts?
As a general rule: VMs take minutes to start, containers take seconds, but Wasm gets us into the exciting world of sub-second start times. So much so that Wasm cold starts are so fast they don't feel like cold starts. For example, Wasm apps commonly start in around ten milliseconds or less.

### ❓ What are two example use cases where Wasm's fast startup performance is particularly beneficial?
For example, Wasm is great for event-driven architectures like serverless functions. It also makes things like true scale-to-zero a possibility.

