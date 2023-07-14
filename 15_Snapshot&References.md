# 15_Snapshot and References in Cairo

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

In the previous article [14_Cairo Variable Ownership](./14_Variable_Ownership.md), we mentioned the Copy trait. When an object that implements the Copy trait is passed as a function parameter, a copy of the variable is automatically made and passed into the function. If the variable does not implement the Copy trait, a move operation occurs when it is passed into the function. However, in programming, we often want to retain ownership of a variable in a context without making a copy. This is where snapshots and references come in.

## Snapshot

A snapshot is the value of a variable at a certain point in time. It is immutable and read-only, making it suitable for passing as a parameter to a function without modifying the parameter (similar to a `view` function in a contract).

### Basic usage
In Cairo, snapshots are used with the `@` and `*` operators. The `@` operator is used to obtain a snapshot, and the `*` operator is used to read the value of the snapshot. For example:

```
use debug::PrintTrait;

#[derive(Drop, Copy)]
struct Student {
    name: felt252,
    age: u8,
}

fn print_student(s: @Student) {
    // s.name.print(); // Compilation error
    (*s).name.print();
}

fn main() {
    let mut s = Student { name: 'sam', age: 17 };

    let snapshot01 = @s;
    // Change the value of `s` to see if `snapshot01` changes
    s.name = 'tom';
    print_student(snapshot01);
    print_student(@s);
}
```

In the above code, we created a mutable variable `s` of the `Student` struct, obtained `snapshot01` using the `@` operator, modified the `name` field of `s`, and printed both `snapshot01` and the `name` field of the latest snapshot using the `*` operator. The output is "sam" for `snapshot01` and "tom" for the latest snapshot. This shows that the snapshot does not change with the changes made to the original variable.

Note that the `*` operator can only be used to read the value of a snapshot if the type implements the `Copy` trait.

Also, in the chained call `(*s).name.print();`, the `*` operator has the lowest precedence, so parentheses are needed to first get the value of the snapshot.

### Arrays and Snapshots

In [12_Array in Cairo], we mentioned that arrays do not implement the `Copy` trait, so they cause a move operation when used as function arguments. Also, types that do not implement `Copy` cannot use the `*` operator to read values from snapshots. However, we can still use snapshots to avoid moving arrays. For example:

```
use debug::PrintTrait;
use array::ArrayTrait;

fn use_array(arr: @Array<usize>) {
    arr.len().print();
    let v_z = *arr.at(0);
    v_z.print();
}

fn main(){
    let mut arr = ArrayTrait::<usize>::new();
    arr.append(9);

    use_array(@arr);
}
```

Arrays as a whole cannot use the `*` operator, but their elements can. Fortunately, `at` returns a snapshot of the element, so we can extract the element from the array.

A remarkable point is that the `[]` operator can replace `at` to get a snapshot of the array. Here's an example:

```
use debug::PrintTrait;
use array::ArrayTrait;

fn use_array(arr: @Array<usize>) {
    arr.len().print();
    // Here, *arr[0] replaces *arr.at(0)
    let v_z = *arr[0];
    v_z.print();
}

fn main(){
    let mut arr = ArrayTrait::<usize>::new();
    arr.append(9);

    use_array(@arr);
}
```

This code still compiles, but note that this only applies to snapshots of the array, not the array itself.

Additionally, the get member function of the array seems to be unusable. The following code will cause a compilation error:

```
use debug::PrintTrait;
use array::ArrayTrait;
use option::OptionTrait;

fn use_array(arr: @Array<usize>) {
    match arr.get(0) {
        Option::Some(v) => {
            // We cannot use * to read the value here
            let tem = *v.unbox();
            tem.print();
        },
        Option::None(_) => {}
    }
}

fn main() {
    let mut arr = ArrayTrait::<usize>::new();
    arr.append(9);

    use_array(@arr);
}
```

## References

When we need to modify the value of a parameter in a function, while preserving the context when calling the function, we need to use references. Here's how to use them:

```
use core::debug::PrintTrait;

#[derive(Copy, Drop)]
struct Rectangle {
    width: felt252,
    high: felt252,
}

fn setWidth(ref r: Rectangle, new_width: felt252) {
    r.width = new_width;
}

fn main() {
    let mut r = Rectangle { width: 100, high: 200 };
    setWidth(ref r, 300);
    r.width.print();
}
```

The width printed in the above code is the value set in the `setWidth` function. As we can see, even if the variable `r` is passed to the `setWidth` function, it does not affect the value of the `r` variable read by the `main` function, and the printed value is the one set by the `setWidth` function.

References are specified using the `ref` keyword and need to be specified when defining the function and when passing the argument to indicate that a reference is being passed. Also, variables marked with `ref` must be mutable.

In the Cairo_book, it is written that references are actually a shorthand for two move operations, which means that the passed variable is moved to the called function and then implicitly moved back with ownership.

### An example of using a reference with an array

```
use debug::PrintTrait;
use array::ArrayTrait;

fn main() {
    let mut arr0 = ArrayTrait::new();
    fill_array(ref arr0);
    // This prints the values added by fill_array function
	arr0.print();
}

fn fill_array(ref arr0: Array<felt252>) {
    arr0.append(22);
    arr0.append(44);
    arr0.append(66);
}
```
