
# Cairo1\.0文件基本结构<a name="heading-1"></a>

## 程序入口<a name="heading-2"></a>
需要一个 main 函数作为程序入口。


```
fn main() {
	
}
```


main函数可以有返回值，也可以没有，但是不可以有参数。


```
fn main() -> felt252{

}
```


- [x] 有返回值和没返回值有什么区别？答：有返回值可以在程序运行结束输出返回的结果。

## 官方库的使用<a name="heading-3"></a>
一下是打印库的例子：


```
// 导入
use debug::PrintTrait;

fn main() {
    let x = 5;
    // 变量中自动包含库中的成员函数
    x.print();
}
```


## 常量<a name="heading-4"></a>
常量使用关键字 const 进行声明，并且也需要显试指定数据类型


```
const NUMBER:felt252 = 3;
// 未指定类型，编译报错
const SMALL_NUMBER = 3_u8;
// 正确写法
const SMALL_NUMBER:u8 = 3_u8;
```

<br>

## 变量<a name="heading-5"></a>
声明变量的时候不可以只声明类型不赋值


```
fn main() {
	// 编译报错
    let x: felt252;
    x = 10;
    x.print();
}
```


### 变量的可变性<a name="heading-6"></a>
Cairo1沿用了Rust的规则。

Rust里的变量声明默认是“不可变的”，如果你声明一个变量 `let x = 5;`  变量 `x` 是不可变的，也就是说，`x = y + 10;` 编译器会报错的。如果你要变量的话，你需要使用 `mut` 关键词，也就是要声明成 `let mut x = 5;` 表示这是一个可以改变的变量。这个是比较有趣的，因为其它主流语言在声明变量时默认是可变的，而Rust则是要反过来。这可以理解，不可变的通常来说会有更好的稳定性，而可变的会代来不稳定性。所以，Rust应该是想成为更为安全的语言，所以，默认是 immutable 的变量。当然，Rust同样有 `const` 修饰的常量。于是，Rust可以玩出这么些东西来：

* 常量：`const LEN:u32 = 1024;` 其中的 `LEN` 就是一个`u32` 的整型常量（无符号32位整型），是编译时用到的。
* 可变的变量： `let mut x = 5;` 这个就跟其它语言的类似， 在运行时用到。
* 不可变的变量：`let x= 5;` 对这种变量，你无论修改它，但是，你可以使用 `let x = x + 10;` 这样的方式来重新定义一个新的 `x`。这个在Rust里叫 Shadowing ，第二个 `x`  把第一个 `x` 给遮蔽了。

不可变的变量对于程序的稳定运行是有帮助的，这是一种编程“契约”，当处理契约为不可变的变量时，程序就可以稳定很多，尤其是多线程的环境下，因为不可变意味着只读不写，其他好处是，与易变对象相比，它们更易于理解和推理，并提供更高的安全性。有了这样的“契约”后，编译器也很容易在编译时查错了。这就是Rust语言的编译器的编译期可以帮你检查很多编程上的问题。

对于标识不可变的变量，在 C/C++中我们用`const` ，在Java中使用 `final` ，在 C#中使用 `readonly` ，Scala用 `val` ……（在Javascript 和Python这样的动态语言中，原始类型基本都是不可变的，而自定义类型是可变的）。

对于Rust的Shadowing，使用起来可能会带来麻烦。使用同名变量（在嵌套的scope环境下）带来的bug还是很不好找的。一般来说，每个变量都应该有他最合适的名字，最好不要重名。<br>

## 流程控制<a name="heading-7"></a>
关于流程控制有一个很迷惑的现象，首先来看一个正确的例子：


```
fn bigger(a: usize, b: usize) -> usize {
    if a > b {
        a
    }else{
        b
    }
}
```


然后改成：


```
fn bigger(a: usize, b: usize) -> usize {
    if a > b {
        a
    }
    b
}
```


就不行了，我以为Cairo1的 if语句必须要包含所有分支才可以编译成功，但是我改成如下代码又编译成功了：


```
fn bigger(a: usize, b: usize) -> usize {
    if a > b {
        return a;
    }

    b
}
```


编译的错误信息没看明白：


```
error: If blocks have incompatible types: "core::integer::u32" and "()"
 --> forRun.cairo:12:5
    if a > b {
    ^********^
```


## 函数<a name="heading-8"></a>

### 参数 & 返回值<a name="heading-9"></a>
Cairo是一门静态语言，函数的每个参数和返回值都需要显式指定类型。下面代码将会报错：


```
fn add(a: felt252, b) {
    let c = a + b;
    return c;
}

fn main() -> felt252 {
   add(3, 5) 
}
```


两个错误点：<br>
1. 参数 b 没有指定类型
2. 函数 add 没有指定返回值类型，但是确用了return语句将c变量返回

正确代码：


```
fn add(a: felt252, b: felt252) -> felt252{
    let c = a + b;
    return c;
}

fn main() -> felt252 {
   add(3, 5) 
}
```


### 返回语句<a name="heading-10"></a>
可以使用 return 显试返回，也可以使用不加分号的语句返回


```
fn add(a: felt252, b: felt252) -> felt252 {
    a + b
}

fn sub(a: felt252, b: felt252) -> felt252 {
    return a - b;
}

fn main() -> felt252 {
   add(3, 5);
   sub(11, 7)
}
```

<br>
