# 07_Tuples in Cairo

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

A tuple is an interesting type that many programming languages have. It allows multiple different types to be combined into a collection. Once declared, the number of types it contains can't be increased or decreased, and the types inside it can't be changed.

## Basic Usage

```
use debug::PrintTrait;

fn main() {
    let tup: (u32, u64, bool) = (10, 20, true);
    let (x, y, z) = tup;
    x.print();
}
```

In the above code, a tuple containing three types, `u32, u64, bool`, is created. `let (x, y, z) = tup;` shows how elements are extracted from a tuple.
