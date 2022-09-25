# Flag events

> **Note** Flag events require OpenCL 1.1 or higher

Flag events allow the creation of events that complete whenever the user marks them.
Flag events are marked via the `try_mark` method, which returns `true` if the event was successfully marked, and `false` if the event was already marked.

```rust
use blaze_rs::prelude::*;

#[global_context]
static CTX : SimpleContext = SimpleContext::default();

let flag = FlagEvent::new()?;
assert_eq!(flag.status(), Ok(EventStatus::Submitted));
assert_eq!(flag.try_mark(None), Ok(true));
assert_eq!(flag.status(), Ok(EventStatus::Complete));
assert_eq!(flag.try_mark(None), Ok(false));

# Ok::<_, Error>()
```