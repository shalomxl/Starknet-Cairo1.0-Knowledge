# 14_Variable Ownership in Cairo

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

## Variable Scope

Variable scope, also known as variable ownership scope, typically refers to the range of validity or accessibility of a variable, which determines its lifetime and visibility. Let's take an example:

```
fn main() {
	...
    {
        // Cannot access before variable declaration
        let mut v1 = 1; // Variable declaration statement
        // Accessible after variable declaration, but within the variable's scope
        v1 += 1;
    } // End of the braces, variable scope ends, v1 variable cannot be accessed anymore
    ...
}
```

In the example below, variable v1 is created within the braces of the main function, so the scope of v1 is from its creation to the end of the braces. This applies to braces used in if, loop, and fn statements as well.

## Origin of Variable Ownership

In programming, there are many cases where variables need to be passed around, such as passing a variable as a parameter to a function. This creates a phenomenon where variables can move between multiple scopes (note: this is a phenomenon, not a violation of variable scope rules).

There are two ways to implement this phenomenon:

1. **Passing by copy (passing by value)**. Copying a variable's value to a function or a data structure container requires a copy operation. For an object, a deep copy is required for safety, which can cause performance issues.
2. **Passing the object itself (passing by reference)**. Passing by reference does not require consideration of the cost of copying an object, but it requires consideration of the problem of multiple references to the object after it is passed. For example, we write a reference to an object into an array or pass it to another function. This means that everyone has control over the same object, and if one person releases the object, the others will suffer. Therefore, reference counting rules are generally used to share an object.

The concept of variable ownership in Cairo evolved from the idea of control mentioned above.

## Rules of Variable Ownership in Cairo

In Cairo, the concept of "ownership" is reinforced, and there are three ironclad rules for variable ownership in Cairo:

1. Every value in Cairo has an **owner**.
2. There can only be one owner at a time.
3. When a variable goes out of scope, the variable will be dropped.

To appreciate the impact of these three rules on code, consider the following Cairo code: 

```
use core::debug::PrintTrait;

struct User {
    name: felt252,
    age: u8
}

// takes_ownership takes ownership of the parameter passed to it, and does not return it, so the variable cannot leave the function
fn takes_ownership(s: User) {
    s.name.print();
} // Here, variable s goes out of scope and drop method is called. The memory is freed.

// gives_ownership gives ownership of the return value to the function that calls it
fn give_ownership(name: felt252, age: u8) -> User {
    User{name, age}
}

fn main() {
    // gives_ownership gives the return value to s
    let s = gives_ownership();
    
    // Ownership is transferred to the takes_ownership function, and s is no longer available
    takes_ownership(s);
    
    // If the following code is compiled, an error will occur because s1 is not available
    // s.name.print();
}
```

Moving ownership of an object to another object is called a move. This move approach is very effective in terms of both performance and safety, and the Cairo compiler will help you detect errors when using variables that have had their ownership moved.

Note: Basic types (such as felt252 and u8) have been implemented with the Copy Trait, so there is no move involved when using them. Therefore, to demonstrate the effect of move, a struct without the Copy Trait was used as a parameter for the `takes_ownership` function.

### Struct fields can also be moved individually

Struct fields can be moved, so code that may seem normal in other languages may result in a compilation error in Cairo:

```
use core::debug::PrintTrait;

#[derive(Drop)]
struct User {
    name: felt252,
    age: u8,
    school: School
}

#[derive(Drop)]
struct School {
    name: felt252
}

fn give_ownership(name: felt252, age: u8, school_name: felt252) -> User {
    User { name: name, age: age, school: School { name: school_name } }
}

fn takes_ownership(school: School) {
    school.name.print();
}

fn use_User(user: User) {}

fn main() {
    let mut u = give_ownership('hello', 3, 'high school');
    takes_ownership(u.school); // Passes the school field of the struct alone, so the school field is moved

    // u.school.name.print(); // This line will cause a compilation error because the school field has been moved
    u.name.print(); // The name field can still be accessed normally

    // use_User(u); // As the school field has been moved, the entire struct cannot be moved anymore

    // If the school field is reassigned, the struct variable can be moved again
    u.school = School{
        name:'new_school'
    };
    use_User(u)
}
```

Struct members can be moved in Cairo, and accessing a moved member will result in a compilation error. However, other members of the struct can still be accessed.

If a field is moved, the entire struct cannot be moved anymore. However, if the moved field is reassigned, the struct variable can be moved again.

## Copy trait

The Copy trait is a feature in Cairo that enables value copying. When a type implements the Copy trait, a copy of the value is passed as a function argument. To add the Copy trait to a type, use the following syntax:

```
use core::debug::PrintTrait;

#[derive(Copy,Drop)]
struct User {
    name: felt252,
    age: u8
}

fn give_ownership(name: felt252,age: u8)->User{
    User{name,age}
}

fn takes_ownership(s: User) {
    s.name.print();
}

fn main() {
    let s = give_ownership('hello',3);
    s.age.print();
    takes_ownership(s);
    s.name.print(); // Since the User type implements the Copy trait, the ownership of `s` is not transferred, and it can still be accessed.
}
```

When using the Copy trait, there are some limitations to keep in mind. If a type contains fields that do not implement the Copy trait, the type cannot be given the Copy trait.

## Drop trait

The Drop trait contains a method that can be thought of as a destructor, which is the counterpart to the constructor. The destructor is automatically called when an object is destroyed, to clean up the resources or state that the object has occupied. In contrast, the constructor is used to initialize the object's state.

As mentioned earlier, when a variable goes out of scope, it means the end of its lifetime, and the Drop trait is used to clean up the resources that the variable has occupied. If a type does not implement the Drop trait, the compiler will catch this and throw an error. For example:

```
use core::debug::PrintTrait;

// #[derive(Drop)]
struct User {
    name: felt252,
    age: u8
}

fn give_ownership(name: felt252,age: u8)->User{
    User{name,age}
}

fn main() {
    let s = give_ownership('hello',3);
    //  ^ error: Variable not dropped.
}
```

The above code will result in a compilation error. If the `#[derive(Drop)]` attribute is uncommented, the User type is given the Drop trait, and it can then be dropped without causing a compilation error. In addition, scalar types implement the Drop trait by default.

## Destruct trait

The Destruct trait is currently only used in dictionaries. Dictionaries cannot implement the Drop trait, but by implementing the Destruct trait, they can automatically call the `squashed` method to free memory when they go out of scope. When defining a struct that contains a dictionary, you need to be careful, as shown in the following example:

```
use dict::Felt252DictTrait;
use traits::Default;

#[derive(Destruct)]
struct A{
    mapping: Felt252Dict<felt252>
}

fn main(){
    A { mapping: Default::default() };
}
```

The structure A needs to specify that it implements the Destruct trait, as it cannot implement the Drop trait due to the Felt252Dict. 

## Summary

When passing a variable as a function parameter, either a copy of the value is passed, or the variable itself is passed, and ownership is transferred, changing the scope. Similarly, a function's return value can also transfer ownership.

When a variable goes out of scope, it is either dropped or destructed.

These actions correspond to the Copy, Drop, and Destruct traits, respectively.

Some of the content in this article was based on the work of another author, and we pay tribute to him. 🫡
