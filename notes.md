In Rust, "if" expressions are expressions, not statements: they return a value.
That value can be assigned to a variable or used in other expressions. For example:

```rust
let number = 3;
let message = if number < 5 {
    "smaller than 5"
} else {
    "greater than or equal to 5"
};
```

---

`Panic` is Rust's way to signal that something went so wrong that the program can't continue executing, it's an unrecoverable error.
You can intentionally trigger a panic by calling the `panic!` macro:

```rust
fn main() {
    panic!("This is a panic!");
    // The line below will never be executed
    let x = 1 + 2;
}
```


---

Variables in Rust are immutable by default. You can't change their value once it has been assigned.
If you want to allow modifications, you have to declare the variable as mutable using the mut keyword:

```rust
// `sum` and `i` are mutable now!
let mut sum = 0;
let mut i = 1;

while i <= 5 {
    sum += i;
    i += 1;
}
```

---


Cargo provides 4 built-in profiles: dev, release, test, and bench. The dev profile is used every time you run cargo build, cargo run or cargo test. It's aimed at local development, therefore it sacrifices runtime performance in favor of faster compilation times and a better debugging experience.
The release profile, instead, is optimized for runtime performance but incurs longer compilation times. You need to explicitly request via the --release flag—e.g. cargo build --release or cargo run --release. The test profile is the default profile used by cargo test. The test profile inherits the settings from the dev profile. The bench profile is the default profile used by cargo bench. The bench profile inherits from the release profile. Use dev for iterative development and debugging, release for optimized production builds, test for correctness testing, and bench for performance benchmarking.


Our recommendation is to enable overflow-checks for both profiles: it's better to crash than to silently produce incorrect results. The runtime performance hit is negligible in most cases; if you're working on a performance-critical application, you can run benchmarks to decide if it's something you can afford.


You can opt into wrapping arithmetic on a per-operation basis by using the wrapping_ methods. For example, you can use wrapping_add to add two integers with wrapping:

```rust
let x = 255u8;
let y = 1u8;
let sum = x.wrapping_add(y);
assert_eq!(sum, 0);
```
(wraps to 0)

Alternatively, you can opt into saturating arithmetic by using the saturating_ methods.
Instead of wrapping around, saturating arithmetic will return the maximum or minimum value for the integer type. For example:

```rust
let x = 255u8;
let y = 1u8;
let sum = x.saturating_add(y);
assert_eq!(sum, 255);
```
(sum becomes 255 because it's the u8::MAX. similar thing happens for underflow too, e.g., for 0-1, it returns 0 since u8::MIN is 0)




