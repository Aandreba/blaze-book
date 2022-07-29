# Rustified experience

The main goal of Blaze is to provide a Rustified experience of OpenCL.
This is achieved by wrapping the OpenCL API inside Rust types that offer safety guarantees provided by the Rust compiler, such as automatic release of memory through the [`Drop`](https://doc.rust-lang.org/stable/std/ops/trait.Drop.html) trait, and thread safety via [`Send`](https://doc.rust-lang.org/stable/std/marker/trait.Send.html) and [`Sync`](https://doc.rust-lang.org/stable/std/marker/trait.Sync.html).

Another major part of the Rust experience is the use of zero-cost abstractions, which also means that Blaze is written in such a way that the Rust compiler can maximally optimize the resulting code, such with the use of [`NonNull`](https://doc.rust-lang.org/stable/std/ptr/struct.NonNull.html) pointers.  