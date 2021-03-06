# 字面常量

## 数值字面常量

即有理数。由 0 - 9 和小数点`.`组成。支持 10 进制和 16 进制。16进制需要在最前加上 `0X`,不支持 8 进制。

支持指数符号，例如  `10e2` 代表 10  的 2 次方。

### 数值字面常量类型

Solidity 对每个有理数都有相应的数值字面常量类型，整数字面常量和有理数字面常量都属于数值字面常量类型。

### 字面常量表达式

字面常量表达式是指只包含数值字面常量和运算符的表达式。但"字面常量表达式"只是一个帮助我们理解的概念，并不是 Solidity 中的一个类型。字面常量表达式也属于数值字面常量类型。例如`1+2`与`2+1`的结果跟有理数 `3` 的数值字面常量类型相同。

数值字面常量表达式支持任意精度，这意味着在数值常量表达式中，计算既不会溢出，除法也不会截断。

例如：

- `(2**800 + 1) - 2**800` 的结果是字面常量 `1` ，即时计算的中间步骤已经超过了以太坊虚拟机的机器字节长度；
- `0.5*8`的结果是整型 `4` ，尽管非整型参与了计算；


>在 Solidity 0.4.0 之前，整数字面量的除法也会被截断。例如 `5/2 = 2`。但之后的版本修正了这一逻辑，` 5 / 2 = 2.5`。

### 非字面常量表达式

数值字面常量表达式只要在非字面常量表达式中使用就会转换成非字面常量类型，则需要考虑类型转化相关的逻辑。例如：

```js
uint128 a = 1;
uint128 b = 2.5 + a + 0.5;
// 无法正常计算。加法计算需要将两边的数字转化成同一类型，显然两者无法转成同一类型。
// TypeError: Type rational_const 2 / 5 is not implicitly convertible to expected type uint128. Try converting to type ufixed8x1 or use an explicit conversion.

uint128 c = 2 + a + 3
// 可以成功计算。
```



## 字符串字面常量

字符串字面量只能包含可打印的ASCII字符，也就是意味着介于 0x1F 与 0x7E 之间的字符。

字符串字面常量有两个写法：

```JS
"Hello World"
"hello word" "您好" // 用空格隔开的多个字面量将会被自动合并。
```

### 类型转换

字符串字面量可以隐式地转换成 `bytes1`、`bytes2`、....、`bytes32`，如果合适的话，可以转成 `bytes `及 `string`。

## Unicode 字面量

常规字符串文字只能包含 ASCII，而 Unicode 文字可以包含任何有效的 UTF-8 序列。

```
string memory a = unicode"Hello 😯";
```

## 16进制字面常量

16进制字面量的标识方法为使用 `hex`前缀。其使用方法与 string 类型相似。
