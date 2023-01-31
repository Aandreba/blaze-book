# Slices

> **Note** slices require OpenCL 1.1 or higher.

Buffer slices act the same way as Rust [slices](https://doc.rust-lang.org/stable/core/primitive.slice.html), allowing access to a specified region of a buffer.

## Examples
```rust
use std::ops::Deref;
use blaze_rs::prelude::*;

#[global_context]
static CONTEXT : SimpleContext = SimpleContext::default();

fn main () -> Result<()> {
    let buffer = Buffer::new(&[1, 2, 3, 4, 5], MemAccess::default(), false)?;
    let slice = buffer.slice(..)?;

    assert_eq!(slice.deref(), &buffer);
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
    let slice = buffer.slice(None)?;
    let mut mut_slice = buffer.slice_mut(None)?; // compile error: cannot borrow `buffer` as mutable because it is also borrowed as immutable

    assert_eq!(slice, slice_mut);
    Ok(())
}
```

> Note that when maping mutably, the OpenCL mapping is done as a read-write mapping, not a write-only map.