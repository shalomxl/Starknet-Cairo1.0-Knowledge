# 17_Traits in Cairo

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

We have already written a lot of code that uses traits. Now let's summarize how to use traits.

The literal meaning of "trait" is "characteristic," which is equivalent to the "interface" in other programming languages. A trait can be used to define a set of methods that represent the specific features of that trait. Any type can implement all the methods of a trait, which means the type has the characteristic of that trait.

## Basic Usage

When a struct implements a trait, it needs to implement all the methods in the trait, otherwise it cannot be considered as implementing that trait and will cause a compilation error. Let's look at an example:

```
use debug::PrintTrait;

#[derive(Copy, Drop)]
struct Rectangle {
    width: u64,
    high: u64
}

trait ShapeGeometry {
    fn new(width: u64, high: u64) -> Rectangle;
    fn boundary(self: @Rectangle) -> u64;
    fn area(self: @Rectangle) -> u64;
}

// Here we implement the logic for the three functions
impl RectangleGeometryImpl of ShapeGeometry {
    fn new(width: u64, high: u64) -> Rectangle {
        Rectangle { width, high }
    }

    fn boundary(self: @Rectangle) -> u64 {
        2 * (*self.high + *self.width)
    }
    fn area(self: @Rectangle) -> u64 {
        *self.high * *self.width
    }
}

fn main() {
	// Here we directly use the `impl` to call the `new` method
    let r = RectangleGeometryImpl::new(10, 20);
    
    // Here we use the struct to call the `boundary` and `area` methods
    r.boundary().print();
    r.area().print();

    // Here we use the `impl` to directly call the `area` method
    RectangleGeometryImpl::area(@r).print();
}
```

Above, we defined a structure named `Rectangle`, followed by a trait named `ShapeGeometry`, which includes the signatures of three methods: `new`, `boundary`, and `area`. To implement the `ShapeGeometry` trait for the `Rectangle` structure, we need to write the logic for all three functions in the `impl` section.

Additionally, we can see in the `main` function that we can also **directly call member functions using `impl`**.

## Generic Traits

In the above example, the `Rectangle` type is explicitly specified in the trait, which means that this trait can only be implemented by the `Rectangle` structure. If multiple types share the same trait features, we need to use a generic trait.

```
use debug::PrintTrait;

#[derive(Copy, Drop)]
struct Rectangle {
    height: u64,
    width: u64,
}

#[derive(Copy, Drop)]
struct Circle {
    radius: u64
}

// This generic trait can be implemented by multiple structs
trait ShapeGeometryTrait<T> {
    fn boundary(self: T) -> u64;
    fn area(self: T) -> u64;
}

// Implemented by the Rectangle type
impl RectangleGeometryImpl of ShapeGeometryTrait<Rectangle> {
    fn boundary(self: Rectangle) -> u64 {
        2 * (self.height + self.width)
    }
    fn area(self: Rectangle) -> u64 {
        self.height * self.width
    }
}

// Implemented by the Circle type
impl CircleGeometryImpl of ShapeGeometryTrait<Circle> {
    fn boundary(self: Circle) -> u64 {
        (2 * 314 * self.radius) / 100
    }
    fn area(self: Circle) -> u64 {
        (314 * self.radius * self.radius) / 100
    }
}

fn main() {
    let rect = Rectangle { height: 5, width: 7 };
    rect.area().print(); // 35
    rect.boundary().print(); // 24

    let circ = Circle { radius: 5 };
    circ.area().print(); // 78
    circ.boundary().print(); // 31
}
```

Above, we defined the `ShapeGeometryTrait<T>` trait, which is implemented by both the `Rectangle` and `Circle` structures. The same named member methods are used in the `main` function.
