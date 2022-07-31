# Atomics
> Note that SVM atomic are only available when the context devices support them, and can onloy be used in fine grained allocations.

OpenCL supports the use of atomics through SVM pointers. However, the following would not result in a correct SVM atomic implementation.

```rust
use blaze_rs::{prelude::*, svm::*};
use std::sync::atomic::AtomicU32;

#[global_context]
static CONTEXT : SimpleContext = SimpleContext::default();

#[blaze(BadAtomic)]
#[link = ...]
extern "C" {
    fn use_atomic (value: *const AtomicU32);
}

fn main () -> Result<()> {
    let program = BadAtomic::new(None)?;
    let pointer = SvmBox::new_in(AtomicU32::default(), Svm::try_default()?);

    let result = unsafe {
        program.use_atomic(&pointer)
    };

    Ok(())
}
```

This implementation is incorrect because, when using atomics, OpenCL must be notified that the SVM allocation needs support for atomics. The correct implementation would be the following.

```rust
use blaze_rs::{prelude::*, svm::*};
use std::sync::atomic::AtomicU32;

#[global_context]
static CONTEXT : SimpleContext = SimpleContext::default();

#[blaze(GoodAtomic)]
#[link = ...]
extern "C" {
    fn use_atomic (value: *const AtomicU32);
}

fn main () -> Result<()> {
    let program = GoodAtomic::new(None)?;
    let pointer = SvmAtomicU32::new(&[0]);

    let result = unsafe {
        program.use_atomic(&pointer)
    };

    Ok(())
}
```

## Currently supported atomics
| Name            | OpenCL type        | Rust atomic   | OpenCL features                                                                     |
| -------------   | ------------------ | ------------- | ----------------------------------------------------------------------------------- |
| SvmAtomicI32    | `atomic_int`       | `AtomicI32`   | None                                                                                |
| SvmAtomicU32    | `atomic_uint`      | `AtomicU32`   | None                                                                                |
| SvmAtomicI64    | `atomic_long`      | `AtomicI32`   | `cl_khr_int64_base_atomics` and `cl_khr_int64_extended_atomics`                     |
| SvmAtomicU64    | `atomic_ulong`     | `AtomicU32`   | `cl_khr_int64_base_atomics` and `cl_khr_int64_extended_atomics`                     |
| SvmAtomicIsize  | `atomic_ptrdiff_t` | `AtomicIsize` | `cl_khr_int64_base_atomics` and `cl_khr_int64_extended_atomics` (on 64-bit devices) |
| SvmAtomicUsize  | `atomic_size_t`    | `AtomicUsize` | `cl_khr_int64_base_atomics` and `cl_khr_int64_extended_atomics` (on 64-bit devices) |