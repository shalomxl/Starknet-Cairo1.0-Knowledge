# 13_Map in Cairo

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

## Basic usage

Map can also be referred to as a dictionary in Cairo. The basic usage includes creating a dictionary, inserting key-value pairs, and reading data. Let's look at an example:

```
use core::debug::PrintTrait;
use dict::Felt252DictTrait;
use traits::Default;

fn main(){
    let mut map : Felt252Dict<felt252> = Default::default();
    map.insert(1,'shalom');
    map[1].print();
}
```

To create a dictionary, we need to use the `Default` trait, which returns an initial dictionary of type `Felt252Dict`. We also need to specify the variable type in the dictionary. The type of the map created above is `Felt252Dict<felt252>`.

`Felt252Dict` is the dictionary type currently supported by Cairo, and it can only use variables of type `felt252` as keys. The value can be of multiple types: `u8`, `u16`, `u32`, `u64`, `u128`, `felt252`. Therefore, it is named `Felt252Dict`.

To insert key-value pairs, use the `insert(key, value)` member function, where the first parameter is `key` and the second parameter is `value`.

To read data, simply use the key in square brackets `map[key]`.
