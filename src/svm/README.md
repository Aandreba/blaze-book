# Shared Virtual Memory
> All functionality related to SVM requires the `svm` features.

Blaze implements support for shared virtual memory (also known as SVM) through Rust's Allocator API.

```rust
pub struct Svm<C: Context = Global> {
    ctx: C,
    coarse: bool
}
```