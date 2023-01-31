# Errors

Blaze provides three types to describe errors: `ErrorKind`, `ErrorCode` and `Error`.\
`ErrorType` is an enum that maps to the OpenCL error codes, whilst `Error` also contains an optional description and (in debug mode) a backtrace, and has athe following signature:

```rust,ignore
pub struct Error {
    pub ty: ErrorCode,
    pub desc: Option<Arc<dyn Display>>,
    #[cfg(debug_assertions)]
    pub backtrace: Arc<Backtrace>
}
```

The [`Backtrace`](https://doc.rust-lang.org/stable/std/backtrace/struct.Backtrace.html) is provided in debug mode to facilitate the debugging of errors, allowing to find their source more quickly.

## ErrorKind and ErrorCode
Most raw OpenCL errors will be converted to `ErrorKind` automatically, but there are instances where the given error code is not recognized as an `ErrorKind`. For such cases, `ErrorCode` exists.

```rust,ignore
pub enum ErrorCode {
    Kind (ErrorKind),
    Unknown (i32)
}
```