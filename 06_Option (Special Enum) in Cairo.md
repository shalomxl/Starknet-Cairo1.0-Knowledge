# 06_Option (Special Enum) in Cairo

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

Like Rust, Cairo also does not have a system-level variable or property that represents null because it is easy to make mistakes such as treating null values as non-null values or treating non-null values as null values.

To better understand this error, let's take an example from Golang:

When using Maps in Golang, there is often an error of "reading data from a Map with a nil state" because the Map may be nil at runtime. nil in Golang means null.

In Cairo, such errors will not occur because the Cairo compiler can detect them at compile time. The reason for this is that Cairo does not have a system-level variable or property that represents null, but instead uses a special enum type to implement non-null judgment (Option).

## Basic usage of Option

Option is defined in the standard library as follows:

```
enum Option<T> {
    Some: T,
    None: (),
}
```

In actual coding, Option type variables can be defined as follows:

```
use option::OptionTrait;

fn main(){
    let some_char = Option::Some('e');
    let some_number: Option<felt252> = Option::None(());
    let absent_number: Option<u32> = Option::Some(8_u32);
}
```

The `Some` member of Option is generic, so it can hold any type. We have put a short string into `Some` above. It should also be noted that if `None` is used, the parameter cannot be empty, and the **unit type** `()` needs to be passed in. The unit type is a special type in Cairo. It is used to ensure that the code in its position will not be compiled (empty state during compilation).

### Similarities and Differences with Rust

**Similarities with Rust:**

* When using `None`, the type in Option needs to be specified, like `let some_number: Option<felt252> = Option::None(());` above, because the compiler cannot determine what type is in Option through `None`.

**Differences with Rust:**

* `None` and `Some` cannot be used globally directly.


### It is not possible to mix variables of other types with `Option`

```
use option::OptionTrait;

fn main() {
    let x: u8 = 5_u8;
    let y: Option<u8> = Option::Some(5_u8);

    let sum = x + y;
}

// You will get the following error:
error: Unexpected argument type. Expected: "core::integer::u8", found: "core::option::Option::<core::integer::u8>".
 --> h04_enum_option.cairo:11:19
    let sum = x + y;
                  ^
```

Although the Option variable y holds a u8 value, the types of y and x are different, so they cannot be added together.

## Non-nullable judgment in Cairo

All types of variables in Cairo are non-null, and the compiler ensures that variables are never null at any time, like x above. That is, when writing Cairo code, we don't need to worry about whether our variables are null at runtime. The only place to consider null values is when using Option variables.

In other words, only Option can obtain the state of null (None). We can see the actual use cases of Option through the control mode of match.

## Source Code Analysis

Source code of the Option core library: https://github.com/starkware-libs/cairo/blob/main/corelib/src/option.cairo

### Four Member Functions

```
/// Determines if there is a value in Option and returns this value. If there is no value, an `err` error is thrown.
fn expect(self: Option<T>, err: felt252) -> T;

/// Determines if there is a value in Option and returns this value. If there is no value, an error is thrown that is not custom.
fn unwrap(self: Option<T>) -> T;

/// Returns true if there is a value in Option.
fn is_some(self: @Option<T>) -> bool;

/// Returns false if there is no value in Option.
fn is_none(self: @Option<T>) -> bool;
```


