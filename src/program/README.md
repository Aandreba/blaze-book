# Program

To ease the safe use of OpenCL programs and kernels, Blaze provides the `#[blaze]` macro. The blaze macro will turn pseudo-normal Rust [_extern_](https://doc.rust-lang.org/stable/std/keyword.extern.html) syntax into a struct that will hold a program and it's various kernels, providing a safe API to call the kernels.

## Example

```rust,ignore
use blaze_rs::prelude::*;
use core::mem::*;

#[blaze(MatrixOps)]
#[link = include_str!("matrixops.cl")]
extern "C" {
    #[link_name = "mul"]
    fn matrix_mul (k: u32, lhs: *const f32, rhs: *const f32, out: *mut MaybeUninit<f32>);
}
```

### Expands to
```rust,ignore
use blaze_rs::prelude::*;
use core::mem::*;

struct MatrixOps<C: ::blaze::context::Context = ::blaze::context::Global> {
    __blaze_inner__: ::blaze::core::RawProgram,
    __blaze_ctx__: C,
    matrix_mul: ::std::sync::Mutex<::blaze::core::RawKernel>,
}

impl MatrixOps<::blaze::context::Global> {
    #[inline(always)]
    fn new<'a>(options: impl Into<Option<&'a str>>) -> ::blaze::core::Result<Self> {
        Self::new_in(::blaze::context::Global, options)
    }
}

impl<C: ::blaze::context::Context> MatrixOps<C> {
    fn new_in<'a>(ctx: C, options: impl Into<Option<&'a str>>) -> ::blaze::core::Result<Self> {
        let __blaze_ctx__ = ctx;
        let (__blaze_inner__, __blaze_kernels__) =
            ::blaze::core::RawProgram::from_source_in(&__blaze_ctx__, include_str!("matrixops.cl"), options)?;
        let mut matrix_mul = None;
        for __blaze_kernel__ in __blaze_kernels__.into_iter() {
            match __blaze_kernel__.name()?.as_str() {
                "mul" => matrix_mul = unsafe { Some(__blaze_kernel__.clone()) },
                __other => {
                    return Err(::blaze::core::Error::new(
                        ::blaze::core::ErrorType::InvalidKernel,
                        {
                            let res = ::alloc::fmt::format(::core::fmt::Arguments::new_v1(
                                &["unknown kernel \'", "\'"],
                                &[::core::fmt::ArgumentV1::new_display(&__other)],
                            ));
                            res
                        },
                    ))
                }
            }
        }
        let matrix_mul = match matrix_mul {
            Some(__x) => ::std::sync::Mutex::new(__x),
            None => {
                return Err(::blaze::core::Error::new(
                    ::blaze::core::ErrorType::InvalidKernel,
                    "kernel \'matrix_mul\' not found",
                ))
            }
        };
        Ok(Self {
            __blaze_inner__,
            __blaze_ctx__,
            matrix_mul,
        })
    }
}

impl<C: ::blaze::context::Context> ::std::ops::Deref for MatrixOps<C> {
    type Target = ::blaze::core::RawProgram;
    #[inline(always)]
    fn deref(&self) -> &Self::Target {
        &self.__blaze_inner__
    }
}

struct MatrixMul<LHS, RHS, OUT> {
    __blaze_inner__: ::blaze::event::RawEvent,
    lhs: LHS,
    rhs: RHS,
    out: OUT,
}

impl<C: ::blaze::context::Context> MatrixOps<C> {
    unsafe fn matrix_mul<
        LHS: ::core::ops::Deref,
        RHS: ::core::ops::Deref,
        OUT: ::core::ops::DerefMut,
        const N: usize,
    >(
        &self,
        k: u32,
        lhs: LHS,
        rhs: RHS,
        out: OUT,
        global_work_dims: [usize; N],
        local_work_dims: impl Into<Option<[usize; N]>>,
        wait: impl Into<::blaze::event::WaitList>,
    ) -> ::blaze::core::Result<MatrixMul<LHS, RHS, OUT>>
    where
        <LHS as ::core::ops::Deref>::Target: ::blaze::buffer::KernelPointer<f32>,
        <RHS as ::core::ops::Deref>::Target: ::blaze::buffer::KernelPointer<f32>,
        <OUT as ::core::ops::Deref>::Target: ::blaze::buffer::KernelPointer<MaybeUninit<f32>>,
    {
        let mut wait = wait.into();
        let mut __blaze_kernel__ = match self.matrix_mul.lock() {
            Ok(x) => x,
            Err(e) => e.into_inner()
        };
        __blaze_kernel__.set_argument(0u32, &k)?;
        ::blaze::buffer::KernelPointer::set_arg(
            ::core::ops::Deref::deref(&lhs),
            &mut __blaze_kernel__,
            &mut wait,
            1u32,
        )?;
        ::blaze::buffer::KernelPointer::set_arg(
            ::core::ops::Deref::deref(&rhs),
            &mut __blaze_kernel__,
            &mut wait,
            2u32,
        )?;
        ::blaze::buffer::KernelPointer::set_arg(
            ::core::ops::Deref::deref(&out),
            &mut __blaze_kernel__,
            &mut wait,
            3u32,
        )?;
        let __blaze_inner__ = __blaze_kernel__.enqueue_with_context(
            &self.__blaze_ctx__,
            global_work_dims,
            local_work_dims,
            wait,
        )?;
        drop(__blaze_kernel__);
        ::blaze::buffer::KernelPointer::complete(
            ::core::ops::Deref::deref(&lhs),
            &__blaze_inner__,
        )?;
        ::blaze::buffer::KernelPointer::complete(
            ::core::ops::Deref::deref(&rhs),
            &__blaze_inner__,
        )?;
        ::blaze::buffer::KernelPointer::complete(
            ::core::ops::Deref::deref(&out),
            &__blaze_inner__,
        )?;
        Ok(MatrixMul {
            __blaze_inner__,
            lhs,
            rhs,
            out,
        })
    }
}

impl<LHS: ::core::ops::Deref, RHS: ::core::ops::Deref, OUT: ::core::ops::DerefMut>
    ::blaze::event::Event for MatrixMul<LHS, RHS, OUT>
where
    <LHS as ::core::ops::Deref>::Target: ::blaze::buffer::KernelPointer<f32>,
    <RHS as ::core::ops::Deref>::Target: ::blaze::buffer::KernelPointer<f32>,
    <OUT as ::core::ops::Deref>::Target: ::blaze::buffer::KernelPointer<MaybeUninit<f32>>,
{
    type Output = (LHS, RHS, OUT);
    #[inline(always)]
    fn as_raw(&self) -> &::blaze::event::RawEvent {
        &self.__blaze_inner__
    }
    #[inline(always)]
    fn consume(
        self,
        err: Option<::blaze::prelude::Error>,
    ) -> ::blaze::prelude::Result<Self::Output> {
        if let Some(err) = err {
            return Err(err);
        };
        Ok((self.lhs, self.rhs, self.out))
    }
}
```