# 10_Function in Cairo

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

## Basic Usage

Functions are an essential building block in any programming language. A function generally includes a function name, parameters, and a return value. In Cairo, the convention is to use "snake case" naming for function and variable names, like `my_function_name`.

```
use debug::PrintTrait;

fn another_function() {
    'Another function.'.print();
}

fn main() {
    'Hello, world!'.print();
    another_function();
}
```

Function calls in Cairo are similar to most languages. `another_function()` is the syntax for calling a regular function, and `'Hello, world!'.print()` is the syntax for calling a function in a trait.

## Parameters & Return Values
Cairo is a statically typed language, so the type of each function parameter and return value must be explicitly specified. If not specified, an error will occur, as shown below:

```
fn add(a: felt252, b) {
    let c = a + b;
    return c;
}

fn main() -> felt252 {
   add(3, 5) 
}
```

There are two errors in the code above:

1. The type of parameter `b` is not specified.
2. Function `add` does not specify the return type, but uses the `return` statement to return variable `c`.

The correct code is:

```
fn add(a: felt252, b: felt252) -> felt252{
    let c = a + b;
    return c;
}

fn main() -> felt252 {
   add(3, 5) 
}
```

### Return Statements

You can use return to explicitly return a value, or use a statement without a semicolon to return a value. For example:

```
fn add(a: felt252, b: felt252) -> felt252 {
	// 返回 a + b
    a + b
}

fn sub(a: felt252, b: felt252) -> felt252 {
    return a - b;
}

fn main() -> felt252 {
   add(3, 5);
   sub(11, 7)
}
```
