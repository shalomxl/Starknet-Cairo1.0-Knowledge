
# Cairo1\.0 Core library 的使用<a name="heading-1"></a>

## traits<a name="heading-2"></a>
用来felt252与整型互转

u8转felt252:


```
use traits::Into;

fn sum_big_numbers(x: u8, y: u8) -> felt252 {
    x.into() + y.into()
}
```


felt252转u8：


```
use traits::TryInto;
use option::OptionTrait;

fn convert_felt_to_u8(x: felt252) -> u8 {
    x.try_into().unwrap()
}
```


try_into()返回的是一个Option对象，需要使用unwrap()转换为u8
<br>

## option<a name="heading-3"></a>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
