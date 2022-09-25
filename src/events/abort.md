# Abortable event

> **Note** Abortable events require OpenCL 1.1 or higher

Abortable events allow you to abort events before they're completed. When an abortable event is created, a new [flag event](flag) is created, alongside a host-side flag.

If the event completes before it's aborted, the flag event is marked with it's result and the host-side flag is marked as not-aborted. The abortable event will return `Ok(Some(_))` if it succedded, and `Err(_)` if it didn't.

If the event is aborted before completion, the flag event is marked without an error and the host-side flag is marked as aborted. The abortable event will return `Ok(None)`.

```rust
use blaze_rs::{event::{AbortHandle, consumer::{Noop, AbortableEvent}}, prelude::*};

#[global_context]
static CTX : SimpleContext = SimpleContext::default();

#[test]
fn aborted () -> Result<()> {
    let flag = FlagEvent::new()?;
    let (event, handle): (AbortableEvent<Noop>, AbortHandle) = flag
        .subscribe()
        .into_event()
        .abortable()?;
    
    assert_eq!(handle.try_abort(), Ok(true)); // Abort the event
    assert_eq!(event.join(), Ok(None));
    Ok(())
}

#[test]
fn not_aborted () -> Result<()> {
    let flag = FlagEvent::new()?;
    let (event, handle): (AbortableEvent<Noop>, AbortHandle) = flag
        .subscribe()
        .into_event()
        .abortable()?;
    
    assert_eq!(flag.try_mark(None), Ok(true)); // Complete the event
    assert_eq!(event.join(), Ok(Some(())));
    assert_eq!(handle.try_abort(), Ok(false));
    Ok(())
}
```