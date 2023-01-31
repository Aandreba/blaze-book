# Events

Blaze events are considered a mixture of a Rust [`Future`](https://doc.rust-lang.org/stable/std/future/trait.Future.html) and [`JoinHandle`](https://doc.rust-lang.org/stable/std/thread/struct.JoinHandle.html). Their signature is the following:

```rust,ignore
use std::sync::mpsc::Sender;

pub struct Event<C> {
    inner: RawEvent,
    consumer: C,
    #[cfg(not(feature = "cl1_1"))]
    send: Sender<EventCallback>,
    #[cfg(feature = "cl1_1")]
    send: PhantomData<Sender<()>>,
}
```

Blaze events contain their underlying `RawEvent` alongside a `Consumer`.