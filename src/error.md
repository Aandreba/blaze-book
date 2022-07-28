# Error

Blaze provides two types to describe errors: `ErrorType` and `Error`.\
`ErrorType` is an enum that maps to the OpenCL error codes, whilst `Error` also contains an optional description and (in debug mode) a backtrace, and has athe following signature:

```rust
pub struct Error {
    pub ty: ErrorType,
    pub desc: Arc<str>,
    #[cfg(debug_assertions)]
    pub backtrace: Arc<Backtrace>
}
```

The [`Backtrace`](https://doc.rust-lang.org/stable/std/backtrace/struct.Backtrace.html) is provided in debug mode to facilitate the debugging of errors, allowing to find their source more quickly.

When the description is empty, a static empty `Arc` is cloned and put in it's place.