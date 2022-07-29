# Contexts
A Blaze context is the owner of a single OpenCL context and one or more OpenCL command queues, all of them associated to the context.

It's task is to manage the distribution of command queues amongst the various _enqueue_ functions, maximizing performance by distributing the work amongst them.

The simplified signature of the `Context` trait is the following:
```rust
pub trait Context {
    fn queues (&self) -> &[RawCommandQueue];
    fn next_queue (&self) -> &RawCommandQueue;
}
```

The `queues` method returns a list with all the command queues owned by the context.\
The `next_queue` method returns the next command queue to be used in an _enqueue_ function.

> Note that all `Context` types must implement `Deref` with a target of `RawContext`.