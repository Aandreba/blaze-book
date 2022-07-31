# Random Number Generator

This code requires OpenCL 3.0 or higher.

## Rust code
```rust
use std::{time::{SystemTime}, mem::MaybeUninit};
use once_cell::sync::Lazy;
use blaze_rs::prelude::*;

#[global_context]
static CONTEXT : SimpleContext = SimpleContext::default();

static CODE : Lazy<String> = Lazy::new(|| {
    let nanos = SystemTime::now().duration_since(SystemTime::UNIX_EPOCH).unwrap();
    format!("#define TIME {}l\n{}", nanos.as_nanos(), include_str!("rng.cl"))
});

#[blaze(Rng)]
#[link = Lazy::force(&CODE)]
pub extern "C" {
    fn next_ints (n: u32, out: *mut MaybeUninit<u32>);
}

#[test]
fn main () -> Result<()> {
    let rng = Rng::new(None)?;
    let mut random = Buffer::<u32>::new_uninit(5, MemAccess::WRITE_ONLY, false)?;
    
    let random = unsafe {
        let _ = rng.next_ints(5, &mut random, [5], None, EMPTY)?.wait()?;
        random.assume_init()  
    };

    println!("{random:?}");
    Ok(())
}
```

## OpenCL C code
```c
#define MUTPILIER 0x5DEECE66Dl
#define ADDEND 0xBl
#define MASK() ((1l << 48) - 1)

global atomic_ulong SEED = ATOMIC_VAR_INIT((8682522807148012l * 1181783497276652981L) ^ TIME);

kernel void next_ints (const uint n, global uint* out) {
    const uint ID = get_global_id(0);
    const uint SIZE = get_global_size(1);
    ulong oldseed, nextseed;

    for (uint i = ID; i < n; i += SIZE) {
        do {
            oldseed = atomic_load(&SEED);
            nextseed = (oldseed * MUTPILIER + ADDEND) & MASK();
        } while (!atomic_compare_exchange_strong(&SEED, &oldseed, nextseed));
        out[i] = nextseed >> 16;
    }
}
```