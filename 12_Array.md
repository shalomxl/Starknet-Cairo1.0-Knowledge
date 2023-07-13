# 12_Array in Cairo

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

Arrays are a commonly used data structure, typically representing a collection of data elements of the same type. Whether it is a traditional executable program or a smart contract, arrays are frequently used.

## Basic introduction

Arrays in Cairo are a type exported from the core library `array`, and have many different features:

1. Due to the special memory model of Cairo, once a memory space is written, it cannot be overwritten, so the elements in the Cairo array cannot be modified, only read. This is different from most programming languages.
2. An element can be added to the end of the array.
3. An element can be removed from the front of the array.

## Creating arrays

All arrays are mutable variables, so the `mut` keyword is needed:

```
fn create_array() -> Array<felt252> {
    let mut a = ArrayTrait::new(); 
    a.append(0);
    a.append(1);
    a
}
```

An array can contain elements of any type, since the `Array` type is a generic variable. When creating an array, we need to specify its type.

```
use array::ArrayTrait;

fn main() {
    let mut a = ArrayTrait::new();
    // error: Type annotations needed.
}
```

In the above code, the compiler does not know what kind of data should be stored in the a array, so an error is reported. We can specify the type as follows:

```
use array::ArrayTrait;

fn main() {
    let mut a = ArrayTrait::new();
    a.append(1);
}
```

By adding a felt252 type data to the array, we specify that the array is of type Array<felt252>. We can also specify the type as follows:

```
use array::ArrayTrait;

fn main() {
    let b = ArrayTrait::<usize>::new(); 
}
```

Both of the above methods work.

## Reading array size information
You can read the length of an array using `len()`, and check if an array is empty using `is_empty()`.

```
use array::ArrayTrait;
use debug::PrintTrait;
use option::OptionTrait;

fn main() {
    let mut a = ArrayTrait::new();

    a.append(1);

    // Check if the array is empty
    a.is_empty().print();

    // View the length of the array
    a.len().print();
}
```

## Adding & Removing Elements

As mentioned earlier, arrays in Cairo can only add elements to the end and remove elements from the front. Let's take a look at some example code:

```
use array::ArrayTrait;
use debug::PrintTrait;
use option::OptionTrait;

fn main() {
    let mut a = ArrayTrait::new();

	// Add an element to the end
    a.append(1);
    
    // Remove the first element
    let k = a.pop_front();
    k.unwrap().print();
}
```

Adding an element to the end is relatively simple. When removing the first element using the pop_front method, the deleted element is returned as an Option type value. Here, the unwrap method is used to convert the Option type value to the original type.

## Accessing Array Elements

There are two methods to access elements in an array: get function and at function.

### get function

The get function is a relatively safe option, as it returns an Option type value. If the accessed index is within the range of the array, it returns Some. If it is out of range, it returns None. In this way, we can use the Match pattern to handle these two cases separately and avoid reading elements beyond the index range.

```
use array::ArrayTrait;
use debug::PrintTrait;
use option::OptionTrait;
use box::BoxTrait;

fn main() {
    let mut a = ArrayTrait::new();

    a.append(1);
    let s = get_array(0,a);
    s.print();
}

fn get_array(index: usize, arr: Array<felt252>) -> felt252 {
	// The index is of type usize.
    match arr.get(index) {
        Option::Some(x) => {
	        // The * symbol is used to obtain the original value from the copy.
	        // Since the return value is BoxTrait, we need to use unbox to unwrap it.
            *x.unbox()
        },
        Option::None(_) => {
            panic(arr)
        }
    }
}
```

The `BoxTrait` mentioned above will be explained when we discuss the official core library in the future. For information on copies and references, see the Value Passing and Reference Passing sections in Cairo 1.0.

### `at` function

The `at` function will directly return a snapshot of the corresponding indexed element. Note that this is only a snapshot of a single element, so we need to use the `*` operator to extract the value behind this snapshot. Additionally, if the index is out of range, it will result in a panic error, so use it with caution.

```
use array::ArrayTrait;
use debug::PrintTrait;
use option::OptionTrait;

fn main() {
    let mut a = ArrayTrait::new();

    a.append(100);
    let k = *a.at(0);
    k.print();
}
```

## snap function

The snap function will obtain a snapshot object of the array, which is very useful in read-only scenarios.

```
use core::array::SpanTrait;
use array::ArrayTrait;

fn main() {
    let mut a = ArrayTrait::new();
    a.append(100);
    let s = a.span();
}
```

## Arrays as Function Parameters

Arrays do not implement the Copy trait, so when an array is used as a function parameter, a move operation will occur and ownership will change.

```
use array::ArrayTrait;
fn foo(arr: Array<u128>) {}

fn bar(arr: Array<u128>) {}

fn main() {
    let mut arr = ArrayTrait::<u128>::new();
    foo(arr);
    // bar(arr);
}
```

If you uncomment `bar(arr)` above, an error will occur.

## Deep Copy of Arrays

As the name suggests, deep copy is the process of completely copying all the elements, properties, and nested child elements and properties of an object to form a new object. The new object has the same data as before, but its memory address is different, and they are two objects with consistent data.

```
use array::ArrayTrait;
use clone::Clone;

fn foo(arr: Array<u128>) {}

fn bar(arr: Array<u128>) {}

fn main() {
    let mut arr = ArrayTrait::<u128>::new();
    let mut arr01 = arr.clone();
    foo(arr);
    bar(arr01);
}
```

The above code continues the previous example. arr01 is a new array deep copied from arr, and cloning requires the Clone trait from the official library.

As we can see from the definition of deep copy, it is very resource-intensive. The clone member method uses a loop to copy each element of the array one by one. So when executing this Cairo file, you need to specify the gas:

```
cairo-run --available-gas 200000 $cairo_file
```

## Summary

The core library exports an array type and related functions, making it easy to get the length of the array you are working with, add elements, or get the element at a specific index. It is particularly interesting to use the `ArrayTrait::get()` function, as it returns an Option type, which means that if you try to access an index that is out of bounds, it will return None instead of exiting the program, allowing you to implement error management features. In addition, you can use generic types with arrays, making arrays easier to use than the old way of manually managing pointer values.

### Summary of Array Member Functions

```
trait ArrayTrait<T> {
	// Create an array
    fn new() -> Array<T>;
    
	// Add an element to the end of the array
    fn append(ref self: Array<T>, value: T);
    
    // Remove the first element of the array and return it as an option
    fn pop_front(ref self: Array<T>) -> Option<T> nopanic;
    
    // Get an option value for a certain index
    fn get(self: @Array<T>, index: usize) -> Option<Box<@T>>;
    
    // Get a value for a certain index
    fn at(self: @Array<T>, index: usize) -> @T;
    
    // Get the length of the array
    fn len(self: @Array<T>) -> usize;
    
    // Check if the array is empty
    fn is_empty(self: @Array<T>) -> bool;
    
    // Get a snapshot
    fn span(self: @Array<T>) -> Span<T>;
}
```

Keep up the good work! 💪
