# 01_Variables in Cairo

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

Variables are the most fundamental element in programming languages.

## Basic Usage

Creating a variable:

```
use debug::PrintTrait;

fn main() {
    let x = 5;
    let y:u8 = 5;
    x.print();
}
```

Use the let keyword to create a variable. PrintTrait is a printing utility library provided by the official library.

## Variable Mutability

Cairo uses an immutable memory model, meaning that once a memory space is assigned a value, it cannot be overwritten and can only be read.

This means that by default, all variables in Cairo are immutable (which may be counterintuitive 🙃️).

Try running the following code (using the command:`cairo-run $FILE_CAIRO`):

```
use debug::PrintTrait;
fn main() {
    let x = 5;
    x.print();
    x = 6;
    x.print();
}
```

You will get the following error:

```
error: Cannot assign to an immutable variable.
 --> c01_var.cairo:5:5
    x = 6;
    ^***^

Error: failed to compile: src/c01_var.cairo
```

To make a variable mutable, use the mut keyword:

```
use debug::PrintTrait;
fn main() {
    let mut x = 5;
    x.print();
    x = 6;
    x.print();
}
```

In the above example, the mut keyword is added before the variable x, which allows it to be changed.

Think about it 🤔: Immutable variables are similar to constants to some extent. Can they be used as constants?

## Shadowing

Shadowing in Cairo is similar to Rust, which allows multiple variables to use the same variable name. Let's take a look at a specific example:

```
use debug::PrintTrait;
fn main() {
    let mut x = 5_u32;
    let x: felt252 = 10;
    {
	    // 只会影响大括号内的变量，不影响括号以外的
        let x = x * 2;
        'Inner scope x value is:'.print();
        x.print()
    }
    'Outer scope x value is:'.print();
    x.print();
}
```

In the above example, the variable x defined in the line `let x: felt252 = 10;` completely shadows the previous x variable. This is where the term Shadowing comes from. Shadowing has the following characteristics:

1. Use the let keyword to redefine a variable.
2. The redefined variable can have a different data type than the previous one.
3. Shadowing only affects variables in the same namespace.
4. Whether the variable is immutable or mutable, it can be shadowed.

Note⚠️: When using Shadowing, try to avoid using the same variable name in different namespaces, as this can make it difficult to locate bugs.
