# Consumers

An event's `Consumer` is the responsable to perform the necessary underlying operations when the event has completed, with the ability to return a value.

```rust,ignore
pub trait Consumer<'a>: 'a {
    type Output;
    
    unsafe fn consume (self) -> Result<Self::Output>;
}
```