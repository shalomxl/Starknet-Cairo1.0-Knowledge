# 02_Constants in Cairo

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

## Basic Usage

```
use debug::PrintTrait;

const ONE_HOUR_IN_SECONDS: felt252 = 3600;

fn main(){
    ONE_HOUR_IN_SECONDS.print();
}
```

To define a constant in Cairo, use the const keyword, specify the type of the constant, and give it a value.

## Difference between constants and immutable variables

Constants have the following characteristics:

1. You cannot use the `mut` keyword with constants.
2. Constants can only be declared in global scope.
3. Only literals can be used to assign values to constants.

Try declaring a constant inside a function:

```
use debug::PrintTrait;

fn main(){
	const ONE_HOUR_IN_SECONDS: felt252 = 3600;
    ONE_HOUR_IN_SECONDS.print();
}
```

You will receive a series of errors 🙅.

Assigning a non-literal value to a constant will also result in an error:

```
use debug::PrintTrait;

const TEST: felt252 = 3600;
const ONE_HOUR_IN_SECONDS: felt252 = TEST;

fn main(){
    ONE_HOUR_IN_SECONDS.print();
}
```

In the above code, an error will be received when using one constant to assign a value to another constant.

```
error: Only literal constants are currently supported.
 --> d_const.cairo:4:38
const ONE_HOUR_IN_SECONDS: felt252 = test;
                                     ^**^
```

## Summary of Variables and Constants in Cairo

In Cairo, variable declarations are "immutable" by default, which is quite interesting. Because in other mainstream programming languages, variables are declared as mutable by default, while Cairo does the opposite. This can be understood as "immutable" usually has better stability, while mutable can bring instability. Therefore, Cairo should be a language that aims to be safer. In addition, Cairo also has constants that can be modified by the const modifier. Therefore, Cairo can do the following:

* Constants: `const LEN:u32 = 1024;` The `LEN` here is an unsigned 32-bit integer constant (u32), which is used at compile-time.
* Mutable variables: `let mut x = 5;` This is similar to other languages, used at run-time.
* Immutable variables: `let x = 5;` You cannot modify this type of variable. However, you can redefine a new `x` using the syntax `let x = x + 10;`. This is called Shadowing in Cairo, where the second x shadows the first x.

Using Shadowing in Cairo can be tricky. Bugs caused by using variables with the same name (in nested scope environments) can be difficult to find. In general, each variable should have its own appropriate name, and it is best not to reuse names.

### Advantages of Default Immutability

Immutable variables are helpful for stable program execution. This is a programming "contract". When dealing with contracts for immutable variables, the program can be more stable, especially in multi-threaded environments, because immutable means read-only and not write.

Other advantages include that they are easier to understand and reason about than mutable objects, and they provide higher security. With such a "contract", the compiler can easily check for errors at compile-time. This is why the Cairo compiler can help you detect many programming issues during compilation.
