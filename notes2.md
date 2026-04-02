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

