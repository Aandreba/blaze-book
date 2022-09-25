# Mapping

OpenCL provides a feature on buffers (and other memory objects) named _mapping_. With this feature, a region of device memory is mapped to hast memory, where it can be more efficiently accessed.
Blaze offers support for this feature through the use of the `MapBufferGuard` and `MapBufferMutGuard`, which act simillarly to a [`RwLockReadGuard`](https://doc.rust-lang.org/stable/std/sync/struct.RwLockReadGuard.html) and a [`RwLockWriteGuard`](https://doc.rust-lang.org/stable/std/sync/struct.RwLockWriteGuard.html).

## Examples
```rust
use std::ops::Deref;
use blaze_rs::prelude::*;

#[global_context]
static CONTEXT : SimpleContext = SimpleContext::default();

fn main () -> Result<()> {
    let buffer = Buffer::new(&[1, 2, 3, 4, 5], MemAccess::default(), false)?;
    let map = buffer.map_blocking(.., None)?;

    assert_eq!(map.deref(), &[1, 2, 3, 4, 5]);
    Ok(())
}
```

```rust,compile_fail
use std::ops::Deref;
use blaze_rs::prelude::*;

#[global_context]
static CONTEXT : SimpleContext = SimpleContext::default();

fn main () -> Result<()> {
    let mut buffer = Buffer::new(&[1, 2, 3, 4, 5], MemAccess::default(), false)?;
    let map = buffer.map_blocking(.., None)?;
    let mut mut_map = buffer.map_mut_blocking(.., None)?; // compile error: cannot borrow `buffer` as mutable because it is also borrowed as immutable

    assert_eq!(map.deref(), &[1, 2, 3, 4, 5]);
    Ok(())
}
```

> Note that when maping mutably, the OpenCL mapping is done as a read-write mapping, not a write-only map.