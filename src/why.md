# Why Blaze?

OpenCL is an open-source cross-platform API for high-performance parallel computing,
enabling the use of GPU's and other hardware accelerators to perform highly parallel computations.

Historically, this API has been used in C/C++ code, which has lead to the usual C and C++ problems (segfaults, memory leaks, etc).

Blaze fixes this by wrapping the OpenCL API in a Rust-friendly interface, and providing a simpler and safer, yet equally powerful, way to use it.