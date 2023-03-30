
# Cairo1\.0类型<a name="heading-1"></a>

## felt252<a name="heading-2"></a>
felt252是Cairo中基础类型，代表一个存储槽，未指定变量类型的变量默认类型都是felt252。felt252可以是负数或者是0，它的取值范围是：


```
-X < felt < X, where X = 2^{251} + 17* 2^{192} + 1
```

并不是 -(2<sup>251</sup>) \~ (2<sup>251</sup> - 1)。任何在这个取值范围内的值都可以存储在felt252中。

下面👇是存储整数和字符串的例子：


```
fn main() {
	// let关键字声明，直接用字面量赋值，默认类型应该是felt252
    let x: felt252 = 5;
    let y: felt252 = 'ppppppppppppppppppppppppppp';
    let z: felt252 = 'ppppppppppppppppppppppppppp99999999999999'; // 溢出
    x.print();
}
```


> 注意⚠️：是252不是256，而且felt后面接其他数字或者不接都会编译报错，例如felt256 felt

### felt252与整型的区别<a name="heading-3"></a>
Cairo1.0中，felt252不支持除法运算，也不支持取余，整型可以。

以下是Cairo 0.1中关于felt的描述：

flet是field elements的缩写，可以翻译为域元素。felt252与整型的区别主要体现在除法运算上。当出现不能整除的运算时，例如 7/3，整型运算的结果通常是 2，但是felt252不是。felt252 会始终满足 x*3 = 7 这个等式，由于只能是整数，所以 x*3 的值将会是一个巨大的整数，以至于溢出，溢出后的值刚好是 7。

## 整型<a name="heading-4"></a>
核心库中包含了这些整型变量： u8, u16, u32 (usize), u64, u128, and u256。他们都是用 felt252 实现的，并且自带整数溢出检测，溢出后触发Panic错误。

使用整型字面量时，需要显式指定字面量类型


```
let x = 2_u8;
```


不指定则默认为felt252类型


```
let y = 2;
```


### 运算符<a name="heading-5"></a>
整型支持大多数运算符，并且自带溢出检测，支持如下：


```
fn test_u8_operators() {
    assert(1_u8 == 1_u8, '1 == 1');
    assert(1_u8 != 2_u8, '1 != 2');
    assert(1_u8 + 3_u8 == 4_u8, '1 + 3 == 4');
    assert(3_u8 + 6_u8 == 9_u8, '3 + 6 == 9');
    assert(3_u8 - 1_u8 == 2_u8, '3 - 1 == 2');
    assert(1_u8 * 3_u8 == 3_u8, '1 * 3 == 3');
    assert(2_u8 * 4_u8 == 8_u8, '2 * 4 == 8');
    assert(19_u8 / 7_u8 == 2_u8, '19 / 7 == 2');
    assert(19_u8 % 7_u8 == 5_u8, '19 % 7 == 5');
    assert(231_u8 - 131_u8 == 100_u8, '231-131=100');
    assert(1_u8 < 4_u8, '1 < 4');
    assert(1_u8 <= 4_u8, '1 <= 4');
    assert(!(4_u8 < 4_u8), '!(4 < 4)');
    assert(4_u8 <= 4_u8, '4 <= 4');
    assert(5_u8 > 2_u8, '5 > 2');
    assert(5_u8 >= 2_u8, '5 >= 2');
    assert(!(3_u8 > 3_u8), '!(3 > 3)');
    assert(3_u8 >= 3_u8, '3 >= 3');
}
```

<br>
<br>
<br>

## <a name="heading-6"></a>
<br>
<br>
<br>

## Boolean<a name="heading-7"></a>
同样需要显试指定类型


```
let is_morning:bool = true;
```


## 短字符串<a name="heading-8"></a>
短字符串用单引号表示，并且长度不能超过31个字符。短字符串本质上是felt252类型，只是表现形式为字符串，计算机通过ASCII协议将字符转换成数字。短字符串长度不能超过31个字符，本质上是不能超过felt252的最大值。


```
let mut my_first_initial = 'C';
```


## 元组tuple<a name="heading-9"></a>
元组使用案例


```
fn main() {
    let cat = ('Furry McFurson', 3);
    let (name, age) = cat; 
    name.print();
    age.print();
}
```

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
