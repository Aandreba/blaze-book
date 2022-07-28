# Buffer

The `Buffer` type is a wrapper arround a `RawBuffer` that provides extra functionality and safety guarantees. It has the following signature:

```rust
pub struct Buffer<T: Copy, C: Context = Global> {
    inner: RawBuffer,
    ctx: C,
    phtm: PhantomData<T>
}
```

## Example

```rust
use std::ptr::NonNull;
use blaze::{prelude::*, context::SimpleContext};

#[global_context]
static CONTEXT : SimpleContext = SimpleContext::default();

fn with_buffer () -> Result<()> {
    let values = [1, 2, 3, 4, 5];
    let buffer = Buffer::new(&values, MemAccess::READ_ONLY, false)?;
    let evt : ReadBuffer<i32, _> = buffer.read_all(EMPTY)?;
    
    let read : Vec<i32> = evt.wait()?;
    assert_eq!(values.as_slice(), read.as_slice());
    Ok(())
}

fn without_buffer () -> Result<()> {
    let values = [1, 2, 3, 4, 5];
    let buffer = RawBuffer::new(
        values.len() * core::mem::size_of::<i32>(), 
        MemFlags::new(MemAccess::READ_ONLY, HostPtr::COPY), 
        NonNull::new(values.as_ptr() as *mut i32)
    )?;
    
    let mut read = Vec::<i32>::with_capacity(values.len());
    unsafe {
        let evt : RawEvent = buffer.read_to_ptr(.., read.as_mut_ptr(), EMPTY)?;
        let _ : () = evt.wait()?;
        read.set_len(values.len())
    }

    assert_eq!(values.as_slice(), read.as_slice());
    Ok(())
}
```