# Raw types

In Blaze, the _raw_ types are used to represent an underlying OpenCL objects plainly. They offer no extra functionality and have the following signature.

```rust
#[repr(transparent)]
pub struct RawType (NonNull<c_void>);
```

Raw types implement [`Drop`](https://doc.rust-lang.org/stable/std/ops/trait.Drop.html) and, since they're reference counted by OpenCL itself, they also implement [`Clone`](https://doc.rust-lang.org/stable/std/clone/trait.Clone.html).

Currently, this are the existing raw types:

| Name              | OpenCL type            | OpenCL Version |
| ----------------- | ---------------------- | -------------- |
| `RawPlatform`     | `cl_platform_id`       | All            |
| `RawDevice`       | `cl_device_id`         | All            |
| `RawContext`      | `cl_context`           | All            |
| `RawCommandQueue` | `cl_command_queue`     | All            |
| `RawProgram`      | `cl_program`           | All            |
| `RawKernel`       | `cl_kernel`            | All            |
| `RawEvent`        | `cl_event`             | All            |
| `RawMemObject`    | `cl_mem`               | All            |
| `RawBuffer`       | `cl_mem`               | All            |
| `RawPipe`         | `cl_mem`               | 2.0 or higher  |

> Note that since _raw_ types are transparent wrappers of `NonNull<c_void>`, `RawType` and `Option<RawType>` have the same size, alongside other optimizations of [`NonNull`](https://doc.rust-lang.org/stable/std/ptr/struct.NonNull.html) performed by the compiler.