# Rectangular Buffer
> **Note**\
> - Rectangular buffers require OpenCL 1.1 or higher. 
> - Currently, only 2 dimensional rect buffers are implemented.

Rectangular buffers allow the use of buffers as 2D or 3D arrays, with the same safety guarantees provided by `Buffer`. They have the following signature:

```rust
pub struct BufferRect2D<T: Copy, C: Context = Global> {
    inner: Buffer<T, C>,
    width: NonZeroUsize,
    height: NonZeroUsize
}
```

Also, a host implementation of rectangular buffers exists, faciltating the use of rectangular buffers on the host.
```rust
pub struct Rect2D<T, A: Allocator = Global> {
    ptr: NonNull<T>,
    width: NonZeroUsize,
    height: NonZeroUsize,
    alloc: A
}
```

> Note that the `Global` here refers to Rust's [`Global`](https://doc.rust-lang.org/stable/std/alloc/struct.Global.html) allocator.

## Example
```rust

```