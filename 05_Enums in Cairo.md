# 05_Enums in Cairo

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

Enums in Cairo are a set of enumerated types, or multiple subtypes sharing the same enumerated type. They are suitable for scenarios where there are a group of types with common characteristics, but each type has many differences and is mutually exclusive at the same time.

## Basic usage

For example, there are two versions of IP, IPV6 & IPV4. These two versions are also the version numbers of IP, and the same value can only be one of the two types. In this case, enums can be used. The definition is as follows.

```
enum IpAddrKind {
    V4:(),
    V6:(),
}
```

Instantiating variables using enums:

```
let four = IpAddrKind::V4(());
let six = IpAddrKind::V6(());
```

Both V4 and V6 can be considered as one type IpAddrKind when passed as function parameters.

```
#[derive(Drop)] 
enum IpAddrKind {
    V4: (),
    V6: (),
}

fn route(ip_kind: IpAddrKind) {}

fn main() {
    route(IpAddrKind::V4(()));
    route(IpAddrKind::V6(()));
}
```

## Adding types to enums

Like Rust, we can add various types to an enum, and each element in the enum is like an alias for a type.

```
#[derive(Drop)]
enum Message {
    Quit: (),
    Echo: felt252,
    Move: (usize, usize),
    ChangeColor: (u8, u8, u8)
}

fn route(msg: Message) {}

fn main() {
    route(Message::Quit(()));
    route(Message::Echo(20));
    route(Message::Move((32_usize, 32_usize)));
    route(Message::ChangeColor((8_u8, 8_u8, 8_u8)));
}
```

Above, felt252 and tuples are added to the Message enum, and the corresponding type variables are used as parameters for route.

## Adding impl to enums

We can add impl to a struct to define functions, and we can do the same with enums. The code example is as follows:

```
use debug::PrintTrait;

#[derive(Drop)]
enum Message {
    Quit: (),
    Echo: felt252,
    Move: (usize, usize),
    ChangeColor: (u8, u8, u8)
}

trait Processing {
    fn process(self: Message);
}

impl ProcessingImpl of Processing {
    fn process(self: Message) {
        match self {
            Message::Quit(()) => {
                'quitting'.print();
            },
            Message::Echo(value) => {
                value.print();
            },
            Message::Move((x, y)) => {
                'moving'.print();
            },
            Message::ChangeColor((x, y, z)) => {
                'ChangeColor'.print();
            },
        }
    }
}
```

This brings great versatility to data types.
