
# Cairo1\.0 变量所有权&借用\(引用\)<a name="heading-1"></a>
Cairo1.0沿用了Rust的变量所有权系统，我们先来看看Rust的变量所有权系统是怎样的。

编程中，很多情况下，都是把一个对象（变量）传递过来传递过去，在传递的过程中，传的是一份复本，还是这个对象本身，也就是所谓的“传值还是传引用”的被程序员问得最多的问题。

* **传递副本（传值）**。把一个对象的复本传到一个函数中，或是放到一个数据结构容器中，可能需要出现复制的操作。这个复制对于一个**对象**来说，需要深度复制才安全，否则就会出现各种问题，而**深度复制就会导致性能问题**。
* **传递对象本身（传引用）**。
	1. 传引用也就是不需要考虑对象的复制成本，但是需要考虑对象在传递后，会多个变量所引用的问题。比如：我们把一个对象的引用传给一个List或其它的一个函数，这意味着，大家对同一个对象都有控制权，如果有一个人释放了这个对象，那边其它人就遭殃了，所以，一般会采用引用计数的方式来共享一个对象。
	2. 引用除了共享的问题外，还有作用域的问题，比如：你从一个函数的栈内存中返回一个对象的引用给调用者，调用者就会收到一个被释放了个引用对象（因为函数结束后栈被清了）。

这些东西在任何一个编程语言中都是必需要解决的问题，要足够灵活到让程序员可以根据自己的需要来写程序。

## 变量所有权<a name="heading-2"></a>
在Rust中，Rust强化了“所有权”的概念，下面是Rust的所有者的三大铁律：

1. Rust 中的每一个值都有一个被称为其 **所有者**（owner）的变量。
2. 值有且只有一个所有者。
3. 当**所有者**（变量）离开作用域，这个**值**将被丢弃。

为了体会以上三个规则对代码的影响，可以看下面这段Rust代码(不用担心😁，代码和Cairo1很像)：


```
// takes_ownership 取得调用函数传入参数的所有权，因为不返回，所以变量进来了就出不去了
fn takes_ownership(some_string: String) {
    println!("{}", some_string);
} // 这里，some_string 移出作用域并调用 drop 方法。占用的内存被释放

// gives_ownership 将返回值移动给调用它的函数
fn gives_ownership() -> String {
    let some_string = String::from("hello"); // some_string 进入作用域.
    some_string // 返回 some_string 并移出给调用的函数
}

fn main()
{
    // gives_ownership 将返回值移给 s1
    let s1 = gives_ownership();
    // 所有权转给了 takes_ownership 函数, s1 不可用了
    takes_ownership(s1);
    // 如果编译下面的代码，会出现s1不可用的错误
    // println!("s1= {}", s1);
    //                    ^^ value borrowed here after move
}
```


把一个对象的所有权移动到给另外一个对象，这样的操作被称作move。这样的 Move 的方式，在性能上和安全性上都是非常有效的，而Rust的编译器会帮你检查出*使用了所有权被move走的变量的错误*。

### 结构体字段也可以被单独move<a name="heading-3"></a>
结构体字段可以被move，所以在其他语言中看似很正常的代码，在Rust中编译会报错：


```
#[derive(Debug)] // 让结构体可以使用 {:?} 的方式输出
struct Person {
    name :String,
    email:String,
}

let _name = p.name; // 把结构体 Person::name Move掉
println!("{} {}", _name, p.email); //其它成员可以正常访问
println!("{:?}", p); //编译出错 "value borrowed here after partial move"
p.name = "Hao Chen".to_string(); // Person::name又有了。
println!("{:?}", p); //可以正常的编译了
```


结构体中的成员是可以被Move掉的，Move掉的结构实例会成为一个部分的未初始化的结构，如果需要访问整个结构体的成员，会出现编译问题。但是后面把 Person::name补上后，又可以愉快地工作了。

## 变量借用borrow(引用reference)<a name="heading-4"></a>
有些情况不适合将变量所有权转移。

比如，我有一个 `compare(s1: Student, s2: Student) -> bool` 我想比较两个学生的平均份成绩， 我不想传复本，因为太慢，我也不想把所有权交进去，因为只是想计算其中的数据。这个时候，传引用就是一个比较好的选择，Rust同样支持传引用。只需要把上面的函数声明改成：`compare(s1 :&Student, s2 : &Student) -> bool` 就可以了，在调用的时候，`compare (&s1, &s2)`。

- [x] 在Cairo中是如何编写的？答：`compare(ref s1: Student, ref s2 : Student) -> bool`，调用的时候，`compare (ref s1, ref s2)`

在Rust中，引用也称作为借用(borrow)。

### 可变引用<a name="heading-5"></a>
如果你要修改这个引用对象，就需要使用“可变引用”，使用 mut 关键字，如：`foo( s : &mut Student)` 以及 `foo( &mut s);`，这个与变量的可变性关键字是一样的🔗[Cairo1.0类型](brain://api.thebrain.com/zBJh6lmfMEWrNhIe90B2qA/pHb_kPXJRUOCu4Y3NuN74A/Cairo10%E7%B1%BB%E5%9E%8B)

关于可变引用还有一个限制：Rust严格规定了——**在任意时刻，要么只能有一个可变引用，要么只能有多个不可变引用**。

这些严格的规定会导致程序员失去编程的灵活性，不熟悉Rust的程序员可能会在一些编译错误下会很崩溃，但是你的代码的稳定性也会提高，bug率也会降低。

- [x] 在Cairo中可变引用是怎样的？答：在Cairo中，引用不分可变引用和不可变引用，但是被引用的变量一定要是可变变量(用mut关键字)。

### 引用的生命周期<a name="heading-6"></a>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
