Traits == Rust's take on interfaces.

On top of traits as a concept, we'll also cover some of the key traits that are defined in Rust's standard library:

Operator traits (e.g. `Add`, `Sub`, `PartialEq`, etc.)
- `From` and `Into`, for infallible conversions
- `Clone` and `Copy`, for copying values
- `Deref` and deref coercion
- `Sized`, to mark types with a known size
- `Drop`, for custom cleanup logic


---


What if we wanted to compare two Ticket instances directly?

```rust
let ticket1 = Ticket::new(/* ... */);
let ticket2 = Ticket::new(/* ... */);
ticket1 == ticket2
```

The compiler will stop us:

```rust
error[E0369]: binary operation `==` cannot be applied to type `Ticket`
  --> src/main.rs:18:13
   |
18 |     ticket1 == ticket2
   |     ------- ^^ ------- Ticket
   |     |
   |     Ticket
   |
note: an implementation of `PartialEq` might be missing for `Ticket`
```

Ticket is a new type. Out of the box, there is no behavior attached to it.
Rust doesn't magically infer how to compare two Ticket instances just because they contain Strings.

The Rust compiler is nudging us in the right direction though: it's suggesting that we might be missing an implementation of PartialEq. PartialEq is a trait!

Traits are Rust's way of defining interfaces.
A trait defines a set of methods that a type must implement to satisfy the trait's contract.


The syntax for a trait definition goes like this:

```rust
trait <TraitName> {
    fn <method_name>(<parameters>) -> <return_type>;
}
```

We might, for example, define a trait named MaybeZero that requires its implementors to define an is_zero method:

```rust
trait MaybeZero {
    fn is_zero(self) -> bool;
}
```


To implement a trait for a type we use the impl keyword, just like we do for regular methods, but the syntax is a bit different. e.g.:

```rust
pub struct WrappingU32 {
    inner: u32,
}

impl MaybeZero for WrappingU32 {
    fn is_zero(self) -> bool {
        self.inner == 0
    }
}
```


To invoke a trait method, we use the . operator, just like we do with regular methods:

```rust
let x = WrappingU32 { inner: 5 };
assert!(!x.is_zero());
```

To invoke a trait method, two things must be true:

- The type must implement the trait.
- The trait must be in scope.
- To satisfy the latter, you may have to add a use statement for the trait:

```rust
use crate::MaybeZero;
```


trait example:

```rust
// Define a trait named `IsEven` that has a method `is_even` that returns a `true` if `self` is
// even, otherwise `false`.
//
// Then implement the trait for `u32` and `i32`.

trait IsEven {
    fn is_even(self) -> bool;             
}

impl IsEven for i32 {
    fn is_even(self) -> bool {
        self % 2 == 0
    }
}

impl IsEven for u32 {
    fn is_even(self) -> bool {
        self % 2 == 0
    }
}

```
- self vs &self — Taking self by value is fine for primitives (they're Copy), but using &self is more idiomatic for traits that don't need ownership:

```rust
fn is_even(&self) -> bool;
```


---


There are limitations to the trait implementations you can write.
The simplest and most straight-forward one: you can't implement the same trait twice, in a crate, for the same type.

For example:

```rust
trait IsEven {
    fn is_even(&self) -> bool;
}

impl IsEven for u32 {
    fn is_even(&self) -> bool {
        true
    }
}

impl IsEven for u32 {
    fn is_even(&self) -> bool {
        false
    }
}
```

The compiler will reject it:

```rust
error[E0119]: conflicting implementations of trait `IsEven` for type `u32`
   |
5  | impl IsEven for u32 {
   | ------------------- first implementation here
...
11 | impl IsEven for u32 {
   | ^^^^^^^^^^^^^^^^^^^ conflicting implementation for `u32`
```

There can be no ambiguity as to what trait implementation should be used when `IsEven::is_even` is invoked on a `u32` value, therefore there can only be one.


Things get more nuanced when multiple crates are involved. In particular, at least one of the following must be true:

- The trait is defined in the current crate
- The implementor type is defined in the current crate
- This is known as Rust's orphan rule. Its goal is to make the method resolution process unambiguous.

Imagine the following situation:

- Crate A defines the IsEven trait
- Crate B implements IsEven for u32
- Crate C provides a (different) implementation of the IsEven trait for u32
- Crate D depends on both B and C and calls 1.is_even()

Which implementation should be used? The one defined in B? Or the one defined in C?
There's no good answer, therefore the orphan rule was defined to prevent this scenario. Thanks to the orphan rule, neither crate B nor crate C would compile.

- There are some caveats and exceptions to the orphan rule as stated above. Check out the [reference](https://doc.rust-lang.org/reference/items/implementations.html#trait-implementation-coherence)
if you want to get familiar with its nuances.


---

In Rust, operators are traits.
For each operator, there is a corresponding trait that defines the behavior of that operator. By implementing that trait for your type, you unlock the usage of the corresponding operators.

For example, the PartialEq trait defines the behavior of the == and != operators:

```rust
// The `PartialEq` trait definition, from Rust's standard library
// (It is *slightly* simplified, for now)
pub trait PartialEq {
    // Required method
    //
    // `Self` is a Rust keyword that stands for 
    // "the type that is implementing the trait"
    fn eq(&self, other: &Self) -> bool;

    // Provided method
    fn ne(&self, other: &Self) -> bool { ... }
}
```

When you write x == y the compiler will look for an implementation of the PartialEq trait for the types of x and y and replace x == y with x.eq(y). It's syntactic sugar!

This is the correspondence for the main operators:

```
Operator            Trait
+                  	Add
-                  	Sub
*                  	Mul
/                  	Div
%                  	Rem
== and !=       	PartialEq
<, >, <=, and >=	PartialOrd
```

In PartialEq trait, there's a default implementation for `ne` which is like this:
```rust
fn ne(&self, other: &Self) -> bool {
        !self.eq(other)
    }
```
So we don't have to implement this if we're not gonna use it differently.

`PartialEq` example:

```rust

impl PartialEq for Ticket {
    fn eq(&self, other: &Self) -> bool {
        self.title == other.title
            && self.description == other.description
            && self.status == other.status
    }
}
```

---


Rust macros are code generators.
They generate new Rust code based on the input you provide, and that generated code is then compiled alongside the rest of your program. Some macros are built into Rust's standard library, but you can also write your own. We won't be creating our own macro in this course, but you can find some useful pointers in the "Further reading" section.
https://lukaswirth.dev/tlborm/
https://github.com/dtolnay/proc-macro-workshop

A derive macro is a particular flavour of Rust macro. It is specified as an attribute on top of a struct.

```rust
#[derive(PartialEq)]
struct Ticket {
    title: String,
    description: String,
    status: String
}
```

Derive macros are used to automate the implementation of common (and "obvious") traits for custom types. In the example above, the PartialEq trait is automatically implemented for Ticket. If you expand the macro, you'll see that the generated code is functionally equivalent to the one you wrote manually, although a bit more cumbersome to read:


```rust
#[automatically_derived]
impl ::core::cmp::PartialEq for Ticket {
    #[inline]
    fn eq(&self, other: &Ticket) -> bool {
        self.title == other.title 
            && self.description == other.description
            && self.status == other.status
    }
}
```

bu derive PartialEq'nun avantajı, mesela struct'a yeni bi paramtere eklemek istersek her seferinde gidip onun PartialEq implementasyonunu da değiştirmemiz gerekir.
fakat deriveladığımız zaman o otomatik hallediyo

birden fazla derive için `#[derive(bir,iki,üç)]`


---

All our functions and methods, so far, have been working with concrete types.
Code that operates on concrete types is usually straightforward to write and understand. But it's also limited in its reusability.
Let's imagine, for example, that we want to write a function that returns true if an integer is even. Working with concrete types, we'd have to write a separate function for each integer type we want to support:

```rust
fn is_even_i32(n: i32) -> bool {
    n % 2 == 0
}

fn is_even_i64(n: i64) -> bool {
    n % 2 == 0
}

// Etc.
```

Alternatively, we could write a single extension trait and then different implementations for each integer type:

```rust
trait IsEven {
    fn is_even(&self) -> bool;
}

impl IsEven for i32 {
    fn is_even(&self) -> bool {
        self % 2 == 0
    }
}

impl IsEven for i64 {
    fn is_even(&self) -> bool {
        self % 2 == 0
    }
}

// Etc.
```

The duplication remains.



We can do better using `generics`.
Generics allow us to write code that works with a type parameter instead of a concrete type:
```rust
fn print_if_even<T>(n: T)
where
    T: IsEven + Debug
{
    if n.is_even() {
        println!("{n:?} is even");
    }
}
```

`print_if_even` is a generic function.
It isn't tied to a specific input type. Instead, it works with any type `T` that:

Implements the `IsEven` trait.
Implements the `Debug` trait.
This contract is expressed with a trait bound: `T: IsEven + Debug`.
The + symbol is used to require that T implements multiple traits. T: IsEven + Debug is equivalent to "where T implements IsEven and Debug".

```rust
fn print_if_even<T: IsEven + Debug>(n: T) {
    //           ^^^^^^^^^^^^^^^^^
    //           This is an inline trait bound
    // [...]
}
```

In the examples above, we used T as the type parameter name. This is a common convention when a function has only one type parameter.
Nothing stops you from using a more meaningful name, though:

```rust
fn print_if_even<Number: IsEven + Debug>(n: Number) {
    // [...]
}
```


It is actually desirable to use meaningful names when there are multiple type parameters at play or when the name T doesn't convey enough information about the type's role in the function. Maximize clarity and readability when naming type parameters, just as you would with variables or function parameters. Follow Rust's conventions, though: use upper camel case for type parameter names.
https://rust-lang.github.io/api-guidelines/naming.html#casing-conforms-to-rfc-430-c-case


---

A &str is a view into a string, a reference to a sequence of UTF-8 bytes stored elsewhere. You can, for example, create a &str from a String like this:

```rust
let mut s = String::with_capacity(5);
s.push_str("Hello");
// Create a string slice reference from the `String`, 
// skipping the first byte.
let slice: &str = &s[1..];
```


In memory, it'd look like this:


```rust
                    s                              slice
      +---------+--------+----------+      +---------+--------+
Stack | pointer | length | capacity |      | pointer | length |
      |    |    |   5    |    5     |      |    |    |   4    |
      +----|----+--------+----------+      +----|----+--------+
           |        s                           |  
           |                                    |
           v                                    | 
         +---+---+---+---+---+                  |
Heap:    | H | e | l | l | o |                  |
         +---+---+---+---+---+                  |
               ^                                |
               |                                |
               +--------------------------------+
```


slice stores two pieces of information on the stack:

- A pointer to the first byte of the slice.
- The length of the slice.

slice doesn't own the data, it just points to it: it's a reference to the String's heap-allocated data.
When slice is dropped, the heap-allocated data won't be deallocated, because it's still owned by s. That's why slice doesn't have a capacity field: it doesn't own the data, so it doesn't need to know how much space it was allocated for it; it only cares about the data it references.

-> As a rule of thumb, use &str rather than &String whenever you need a reference to textual data.
&str is more flexible and generally considered more idiomatic in Rust code.

If a method returns a &String, you're promising that there is heap-allocated UTF-8 text somewhere that matches exactly the one you're returning a reference to.
If a method returns a &str, instead, you have a lot more freedom: you're just saying that somewhere there's a bunch of text data and that a subset of it matches what you need, therefore you're returning a reference to it.


---

By implementing Deref<Target = U> for a type T you're telling the compiler that &T and &U are somewhat interchangeable.
In particular, you get the following behavior:

- References to T are implicitly converted into references to U (i.e. &T becomes &U)
- You can call on &T all the methods defined on U that take &self as input.

String implements Deref with Target = str:

```rust
impl Deref for String {
    type Target = str;
    
    fn deref(&self) -> &str {
        // [...]
    }
}
```


---

str is a dynamically sized type (DST).
A DST is a type whose size is not known at compile time. Whenever you have a reference to a DST, like &str, it has to include additional information about the data it points to. It is a `fat pointer`.
In the case of &str, it stores the length of the slice it points to. We'll see more examples of DSTs in the rest of the course.

Rust's std library defines a trait called Sized.

```rust
pub trait Sized {
    // This is an empty trait, no methods to implement.
}
```

A type is `Sized` if its size is known at compile time. In other words, it's not a DST.


`Sized` is your first example of a marker trait.
A marker trait is a trait that doesn't require any methods to be implemented. It doesn't define any behavior. It only serves to mark a type as having certain properties. The mark is then leveraged by the compiler to enable certain behaviors or optimizations.

In particular, `Sized` is also an auto trait.
You don't need to implement it explicitly; the compiler implements it automatically for you based on the type's definition.

All the types we've seen so far are `Sized`: u32, String, bool, etc.

str, as we just saw, is not `Sized`.
&str is `Sized` though! We know its size at compile time: two usizes, one for the pointer and one for the length.


---

burası biraz karışık o yüzden ilk önce direkt örnek göstereyim:

```rust
// TODO: Implement the `From` trait for the `WrappingU32` type to make `example` compile.

pub struct WrappingU32 {
    value: u32,
}
```

cevap:

```
impl From<u32> for WrappingU32 {
    fn from(value: u32) -> Self {
        WrappingU32 { value }
    }
}
```



The Rust standard library defines two traits for infallible conversions: From and Into, in the std::convert module.

```rust
pub trait From<T>: Sized {
    fn from(value: T) -> Self;
}

pub trait Into<T>: Sized {
    fn into(self) -> T;
}
```

testler:
```rust
fn example() {
    let wrapping: WrappingU32 = 42.into();
    let wrapping = WrappingU32::from(42);
}
```


These trait definitions showcase a few concepts that we haven't seen before: supertraits and implicit trait bounds. Let's unpack those first.

Supertrait / Subtrait
The `From: Sized` syntax implies that From is a subtrait of Sized: any type that implements From must also implement Sized. Alternatively, you could say that Sized is a supertrait of From.

Implicit trait bounds
Every time you have a generic type parameter, the compiler implicitly assumes that it's Sized.

For example:

```rust
pub struct Foo<T> {
    inner: T,
}
```
is actually equivalent to:

```rust
pub struct Foo<T: Sized> 
{
    inner: T,
}
```
In the case of From<T>, the trait definition is equivalent to:

```rust
pub trait From<T: Sized>: Sized {
    fn from(value: T) -> Self;
}
```

In other words, both T and the type implementing From<T> must be Sized, even though the former bound is implicit.



`From` and `Into` are dual traits.
In particular, Into is implemented for any type that implements From using a blanket implementation:

```rust
impl<T, U> Into<U> for T
where
    U: From<T>,
{
    fn into(self) -> U {
        U::from(self)
    }
}
```

If a type U implements From<T>, then Into<U> for T is automatically implemented. That's why we can write `let title = "A title".into();`.

Uzun lafın kısası, işin özeti:

```
Always implement From, never Into.
Implementing From gives you Into for free. Implementing Into directly does not give you From.
```

bi örnek daha:

```rust
#[derive(Debug)]
struct Celsius(f32);

#[derive(Debug)]
struct Fahrenheit(f32);

// Implement From<Celsius> for Fahrenheit
impl From<Celsius> for Fahrenheit {
    fn from(c: Celsius) -> Self {
        Fahrenheit(c.0 * 9.0 / 5.0 + 32.0)     

    }
}

fn main() {
    // Now this works automatically!
    let hot: Fahrenheit = Celsius(30.0).into(); // ← Into<Fahrenheit> for Celsius is auto-generated

    println!("{:?}", hot); // Fahrenheit(86.0)
}
```

(c.0'ın anlamı, Celsius'un ilk parametresi anlamında. Çünkü Celsius bir tuple-struct ve tuple elemanlarına ulaşmak için index kullanırız. örneğin Celsius(f32, f32) olsaydı c.1 de yapabilirdik)


örneğin:

```rust
struct Point(f32, f32);

let p = Point(3.0, 5.0);
p.0 // 3.0
p.1 // 5.0
```

kodun bu haliyle aşağıdaki işlemi yapamayız çünkü bu durumda cel'in ownershipliği gittiği için printleyemiyoruz:

```rust
    let cel: Celsius = Celsius(33.4);
    let fah: Fahrenheit = cel.into();
    
    println!("Celsius: {:?}", cel);
    println!("Fahrenheit: {:?}", fah);
```

buna alternatif bazı çözümler:

- 1
```rust
let cel = Celsius(33.4);
let fah = Fahrenheit::from(Celsius(cel.0)); // use cel.0 to copy the f32
println!("Celsius: {:?}", cel);
println!("Fahrenheit: {:?}", fah);
```


- 2
```rust
#[derive(Debug, Clone, Copy)]
struct Celsius(f32);

#[derive(Debug, Clone, Copy)]
struct Fahrenheit(f32);
```
(bunu yapınca kod olduğu gibi çalışır)


- 3 - printledikten sonra ownershipi ver ama gerek yok buna
```rust
println!("Celsius: {:?}", cel);
let fah: Fahrenheit = cel.into();
println!("Fahrenheit: {:?}", fah);
```


---



