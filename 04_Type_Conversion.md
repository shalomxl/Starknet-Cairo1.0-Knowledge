# 04_Type Conversion in Cairo

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

Currently, type conversion in Cairo mainly involves converting integers of various types to each other, such as u8, u16, u256, felt252, etc. Even for the very similar types u8 and u16, converting between them is not as easy as in other programming languages.

## TryInto & Into traits

First, let's introduce two traits provided by these two official libraries. We need to use these two traits to implement integer type conversion.

**(1). TryInto** 

The TryInto trait provides the `try_into`, which is used when the value range of the source type is larger than that of the target type, such as converting felt252 to u32. The return value of the `try_into` function is of type `Option<T>`. If the target type cannot hold the original value, None is returned; if it can hold it, Some is returned. If Some is returned, the `unwrap` function is then used to convert the return value to the target type. See the example:

```
use traits::TryInto;
use option::OptionTrait;

fn main() {
    let my_felt252 = 10;
	let my_usize: usize = my_felt252.try_into().unwrap();
}
```

In the above example, a `felt252` variable `my_felt252` is defined, then the `try_into` function is used to convert it to an `Option<T>` type, and the `unwrap` function of `Option` is called to convert the generic variable in `Option` to a `usize` type variable `my_usize`.

**(2). Into**

After understanding `TryInto`, `Into` is easy to understand. The `Into` trait provides the `into` function, which is used when the value range of the target type is larger than that of the source type. In this case, we don't have to worry about overflow errors, so we can convert with confidence. For example, converting `u32` to `u64`. The `into` function will directly return the target type variable without the need to use `Option` for conversion.

```
use traits::TryInto;
use traits::Into;
use option::OptionTrait;

fn main() {
    let my_u8: u8 = 10;
    let my_u16: u16 = my_u8.into();

	let my_u256:u256 = my_felt252.into();
}
```
 
It is worth noting that `u256` can also be converted using these two traits.
