# 09_Flow Control in Cairo

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

## If statement
```
use debug::PrintTrait;

fn main() {
    let number = 3;

    if number == 3 {
        'condition was true'.print();
    } 
}
```

The if statement is easy to use and does not require parentheses to enclose the condition.

Let's take a look at the case of multiple conditions:

```
use debug::PrintTrait;

fn main() {
    let number = 3;

    if number == 12 {
        'number is 12'.print();
    } else if number == 3 {
        'number is 3'.print();
    } else if number - 2 == 1 {
        'number minus 2 is 1'.print();
    } else {
        'number not found'.print();
    }
}
```

The execution order of multiple conditions is from top to bottom. Once a condition is met, the execution will stop and the subsequent conditions will not be evaluated. In the example above, the second condition `number == 3` is already met, so the third condition, even if it is true, will not be executed.

### Special if statement that achieves the same effect as the ternary operator

This statement combines the let statement and the if statement together.

```
use debug::PrintTrait;

fn main() {
    let condition = true;
    let number = if condition {5} else {6};

    if number == 5 {
        'condition was true'.print();
    }
}
```

In the above code, if condition is true, number will be created as 5; if condition is false, number will be created as 6.

The ternary operator in Solidity looks like this:

```
bool condition = true;
uint256 a = condition ? 5 : 6;
```

## Loop Statement

loop can make the loop continue indefinitely, and control the loop using `continue` and `break`. Let's first look at an example:

```
use debug::PrintTrait;

fn main() {
    let mut i: usize = 0;
    loop {
        i += 1;

        if i < 10 {
            'again'.print();
            continue;
        }

        if i == 10 {
            break ();
        };
    }
}
```

In the code above, `i` is incremented, and when it is less than 10, it will use the `continue` instruction to directly enter the next iteration, and the logic after `continue` will not be executed; when `i` is equal to 10, it will use the `break` instruction to exit the loop.

* A return value needs to be added after `break`, and if there is no return value, the unit type `()` should be used, but it cannot be omitted.

Note⚠️: When executing Cairo code with `loop`, the `--available-gas` option needs to be used to specify the gas limit. For example:

```
cairo-run --available-gas 200000 $CairoFile
```

### Getting the return value of loop

As mentioned earlier, break must be accompanied by a return value, which we can get:

```
use debug::PrintTrait;

fn main() {
    let mut i: usize = 0;
    let t = loop {
        i += 1;

        if i >= 10 {
            break i;
        };
    };

    t.print();
}
```

The final value of t in the code above is 10.

## Commonality of if and loop: Getting the calculation result of an expression

Both if and loop can be combined with let to obtain the calculation result of an expression, which is the return value of the code inside the curly braces. As mentioned above:

```
let number = if condition {5} else {6};
```

and

```
let t = loop {
    i += 1;

    if i >= 10 {
        break i;
    };
};
```

In Cairo, a common way to obtain the calculation result of an expression is to directly obtain the return value of the code block inside `{}`.

```
use debug::PrintTrait;

fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    y.print();
}
```

We can understand the combined usage of if, loop, and let as syntactic sugar based on the common way of obtaining the calculation result of an expression.
