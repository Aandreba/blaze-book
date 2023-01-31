# Global Context

Inspired by Rust's Allocator syntax, `Global` is a [ZST](https://doc.rust-lang.org/nomicon/exotic-sizes.html#zero-sized-types-zsts) that will be treated as the default context for most operations requiring of an OpenCL context or command queue.

Like with the Allocator API, you can specify a global context with the `#[global_context]` macro.

```rust
use blaze_rs::prelude::*;

#[global_context]
static CONTEXT : SimpleContext = SimpleContext::default();

#[test]
fn with_global () -> Result<()> {
    // Initialize two buffers
    let buffer : Buffer<i32> = Buffer::new(&[1, 2, 3, 4, 5], MemAccess::READ_ONLY, false)?;
    let buffer2 : Buffer<i32> = Buffer::new(&[5, 4, 3, 2, 1], MemAccess::WRITE_ONLY, false)?;

    // Read the full contents of both buffers
    let read = buffer.read_blocking(.., None)?;
    let read2 = buffer2.read_blocking(.., Some(core::slice::from_ref(&read)))?;

    assert_eq!(read.as_slice(), &[5, 4, 3, 2, 1]);
    assert_eq!(read2.as_slice(), &[1, 2, 3, 4, 5]);
    Ok(())
}

#[test]
fn without_global () -> Result<()> {
    // Initialize a context.
    let ctx = SimpleContext::default()?;
    
    // Initialize two buffers
    let buffer : Buffer<i32, &SimpleContext> = Buffer::new_in(&ctx, &[1, 2, 3, 4, 5], MemAccess::READ_ONLY, false)?;
    let buffer2 : Buffer<i32, &SimpleContext> = Buffer::new_in(&ctx, &[5, 4, 3, 2, 1], MemAccess::WRITE_ONLY, false)?;

    // Read the full contents of both buffers
    let read = buffer.read_blocking(.., None)?;
    let read2 = buffer2.read_blocking(.., Some(core::slice::from_ref(&read)))?;

    assert_eq!(read.as_slice(), &[5, 4, 3, 2, 1]);
    assert_eq!(read2.as_slice(), &[1, 2, 3, 4, 5]);
    Ok(())
}
```

> Note that unlike with the `Allocator` API, no default global context is set, so you'll need to specify one explicitly if you want to use it.