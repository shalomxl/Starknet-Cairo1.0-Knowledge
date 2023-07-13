# 08_Match Control Pattern in Cairo

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

The Match control pattern in Cairo is 90% the same as Rust's. As Cairo is still under development and many features are not yet fully developed, we can refer to Match Control Pattern in Rust as a reference to learn this feature in Cairo.

## Basic Usage

If the matched code block is relatively short, curly braces can be omitted and separated by commas. If it is longer, curly braces are required.

```
use debug::PrintTrait;

#[derive(Drop)]
enum Coin {
    Penny:(),
    Nickel:(),
    Dime:(),
    Quarter:(),
}

fn value_in_cents(coin: Coin) -> felt252 {
    match coin {
        Coin::Penny(_) => 1,
        Coin::Nickel(_) => 5,
        Coin::Dime(_) => 10,
        Coin::Quarter(_) => 25,
    }
}

fn main() {
	let p = Coin::Penny(());
	let n = Coin::Nickel(());
	value_in_cents(p).print();
	value_in_cents(n).print();
}
```

The parameter type of `value_in_cents` in the above code is Coin, and variables of Coin's subtypes can also be used as arguments for `value_in_cents`. As previously mentioned, an enum can be understood as a collection of subtypes, and all subtypes can represent the parent type. Therefore, any variable of a subtype can be used as an argument for the `value_in_cents` function.

**Usage with curly braces:**

```
fn value_in_cents(coin: Coin) -> felt252 {
    match coin {
        Coin::Penny(_) => {
            'Lucky penny!'.print();
            1
        },
        Coin::Nickel(_) => 5,
        Coin::Dime(_) => 10,
        Coin::Quarter(_) => 25,
    }
}
```

### Differences from Rust

**(1) Differences in Enum definition**

* The way to specify the type of Enum elements is different. In Cairo, you need to add a colon followed by the type, e.g. `Penny:(u8)`, while in Rust, you don't need the colon, e.g. `Penny(u8)`.
* When the type of Enum elements is not specified, Cairo cannot omit the parentheses, e.g. `Penny:()`, while Rust can directly omit them.

**(2) Differences in Match matching conditions**

* If the type of Enum element is not specified, you must use `(_)` to indicate it in Cairo, while Rust does not require this.

**(3) Differences in passing parameters**

* If the type of Enum element is not specified, you need to pass a standard type as a parameter, e.g. `let n = Coin::Nickel(());`.

|           | **Enum definition**  | **Match matching conditions**    | **Passing parameters**     |
| --------- | ------------ | ---------------- | ----------------- |
| **Rust**  | `Penny(u8)`  | `Coin::Penny`    | `Coin::Penny()`   |
| **Cairo** | `Penny:(u8)` | `Coin::Penny(_)` | `Coin::Penny(())` |

## Binding Parameters

Enums can have associated types, and the match arms of the match expression can bind the associated type of the enum as a parameter to the arm.

```
use debug::PrintTrait;

// #[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama: (),
    Alaska: (),
// --snip--
}

enum Coin {
    Penny: (),
    Nickel: (),
    Dime: (),
    // An enum type has been added here.
    Quarter: UsState,
}

impl UsStatePrintImpl of PrintTrait<UsState> {
    fn print(self: UsState) {
        match self {
            UsState::Alabama(_) => ('Alabama').print(),
            UsState::Alaska(_) => ('Alaska').print(),
        }
    }
}

fn value_in_cents(coin: Coin) -> felt252 {
    match coin {
        Coin::Penny(_) => 1,
        Coin::Nickel(_) => 5,
        Coin::Dime(_) => 10,
        Coin::Quarter(state) => {
            state.print();
            25
        }
    }
}

fn main() {
    let u = Coin::Quarter(UsState::Alabama(()));

    value_in_cents(u);
}
```

In the code above, `UsState::Alabama` becomes the parameter `state` in `Coin::Quarter(state)`.

### Differences from Rust
In Cairo, enums do not implement the `PrintTrait` by default. Therefore, an `impl` is needed for `UsState` to implement the printing function.

```
impl UsStatePrintImpl of PrintTrait::<UsState> {
    fn print(self: UsState) {
        match self {
            UsState::Alabama(_) => ('Alabama').print(),
            UsState::Alaska(_) => ('Alaska').print(),
        }
    }
}
```

## Using Match Patterns with Option
By using match patterns with `Option`, we can achieve null checking.

Let's try to implement a function that adds 1 to a value if it is not empty, otherwise it does nothing.

```
use option::OptionTrait;
use debug::PrintTrait;

fn plus_one(x: Option<u8>) -> Option<u8> {
    match x {
        Option::Some(val) => Option::Some(val + 1_u8),
        Option::None(_) => {
            // Additional operations can be added here ...
            Option::None(())
        },
    }
}

fn main() {
    let five: Option<u8> = Option::Some(5_u8);
    let six: Option<u8> = plus_one(five);
    six.unwrap().print();

    let none = plus_one(Option::None(()));
    if none.is_none() {
        'is none !'.print();
    }
}
```

The `plus_one` function implements this functionality and can add additional logic for empty values inside the function or handle empty cases based on the return value at the call site.

## Rules for Using Match Patterns
**First rule**: Match must cover all possibilities.

```
use option::OptionTrait;
use debug::PrintTrait;

fn plus_one(x: Option<u8>) -> Option<u8> {
    match x {
        Option::Some(i) => Option::Some(i + 1_u8), 
    }
}

fn main() {
    let five: Option<u8> = Option::Some(5_u8);
    let six = plus_one(five);
    let none = plus_one(Option::None(()));
}
```

The code above does not handle the case where the value is `None`, so it will result in a compilation error.

**Second rule**: Cairo currently only has a very simple default effect.

```
use option::OptionTrait;
use debug::PrintTrait;

fn match_default(x: felt252) -> felt252 {
    match x {
        0 => 'zero', 
        _ => 'default',
    }
}

fn main() {
    let r = match_default(0);
    r.print();
    let r = match_default(1);
    r.print();
}

```
