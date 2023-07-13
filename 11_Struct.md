# 11_Struct in Cairo

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

## Basic Usage

Define a struct:

```
#[derive(Copy, Drop)]
struct User {
    active: bool,
    username: felt252,
    email: felt252,
    sign_in_count: u64,
}
```

Creating a struct variable (Note⚠️: When creating a struct variable, all fields need to be assigned a value, otherwise a `error: Missing member` error will be reported during compilation):

```
#[derive(Copy, Drop)]
struct User {
    active: bool,
    username: felt252,
    email: felt252,
    sign_in_count: u64,
}

fn main() {
    let user1 = User {
        active: true, username: 'someusername123', email: 'someone@example.com', sign_in_count: 1
    };
}
```

Using the fields of a struct variable:

```
use debug::PrintTrait;

#[derive(Copy, Drop)]
struct User {
    active: bool,
    username: felt252,
    email: felt252,
    sign_in_count: u64,
}

fn main() {
    let user1 = User {
        active: true, 
        username: 'someusername123', 
        email: 'someone@example.com', 
        sign_in_count: 1
    };

    user1.active.print();
    user1.username.print();
}
```

Modifying the value of a field in a struct variable. The mutability and immutability of a variable also apply to a struct variable. Therefore, to modify a struct variable, the variable needs to be mutable (modified by the `mut` keyword). For example:

```
use debug::PrintTrait;

#[derive(Copy, Drop)]
struct User {
    active: bool,
    username: felt252,
    email: felt252,
    sign_in_count: u64,
}

fn main() {
    let mut user1 = User {
        active: true, 
        username: 'someusername123', 
        email: 'someone@example.com', 
        sign_in_count: 1
    };

    user1.username = 'shalom';
    user1.username.print();
}
```

The `mut` keyword is used to modify the `user1` variable above. Note⚠️: The entire struct variable must be declared mutable at the same time, and it is not allowed to declare only some fields as mutable.

## A convenient way to instantiate a struct variable
In many languages, it is common to use a function specifically to instantiate a struct variable, for example:

```
use debug::PrintTrait;

#[derive(Copy, Drop)]
struct User {
    active: bool,
    username: felt252,
    email: felt252,
    sign_in_count: u64,
}

fn init_user(active: bool, username: felt252, email: felt252, sign_in_count: u64) -> User {
    User { active: active, username: username, email: email, sign_in_count: sign_in_count }
}

fn main() {
    let mut user1 = init_user(true, 'someusername123', 'someone@example.com', 1);

    user1.username = 'shalom';
    user1.username.print();
}
```

In the code above, the `init_user` function is used to instantiate a `User` struct variable.

In this `init_user` function, all parameters have the same names as the fields of the struct, so we can simplify it as follows:

```
fn init_user_simplify(active: bool, username: felt252, email: felt252, sign_in_count: u64) -> User {
	// 这里省略了 `active:` ...
    User { active, username, email, sign_in_count }
}
```

When assigning values to the fields of a `User` struct variable inside a function, you don't need to specify the field names again. You can directly use function parameters with the same names.

## Defining member methods (methods) for a struct

Most advanced programming languages allow defining member methods for structs (or objects), and Cairo is no exception. However, in Cairo, this feature is implemented with the help of Traits. For example:

```
use debug::PrintTrait;

struct Rectangle {
    width: u32,
    high: u32
}

// Declare a trait here
trait GeometryTrait {
    fn area(self: Rectangle) -> u32;
}

// Implement the trait here
impl RectangleImpl of GeometryTrait {
	// Implement the area method in the trait. The first parameter of the method is linked to the struct.
    fn area(self: Rectangle) -> u32 {
        self.width * self.high
    }
}

fn main() {
    let r = Rectangle {
        width: 10,
        high: 2,
    };

    r.area().print();
}
```

The trait declares the signature of the method, which identifies the name, parameters, and return value of the method. The `impl` block contains the actual logic of the method, and **must include the implementation of all methods in the trait**.

Note⚠️: The `impl` code block has strict requirements for its relationship with the struct:

1. **The associated struct must be specified as the first parameter of the method.**
2. **The parameter name must be `self`.**
3. **The `self` variable in an `impl` block must refer to the same struct.**

You can think of an `impl` block as being specific to a particular struct type and needing to implement the logic of all methods in the trait (the meaning of trait is "feature").

### Constructors in Traits

In a trait, not all functions are struct members, and it can also contain functions that do not use the `self` parameter, which are commonly used as constructors for structs. For example:

```
struct Rectangle {
    width: u32,
    high: u32
}

trait RectangleTrait {
    fn square(size: u32) -> Rectangle;
}

impl RectangleImpl of RectangleTrait {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, high: size }
    }
}
```

The `square` function above is a constructor for the `Rectangle` struct, and it does not have a `self` variable. When we need to instantiate a `Rectangle` variable, we can use `let square = RectangleImpl::square(10);` (note that we use `impl` here!).

### Associating a struct with multiple traits

Structs and traits are independent of each other, and the same struct can implement multiple traits. For example:

```
struct Rectangle {
    width: u32,
    high: u32
}

struct Rectangle2 {
    width: u32,
    high: u32
}

trait RectangleCalc {
    fn area(self: Rectangle) -> u32;
}
impl RectangleCalcImpl of RectangleCalc {
    fn area(self: Rectangle) -> u32 {
        (self.width) * (self.high)
    }
}

trait RectangleCmp {
    fn can_hold(self: Rectangle, other: Rectangle) -> bool;
}

impl RectangleCmpImpl of RectangleCmp {
    fn can_hold(self: Rectangle, other: Rectangle) -> bool {
        self.width > other.width && self.high > other.high
    }
}

fn main() {}
```

In the code above, `Rectangle` implements both the `RectangleCalc` and `RectangleCmp` traits.
