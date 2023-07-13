# 00_The entry point of a Cairo program

> This article uses Cairo compiler version 2.0.0-rc0. As Cairo is rapidly evolving, the syntax of different versions may vary slightly, and the content of the article will be updated to a stable version in the future.

## The entry point of a single-file Cairo program

Like most programming languages, the entry point of a single-file Cairo program is the `main` function.

```
use debug::PrintTrait;

const ONE_HOUR_IN_SECONDS: felt252 = 3600;

fn main(){
    ONE_HOUR_IN_SECONDS.print();
}
```

Run this commond:

```
cairo-run $file_path
```

The main function can have a return value, as shown below:

```
fn main() -> felt252 {
   return 10; 
}
```

The return value will be output in the square brackets on this line:

```
Run completed successfully, returning [10]
```

## Starknet smart contract entry point

To define a Starknet smart contract entry point, begin with #[starknet::contract] and add the contract name after the mod keyword.

```
#[starknet::contract]
mod ERC20 {
	...
}
```
