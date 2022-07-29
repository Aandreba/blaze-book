# Asynchronous event

The `Event` trait ofers various ways to wait for the completion of an event.

One of the most interesting methods to await an event is with the `wait_async` method.
This method returns a Rust [`Future`](https://doc.rust-lang.org/stable/std/future/trait.Future.html) that resolves when the underlying event has completed.

```rust
use std::time::Duration;
use blaze::prelude::*;

#[global_context]
static CONTEXT : SimpleContext = SimpleContext::default();

#[tokio::main]
async fn main () -> Result<()> {
    const SIZE : usize = u16::MAX as usize;
    let values = vec![1234; SIZE];

    let mut buffer = Buffer::<i32>::new_uninit(SIZE, MemAccess::READ_WRITE, false)?;
    let flag = FlagEvent::new()?;

    let read = buffer.write_init(0, &values, &flag)?;
    tokio::spawn(async move {
        tokio::time::sleep(Duration::from_secs(2)).await;
        flag.complete(None)
    });

    let _ = read.wait_async()?.await;

    Ok(())
}
```

> Note that `wait_async` requires the `futures` feature.