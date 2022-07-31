# Shared Virtual Memory
> All functionality related to SVM requires the `svm` features.

Blaze implements support for shared virtual memory (also known as SVM) through Rust's Allocator API.

SVM allows the host and device portions of an OpenCL application to seamlessly share pointers and complex pointer-containing data-structures. Moreover, as described in this article, SVM is more than just about shared address space. It also defines memory model consistency guarantees for SVM allocations. This enables the host and the kernel sides to interact with each other using atomics for synchronization, like two distinct cores in a CPU.

> Intel has a great article about SVM [here](https://www.intel.com/content/www/us/en/developer/articles/technical/opencl-20-shared-virtual-memory-overview.html)