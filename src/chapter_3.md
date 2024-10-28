# Dbg macro

The `dbg!` macro introduced in version `1.32` is pretty much a `println!` made for debugging.

An example from the official docs page
```rust
let a = 2;
let b = dbg!(a * 2) + 1;
//      ^-- prints: [src/main.rs:2:9] a * 2 = 4
assert_eq!(b, 5);
```

Notice that the file path and line numbers are printed.
This metadata is generated at compile time, so it will keep up with changes made in your program.

This means it is less essential to make your `dbg` statements unique.

However, there are still some disadvantages (same as `println!`)
- Typing or more likely copying and pasting `dbg!`'s everywhere gets quite tedious
- At some point these `dbg!`'s need to removed or they will block up the logging
