# Buffers

The `Buffer` type is a wrapper arround a `RawBuffer` that provides extra functionality and safety guarantees. It has the following signature:

```rust,ignore
pub struct Buffer<T: Copy, C: Context = Global> {
    inner: RawBuffer,
    ctx: C,
    phtm: PhantomData<T>
}
```

## Example

```rust
use std::ptr::NonNull;
use blaze_rs::{prelude::*, context::SimpleContext, buffer::BufferRange};

#[global_context]
static CONTEXT : SimpleContext = SimpleContext::default();

fn with_buffer () -> Result<()> {
    let values = [1, 2, 3, 4, 5];
    let buffer = Buffer::new(&values, MemAccess::READ_ONLY, false)?;

    let read: Vec<i32> = buffer.read_blocking(.., None)?;
    assert_eq!(values.as_slice(), read.as_slice());

    Ok(())
}

fn without_buffer () -> Result<()> {
    let values = [1, 2, 3, 4, 5];

    let buffer = RawBuffer::new(
        values.len() * core::mem::size_of::<i32>(), 
        MemFlags::new(MemAccess::READ_ONLY, HostPtr::COPY), 
        NonNull::new(values.as_ptr() as *mut _)
    )?;
    
    let mut read = Vec::<i32>::with_capacity(values.len());
    unsafe {
        let evt : RawEvent = buffer.read_to_ptr(BufferRange::from_parts::<i32>(0, 5)?, read.as_mut_ptr().cast(), None)?;
        let _ : () = evt.join_by_ref()?;
        read.set_len(values.len());
    }

    assert_eq!(values.as_slice(), read.as_slice());
    Ok(())
}
```