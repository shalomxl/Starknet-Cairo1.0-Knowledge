# 03_Scalar Types in Cairo(felt,integer,boolean,float)

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

## felt252

felt252 is a fundamental type in Cairo, representing a storage slot. If the variable type is not specified, the default type is felt252. felt252 can be a negative number or zero, and its value range is:

```
-X < felt < X, where X = 2^{251} + 17* 2^{192} + 1
```

Any value within this range can be stored in felt252. Note⚠️: it is not `-(2^251) ~ (2^251 - 1)`。

felt252 can store both integers and strings. Here are examples of storing integers and strings 👇:

```
use debug::PrintTrait;

fn main() {
	// The `let` keyword is used to declare variables, and if a literal value is assigned directly, the default type should be `felt252`.
    let x: felt252 = 5;
    let y: felt252 = 'ppppppppppppppppppppppppppp';
    let z: felt252 = 'ppppppppppppppppppppppppppp99999999999999'; // overflow
    x.print();
}
```

> Note⚠️: It is 252 not 256, and if other numbers are appended to felt or if no number is appended, a compilation error will occur. For example, `felt256 felt`.

### Short Strings

Short strings are represented by single quotes and cannot exceed 31 characters in length. Short strings are essentially felt252 types, but are represented as strings. The computer converts characters to numbers using the ASCII protocol. The length of a short string cannot exceed the maximum value of felt252.

```
let mut my_first_initial = 'C';
```

### Differences between felt252 and integers

In Cairo 1.0, felt252 does not support division or modulo operations, while integers do.

The following is a description of felt in **Cairo 0.1**:

> felt stands for "field elements" and can be translated as "field elements". The difference between felt252 and integers is mainly reflected in the division operation. When an operation that cannot be evenly divided occurs, such as 7/3, the result of the integer operation is usually 2, but not for felt252. felt252 will always satisfy the equation x*3 = 7. Since it can only be an integer, the value of x*3 will be a huge integer, which may cause overflow, and the overflow value will be exactly 7.

In Cairo 1.0, division using felt252 is prohibited.

```
use debug::PrintTrait;

fn main() {
    let x = 7 / 3;
    x.print();
}

// The above code will result in the following error:
error: Trait has no implementation in context: core::traits::Div::<core::felt252>
 --> f_felt252.cairo:4:13
    let x = 7 / 3;
            ^***^

Error: failed to compile: src/f_felt252.cairo
```

## Integers

The core library includes these integer variables: u8, u16, u32 (usize), u64, u128, and u256. They are all implemented using felt252 and come with integer overflow detection. If the variable type is not specified when declaring an integer variable, the default type is felt252, as shown in the following code:q

```
let y = 2;
```

To specify u8, u16, and other types, the variable type must be indicated:

```
let x:u8 = 2;
```

The u256 type is more complex and requires the use of other types to construct. u256 is a struct in the core library, with two fields high and low both of type u128, because one storage slot (felt252) cannot hold u256 data, so it needs to be split into two storage slots for storage.

```
let z: u256 = u256 { high: 0, low: 10 }
```

This involves the concepts of **high-order** bits and **low-order** bits, which can be further researched if interested.

### Operators

Integers support most operators and come with overflow detection, supporting the following:

```
fn test_u8_operators() {
	// Arithmetic Operators
    assert(1_u8 + 3_u8 == 4_u8, '1 + 3 == 4');
    assert(3_u8 + 6_u8 == 9_u8, '3 + 6 == 9');
    assert(3_u8 - 1_u8 == 2_u8, '3 - 1 == 2');
    assert(1_u8 * 3_u8 == 3_u8, '1 * 3 == 3');
    assert(2_u8 * 4_u8 == 8_u8, '2 * 4 == 8');
    assert(19_u8 / 7_u8 == 2_u8, '19 / 7 == 2');
    assert(19_u8 % 7_u8 == 5_u8, '19 % 7 == 5');
    assert(231_u8 - 131_u8 == 100_u8, '231-131=100');

	// Comparison Operators
    assert(1_u8 == 1_u8, '1 == 1');
    assert(1_u8 != 2_u8, '1 != 2');
    assert(1_u8 < 4_u8, '1 < 4');
    assert(1_u8 <= 4_u8, '1 <= 4');
    assert(5_u8 > 2_u8, '5 > 2');
    assert(5_u8 >= 2_u8, '5 >= 2');
    assert(!(3_u8 > 3_u8), '!(3 > 3)');
    assert(3_u8 >= 3_u8, '3 >= 3');
}
```

In addition, `u256` also supports these operators:

```
use debug::PrintTrait;

fn main() {
    let x:u256 = u256{high:3, low: 3};
    let y:u256 = u256{high:3, low: 3};

    let z = x + y;
    assert(z == 2 * y, 'z == 2 * y');
    assert(0 == x - y, '0 == x - y');
    assert(1 == x / y, '0 == x - y');
    assert(0 == x % y, '0 == x % y');

    assert(x == y, 'x == y');
    assert(x <= y, 'x <= y');
    assert(x >= y, 'x <= y');
    assert(x - 1 < y, 'x - 1 < y');
    assert(x + 1 > y, 'x + 1 >= y');
    assert(x != y - 1, 'x != y');
}
```

## Boolean

You can declare `boollean` using the following syntax:

```
let is_morning:bool = true;
let is_evening:bool = false;
```
