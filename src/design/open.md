# Openness

Another major goal of Blaze is to be as open as possible. This translates into a series of rules that all developers must follow when contributing to the Blaze project.

## Trust the user, with caution
In my opinion, one of the main reasons programming languges end up being cluttered messes is a lack of trust in the user.

> To clarify, _user_ here refers to the developers using our library.

A great example of this is Java, where getters and setters galore.
```java
public class Person {
    private String name;
    private LocalDate birthDate;

    public Person (String name, LocalDate birthDate) {
        if birthDate.isAfter(LocalDate.now()) {
            throw new IllegalArgumentException("Birth date cannot be in the future");
        }

        this.name = name;
        this.birthDate = birthDate;        
    }

    public Person (String name, int age) {
        this(name, LocalDate.now().minusYears(age));
    }

    public String getName() {
        return name;
    }

    public LocalDate getBirthDate() {
        return birthDate;
    }

    public int getAge () {
        return Period.between(this.birthDate, LocalTime.now()).getYears();
    }
}
```

This level of distrust amongst develepers is already too ingrained in the Java community to do anything about it, but it doesn't have to be this way with Rust.

```rust,mdbook-runnable
use std::time::{SystemTime, Duration, SystemTimeError};

pub const SECS_IN_YEAR : u64 = 31557600;

#[derive(Debug, Clone)]
#[non_exhaustive]
pub struct Person {
    pub name: Box<str>,
    pub birth_date: SystemTime
}

impl Person {
    pub fn new (name: impl ToString, birth_date: SystemTime) -> Result<Self, SystemTimeError> {
        let _ = birth_date.elapsed()?;

        Ok(Person {
            name: name.to_string().into_boxed_str(),
            birth_date
        })
    }

    #[inline(always)]
    pub fn from_age (name: impl ToString, age: Duration) -> Self {
        let birth_date = SystemTime::now() - age;
        unsafe { Self::new(name, birth_date).unwrap_unchecked() }
    }

    #[inline(always)]
    pub fn age (&self) -> u64 {
        /// SAFETY: We checked at creation that `birth_date` is in the past.
        let dur = unsafe {
            self.birth_date.elapsed().unwrap_unchecked()
        };

        dur.as_secs() / SECS_IN_YEAR
    }
}

fn main () {
    let alex = Person::from_age("Alex", Duration::from_secs(21 * SECS_IN_YEAR));
    println!("{}: {} years old", alex.name, alex.age());
    assert_eq!(alex.age(), 21);
}
```

## No _sealed_ traits
One of the most infuriating experiences I've had as a Rust developer is dealing with _sealed_ traits. Sealed traits are acompliched with the following technique.

```rust,mdbook-runnable
mod sealed {
    pub trait Sealed {}
}

pub trait SelaedTrait: sealed::Sealed {}
```

This way, the `Sealed` trait is public, but cannot be accessed outside of the crate, making `SealedTrait` only implementable inside the crate.

Whilst this may seem like a good idea in some ocations (for example, Blaze's `SvmPointer` trait), it shows a great level of distrust towards the user by the developer, and clashes with the open-source philosophy of this project, and the Rust project in general.

If a contributor finds itself in a position where a _sealed_ trait could make sense, it must abstain from implementing it. Instead, an [`unsafe trait`](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#implementing-an-unsafe-trait) should be implemented, with a 'Safety' section in it's documentation detailing when it can be implemented safely.

Let's take the example of the `SvmPointer` trait. Whilst it might seem like a good idea at first, sealing it would make it impossible for downstream crates and programs to implement this trait for their own custom data type that may utilize SVM pointers (for example, a reference counted SVM pointer).

### _Sealed_ trait example 👎
```rust,mdbook-runnable
mod sealed {
    pub trait Sealed {}
}

pub trait Primitive: sealed::Sealed {}

impl sealed::Sealed for i32 {}
impl Primitive for i32 {}
```

### Unsafe trait example 👍
```rust,mdbook-runnable
/// # Safety
/// This trait must only be implemented on primitive types
pub unsafe trait Primitive {}

unsafe impl Primitive for i32 {}
```