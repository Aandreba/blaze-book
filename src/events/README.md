# Events

Blaze events are similar in sytnax to Rust [`Future`](https://doc.rust-lang.org/stable/std/future/trait.Future.html). It is designed to be able to safely return a value after the underlying `RawEvent` has completed. The `Event` trait has the following signature:

```rust,ignore
pub trait Event {
    type Output;

    fn as_raw (&self) -> &RawEvent;
    fn consume (self, err: Option<Error>) -> Result<Self::Output>;
    
    // Provided methods
    ...
}
```