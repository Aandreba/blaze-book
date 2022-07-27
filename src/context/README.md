# Context
A Blaze context is the owner of a single OpenCL context and one or more OpenCL command queues, all of them associated to the context.

It's task is to manage the distribution of command queues amongst the various _enqueue_ functions, maximizing performance by distributing the work amongst them.

The simplified signature of the `Context` trait is the following:
```rust
pub trait Context {
    fn queues (&self) -> &[RawCommandQueue];
    fn next_queue (&self) -> &RawCommandQueue;
}
```

Note that all `Context` types must implement `Deref` with a target of `RawContext`.