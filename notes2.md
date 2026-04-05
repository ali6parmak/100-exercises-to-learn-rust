Defining a struct:

```rust
struct Ticket {
    title: String,
    description: String,
    status: String
}
```

Instantiation:

```rust
// Syntax: <StructName> { <field_name>: <value>, ... }
let ticket = Ticket {
    title: "Build a ticket system".into(),
    description: "A Kanban board".into(),
    status: "Open".into()
};
```

You can access the fields of a struct using the . operator:

```rust
// Field access
let x = ticket.description;
```

We can attach behaviour to our structs by defining methods.
Using the Ticket struct as an example:

```rust
impl Ticket {
    fn is_open(self) -> bool {
        self.status == "Open"
    }
}
```

- Methods must be defined inside an `impl` block
- Methods may use `self` as their first parameter. `self` is a keyword and represents the instance of the struct the method is being called on.
- If a method doesn't take self as its first parameter, it's a static method.
    - The only way to call a static method is by using the function call syntax:
    ```rust
    let default_config = Configuration::default();
    ```

---


If your module is declared in the root of your crate (e.g. src/lib.rs or src/main.rs), cargo expects the file to be named either:

`src/<module_name>.rs`
`src/<module_name>/mod.rs`
If your module is a submodule of another module, the file should be named:

`[..]/<parent_module>/<module_name>.rs`
`[..]/<parent_module>/<module_name>/mod.rs`
E.g. src/animals/dog.rs or src/animals/dog/mod.rs if dog is a submodule of animals.


- 1. crate:: (Absolute Path)
    - Analogy: Like starting a file path with / on Linux/macOS or C:\ on Windows.
    - What it does: It starts searching from the absolute root of your project (usually your src/main.rs or src/lib.rs file).
    - Why use it: This is generally the preferred way to write paths because if you move the current file or module somewhere else in your project, an absolute path will still correctly find the item, whereas a relative path might break.
    -> Example: crate::module_1::MyStruct says "Go to the very top of this project, find module_1, and then grab MyStruct."
    
- 2. super:: (Relative Path to Parent)
    - Analogy: Like using ../ in a command line to go up one directory.
    - What it does: It starts searching from the parent module of the module you are currently writing code in.
    - Why use it: It's incredibly useful when a child module needs to access an item defined in its parent, especially if that parent item isn't public to the rest of the crate (children can see their parents' private items).
    -> Example: super::my_function says "Go up one level in the module hierarchy and run my_function."

- 3. sub_module_1:: (Relative Path from Current Scope)
    - Analogy: Like just typing the name of a folder that is right inside your current directory (e.g., ./folder_name or just folder_name).
    - What it does: It starts searching from the current module you are in.
    - Why use it: To access a sibling module or a child module that is defined right next to where you are writing code. Rust also has a self:: keyword that does this explicitly (e.g., self::sub_module_1::MyStruct), but you can usually just drop self:: and start with the item's name.


---


- `pub:` makes the entity public, i.e. accessible from outside the module where it's defined, potentially from other crates.
- `pub(crate):` makes the entity public within the same crate, but not outside of it.
- `pub(super):` makes the entity public within the parent module.
- `pub(in path::to::module):` makes the entity public within the specified module.

You can use these modifiers on modules, structs, functions, fields, etc. For example:

```rust
pub struct Configuration {
    pub(crate) version: u32,
    active: bool,
}
```

`Configuration` is public, but you can only access the `version` field from within the same crate. The `active` field, instead, is private and can only be accessed from within the same module or one of its submodules.


In Rust, you never put `pub` in front of an `impl` block.
If your struct is public, anyone can see the struct, but they can only use the specific methods inside the `impl` block that you explicitly marked with `pub`.


---

If at least one field is private it is no longer possible to create a Ticket instance directly using the struct instantiation syntax:

```rust
// This won't work!
let ticket = Ticket {
    title: "Build a ticket system".into(),
    description: "A Kanban board".into(),
    status: "Open".into()
};
```


---

```rust
impl Ticket {
    pub fn description(self) -> String {
        self.description
    }
}
```

`Ticket::description` takes ownership of the Ticket instance it's called on.
This is known as move semantics: ownership of the value (self) is moved from the caller to the callee, and the caller can't use it anymore.

In particular, this is the sequence of events that unfold when we call `ticket.status()`:

- Ticket::status takes ownership of the Ticket instance
- Ticket::status extracts status from self and transfers ownership of status back to the caller
- The rest of the Ticket instance is discarded (title and description)
- When we try to use ticket again via ticket.title(), the compiler complains: the ticket value is gone now, we no longer own it, therefore we can't use it anymore.

To build useful accessor methods we need to start working with references.

Whenever you borrow a value, you get a reference to it.
References are tagged with their privileges:

- Immutable references (&) allow you to read the value, but not to mutate it
- Mutable references (&mut) allow you to read and mutate the value

Going back to the goals of Rust's ownership system:

- Data is never mutated while it's being read
- Data is never read while it's being mutated

To ensure these two properties, Rust has to introduce some restrictions on references:

- You can't have a mutable reference and an immutable reference to the same value at the same time
- You can't have more than one mutable reference to the same value at the same time
- The owner can't mutate the value while it's being borrowed
- You can have as many immutable references as you want, as long as there are no mutable references



```rust
struct Configuration {
    version: u32,
    active: bool,
}

fn main() {
    let config = Configuration {
        version: 1,
        active: true,
    };
    // `b` is a reference to the `version` field of `config`.
    // The type of `b` is `&u32`, since it contains a reference to 
    // a `u32` value.
    // We create a reference by borrowing `config.version`, using 
    // the `&` operator.
    // Same symbol (`&`), different meaning depending on the context!
    let b: &u32 = &config.version;
    //     ^ The type annotation is not necessary, 
    //       it's just there to clarify what's going on
}
```



---


You have a u32 input argument in a function? Those 32 bits will be on the stack.
You define a local variable of type i64? Those 64 bits will be on the stack.
It all works quite nicely because the size of those integers is known at compile time, therefore the compiled program knows how much space it needs to reserve on the stack for them.

You can verify how much space a type would take on the stack using the `std::mem::size_of` function.

For a `u8`, for example:

```rust
// We'll explain this funny-looking syntax (`::<u8>`) later on.
// Ignore it for now.
assert_eq!(std::mem::size_of::<u8>(), 1);
```

-> If you have nested function calls, each function pushes its data onto the stack when it's called but it doesn't pop it off until the innermost function returns. If you have too many nested function calls, you can run out of stack space—the stack is not infinite! That's called a stack overflow. 


---

The stack is great, but it can't solve all our problems. What about data whose size is not known at compile time? Collections, strings, and other dynamically-sized data cannot be (entirely) stack-allocated. That's where the `heap` comes in.

You can visualize the heap as a big chunk of memory—a huge array, if you will.
Whenever you need to store data on the heap, you ask a special program, the allocator, to reserve for you a subset of the heap. We call this interaction (and the memory you reserved) a heap allocation. If the allocation succeeds, the allocator will give you a pointer to the start of the reserved block.

The heap is structured quite differently from the stack.
Heap allocations are not contiguous, they can be located anywhere inside the heap.


---

The stack is great, but it can't solve all our problems. What about data whose size is not known at compile time? Collections, strings, and other dynamically-sized data cannot be (entirely) stack-allocated. That's where the heap comes in.

When you create a local variable of type String, Rust is forced to allocate on the heap: it doesn't know in advance how much text you're going to put in it, so it can't reserve the right amount of space on the stack.
But a String is not entirely heap-allocated, it also keeps some data on the stack. In particular:

The pointer to the heap region you reserved.
The length of the string, i.e. how many bytes are in the string.
The capacity of the string, i.e. how many bytes have been reserved on the heap.
Let's look at an example to understand this better:


```rust
let mut s = String::with_capacity(5);
```

```
      +---------+--------+----------+
Stack | pointer | length | capacity | 
      |  |      |   0    |    5     |
      +--|------+--------+----------+
         |
         |
         v
       +---+---+---+---+---+
Heap:  | ? | ? | ? | ? | ? |
       +---+---+---+---+---+
```



We asked for a String that can hold up to 5 bytes of text.
String::with_capacity goes to the allocator and asks for 5 bytes of heap memory. The allocator returns a pointer to the start of that memory block.
The String is empty, though. On the stack, we keep track of this information by distinguishing between the length and the capacity: this String can hold up to 5 bytes, but it currently holds 0 bytes of actual text.

If you push some text into the String, the situation will change:


```rust
s.push_str("Hey");
```


```
      +---------+--------+----------+
Stack | pointer | length | capacity |
      |  |      |   3    |    5     |
      +--|  ----+--------+----------+
         |
         |
         v
       +---+---+---+---+---+
Heap:  | H | e | y | ? | ? |
       +---+---+---+---+---+
```

s now holds 3 bytes of text. Its length is updated to 3, but capacity remains 5. Three of the five bytes on the heap are used to store the characters H, e, and y.


How much space do we need to store pointer, length and capacity on the stack?
It depends on the architecture of the machine you're running on.

Every memory location on your machine has an address, commonly represented as an unsigned integer. Depending on the maximum size of the address space (i.e. how much memory your machine can address), this integer can have a different size. Most modern machines use either a 32-bit or a 64-bit address space.

Rust abstracts away these architecture-specific details by providing the usize type: an unsigned integer that's as big as the number of bytes needed to address memory on your machine. On a 32-bit machine, usize is equivalent to u32. On a 64-bit machine, it matches u64.

Capacity, length and pointers are all represented as usizes in Rust2.


e.g.

```rust
assert_eq!(size_of::<String>(), 24);
```
For a 64 bit machine, size_of::<String>() would become 24. Because, 64-bit = 8 byte, and String has composed of 3 usize fields (length, capacity, pointer) each being 8 bytes.
(For a 32 bit machine, it would've been 12 bytes)


---

Most references1 in Rust are represented, in memory, as a pointer to a memory location.
It follows that their size is the same as the size of a pointer, a usize.

You can verify this using std::mem::size_of:


```rust
assert_eq!(std::mem::size_of::<&String>(), 8);
assert_eq!(std::mem::size_of::<&mut String>(), 8);
```

A &String, in particular, is a pointer to the memory location where the String's metadata is stored.
If you run this snippet:

```rust
let s = String::from("Hey");
let r = &s;
```

you'll get something like this in memory:


```
           --------------------------------------
           |                                    |
      +----v----+--------+----------+      +----|----+
Stack | pointer | length | capacity |      | pointer |
      |  |      |   3    |    5     |      |         |
      +--|  ----+--------+----------+      +---------+
         |          s                           r
         |
         v
       +---+---+---+---+---+
Heap   | H | e | y | ? | ? |
       +---+---+---+---+---+
```

It's a pointer to a pointer to the heap-allocated data, if you will. The same goes for &mut String.



The example above should clarify one thing: not all pointers point to the heap.
They just point to a memory location, which may be on the heap, but doesn't have to be.



---

When introducing the heap, we mentioned that you're responsible for freeing the memory you allocate.
When introducing the borrow-checker, we also stated that you rarely have to manage memory directly in Rust.

These two statements might seem contradictory at first. Let's see how they fit together by introducing scopes and destructors.


The scope of a variable is the region of Rust code where that variable is valid, or alive.

The scope of a variable starts with its declaration. It ends when one of the following happens:

1. the block (i.e. the code between {}) where the variable was declared ends:
```rust
fn main() {
   // `x` is not yet in scope here
   let y = "Hello".to_string();
   let x = "World".to_string(); // <-- x's scope starts here...
   let h = "!".to_string(); //   |
} //  <-------------- ...and ends here
```


2. ownership of the variable is transferred to someone else (e.g. a function or another variable):
```rust
fn compute(t: String) {
   // Do something [...]
}

fn main() {
    let s = "Hello".to_string(); // <-- s's scope starts here...
                //                    | 
    compute(s); // <------------------- ..and ends here
                //   because `s` is moved into `compute`
}
```


When the owner of a value goes out of scope, Rust invokes its destructor.
The destructor tries to clean up the resources used by that value—in particular, whatever memory it allocated.

You can manually invoke the destructor of a value by passing it to std::mem::drop.
That's why you'll often hear Rust developers saying "that value has been dropped" as a way to state that a value has gone out of scope and its destructor has been invoked.


We can insert explicit calls to drop to "spell out" what the compiler does for us. Going back to the previous example:


```rust
fn main() {
   let y = "Hello".to_string();
   let x = "World".to_string();
   let h = "!".to_string();
}
```

It's equivalent to:


```rust
fn main() {
   let y = "Hello".to_string();
   let x = "World".to_string();
   let h = "!".to_string();
   // Variables are dropped in reverse order of declaration
   drop(h);
   drop(x);
   drop(y);
}
```


Let's look at the second example instead, where s's ownership is transferred to compute:

```rust
fn compute(s: String) {
   // Do something [...]
}

fn main() {
   let s = "Hello".to_string();
   compute(s);
}
```


It's equivalent to this:


```rust
fn compute(t: String) {
    // Do something [...]
    drop(t); // <-- Assuming `t` wasn't dropped or moved 
             //     before this point, the compiler will call 
             //     `drop` here, when it goes out of scope
}

fn main() {
    let s = "Hello".to_string();
    compute(s);
}
```


What if a variable contains a reference?
For example:

```rust
let x = 42i32;
let y = &x;
drop(y);
```

When you call drop(y)... nothing happens. It just drops the pointer `y`, nothing happens to `x`.
If you actually try to compile this code, you'll get a warning:

```rust

warning: calls to `std::mem::drop` with a reference 
         instead of an owned value does nothing
 --> src/main.rs:4:5
  |
4 |     drop(y);
  |     ^^^^^-^
  |          |
  |          argument has type `&i32`
  |
```

It goes back to what we said earlier: we only want to call the destructor once.
You can have multiple references to the same value—if we called the destructor for the value they point at when one of them goes out of scope, what would happen to the others? They would refer to a memory location that's no longer valid: a so-called dangling pointer, a close relative of use-after-free bugs. Rust's ownership system rules out these kinds of bugs by design.


---

For types that implement `Copy` (all primitives: u32, i32, bool, f64, etc.), when you pass them to a function, Rust automatically copies the value instead of moving ownership. So the original variable is still valid after the call.


```rust
let x: u32 = 5;
some_function(x);  // x is copied, not moved
println!("{}", x); // ✅ x is still valid here
```

Compare this to a String, which does not implement Copy:

```rust
let s = String::from("hello");
some_function(s);  // s is MOVED, ownership transferred
println!("{}", s); // ❌ compile error! s was moved
```

That's why for String you either:

- Pass a reference &String / &str (borrow it), or
- Clone it (.clone()) if you need two owned copies



