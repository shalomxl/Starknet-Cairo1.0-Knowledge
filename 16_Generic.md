# 16_Generics in Cairo

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

Generics are a programming language feature that allows type parameters to be replaced with concrete types when the code is instantiated.

In practical programming, we design algorithms to efficiently solve business problems. Without generics, each type would require a separate copy of the same algorithm. Ideally, algorithms should be independent of data structures and types, with each specialized data type doing its own work, while the algorithm focuses on a standard implementation.

So, generics are cool 😎

In Cairo, generics can be used in functions, structs, enums, and methods in traits.

## Generics in Functions

If a function takes an argument that contains a generic type, the generic type needs to be declared in the `<>` before the argument, and this will be part of the function signature. Here's an example of implementing a function that finds the minimum element in a generic array:

```
use debug::PrintTrait;
use array::ArrayTrait;

// PartialOrd implements comparison between generic variables
fn smallest_element<T, impl TPartialOrd: PartialOrd<T>, impl TCopy: Copy<T>, impl TDrop: Drop<T>>(
    list: @Array<T>
) -> T {
	// The * operator is used here, so T must implement the Copy trait
    let mut smallest = *list[0];

    let mut index = 1;

    loop {
        if index >= list.len() {
            break smallest;
        }
        // Comparison between two generic variables requires PartialOrd to be implemented
        if *list[index] < smallest {
            smallest = *list[index];
        }
        index = index + 1;
    }
}

fn main() {
    let mut list: Array<u8> = ArrayTrait::new();
    list.append(5);
    list.append(3);
    list.append(10);

    let s = smallest_element(@list);
    assert(s == 3, 0);
    s.print();
}
```

You can see that in the generic declaration area, we added many constraints to the generic type `T` (which can be named `T` or any other name). Generics can be any data type that conforms to the generic algorithm, and the `impl` added in the `<>` is a constraint on the generic type passed to this function.

1. First, we took a value from the snapshot of type `T`, so `T` must have implemented the `Copy` trait.
2. Secondly, the type variable `smallest` of type `T` is ultimately returned as the return value of the function and includes both move and drop operations, so it needs to implement the `Drop` trait.
3. Finally, we need to compare the sizes of two generics, so we need to implement the `PartialOrd` trait.

Therefore, in the function declaration, there is a section declaring the generics as follows: `<T, impl TPartialOrd: PartialOrd<T>, impl TCopy: Copy<T>, impl TDrop: Drop<T>>`. When calling this function, all elements in the parameter array must implement the three traits described in the constraints.

## Generics in Structs

Generic fields can also be added to struct elements, such as:

```
struct Wallet<T> {
    balance: T
}

impl WalletDrop<T, impl TDrop: Drop<T>> of Drop<Wallet<T>>;
```

```
#[derive(Drop)]
struct Wallet<T> {
    balance: T
}
```

Both of the above methods should work. The Cairo book states that the second method does not declare the type `T` as implementing the `Drop` trait, but it does not provide any example code. I have experimented several times and have not found any differences yet. I will add more information if I find any in the future.

### Using Generics in Struct Methods

In the following code, we can see that generics need to be declared in the `struct`, `trait`, and `impl` sections. Constraints need to be added in the `impl` section as it stores the algorithm logic.

```
use debug::PrintTrait;

#[derive(Copy,Drop)]
struct Wallet<T> {
    balance: T
}

trait WalletTrait<T> {
    fn balance(self: @Wallet<T>) -> T;
}

impl WalletImpl<T, impl TCopy: Copy<T>> of WalletTrait<T>{
    fn balance(self: @Wallet<T>) -> T{
        *self.balance
    }
}

fn main() {
    let w = Wallet{balance:'100 000 000'};

    w.balance().print();
}
```

Here is an example that uses two different generics simultaneously:

```
use debug::PrintTrait;

#[derive(Copy,Drop)]
struct Wallet<T, U> {
    balance: T,
    address: U,
}

trait WalletTrait<T, U> {
    fn getAll(self: @Wallet<T, U>) -> (T, U);
}

impl WalletImpl<T, impl TCopy: Copy<T>, U, impl UCopy: Copy<U>> of WalletTrait<T, U>{
    fn getAll(self: @Wallet<T, U>) -> (T, U){
        (*self.balance,*self.address)
    }
}

fn main() {
    let mut w = Wallet{
        balance: 100,
        address: '0x0000aaaaa'
    };

    let (b,a) = w.getAll();
    b.print();
    a.print();
}
```

## Generics in Enums

Coincidentally, `Option` is an enum that uses generics:

```
enum Option<T> {
    Some: T,
    None: (),
}
```
