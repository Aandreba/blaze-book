# Coarse v. Fine grained

OpenCL's SVM implementation consist's of two diferent variant: **Coarse grained** and **fine grained** memory.
With coarse grained memory, the SVM user must indicate to OpenCL the synchronization points between host and device code, whilst with fine-grained memory, this process is done automatically by OpenCL.

But with Blaze, you don't have to think about this implementation details, since it will automatically detect the context's capabilities, and use fine-grained allocations whenever possible. In situiations where fine-grained allocations are unavailable, Blaze will indicate the synchronization points to OpenCL automatically whenever a pointer is passed as a kernel argument.

> Note that coarse-grained synchronization points are only set automatically for kernels generated with the `#[blaze]` macro