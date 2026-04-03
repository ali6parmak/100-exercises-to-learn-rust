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


