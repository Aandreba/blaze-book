# Scopes

Inspired by Rust's [scopes](https://doc.rust-lang.org/stable/std/thread/fn.scope.html), Blaze scopes allow you to use events with non-`'static` lifetimes.

```rust
use blaze_rs::{buffer, prelude::*};

#[global_context]
static CTX : SimpleContext = SimpleContext::default();

let buffer: Buffer<i32> = buffer![1, 2, 3, 4, 5]?;

let [left, right]: [Vec<i32>; 2] = scope(|s| {
    let left = buffer.read(s, ..2, None)?;
    let right = buffer.read(s, 2.., None)?;
    return Event::join_all_sized_blocking([left, right]);
})?;

assert_eq!(left.as_slice(), &[1, 2]);
assert_eq!(right.as_slice(), &[3, 4, 5]);
# Ok::<_, Error>(())
```

## Asynchronous scopes

With the `scope_async` macro, you can create asynchronous scopes. These async scopes return a [`Future`](https://doc.rust-lang.org/stable/std/future/trait.Future.html) that completes when all the events spawned inside the scope have completed.

```rust
use blaze_rs::{buffer, scope_async, prelude::*};

# #[tokio::main] async fn main () -> Result<()> {
let buffer = buffer![1, 2, 3, 4, 5]?;

let (left, right) = scope_async!(|s| async {
    let left = buffer.read(s, ..2, None)?.join_async()?;
    let right = buffer.read(s, ..2, None)?.join_async()?;
    return tokio::try_join!(left, right);
}).await?;

assert_eq!(left, vec![1, 2]);
assert_eq!(right, vec![3, 4, 5]);
# Ok::<_, Error>(()) }
```

Unlike it's blocking counterpart, `scope_async` does **not** ensure that all events inside the future will be ran. Rather, if the future is dropped before completion, it's destructor will block the current thread until every **already-started** event has completed, and discarting the remaining uninitialized events.

```rust
use blaze_rs::{buffer, scope_async};
use futures::{task::*, future::*};

# #[tokio::main] async fn main () -> Result<()> {
let buffer = buffer![1, 2, 3, 4, 5]?;

let mut scope = Box::pin(scope_async!(|s| async {
    let left = buffer
        .read(s, ..2, None)?
        .inspect(|_| println!("Left done!"))
        .join_async()?
        .await;

    let right = buffer
        .read(s, ..2, None)?
        .inspect(|_| println!("Right done!"))
        .join_async()?
        .await;

    return Ok((left, right));
}));

let mut ctx = std::task::Context::from_waker(noop_waker_ref());
let _ = scope.poll_unpin(&mut ctx)?;
drop(scope); // prints "Left done!", doesn't print "Right done!"
# Ok::<_, Error>(()) }
```