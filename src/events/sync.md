# Asynchronous event

Events can be joined asynchronously with the `join_async` method and the `EventWait` type.

```rust
use std::time::Duration;
use blaze_rs::{event::{EventWait, consumer::Noop}, prelude::*};

#[global_context]
static CONTEXT : SimpleContext = SimpleContext::default();

# #[tokio::main] async fn main () -> Result<()> {
let flag = FlagEvent::new()?;
let sub: EventWait<Noop> = flag
    .subscribe()
    .into_event()
    .join_async()?;

let handle = tokio::spawn(async move {
    tokio::time::sleep(Duration::from_secs(2)).await;
    flag.try_mark(None)?;
    return Ok();
});

tokio::try_join!(handle, sub)?;
# Ok(()) }
```

> Note that `join_async` requires the `futures` feature.