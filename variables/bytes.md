# bytes 与 string

Solidity 中的字节分为定长字节数组与不定长字节数组。

## bytes

### 定长字节数组

定长字节数组类型有 bytes1, bytes2, bytes3, bytes4, ..., bytes32。在 Solidity 0.8.0 版本之前， byte 是 bytes1 的别名。

#### 运算符

- 比较运算符： <=、< 、==、!=、>=、>
- 位运算符：&、|、^、~
- 移位运算符：<<、>>
- 索引访问:  `<bytesX>[i]`

#### 成员变量

- length

> The type `bytes1[]` is an array of bytes, but due to padding rules, it wastes 31 bytes of space for each element (except in storage). It is better to use the `bytes` type instead.
>
> 🤔 这个 padding rules 是啥？没懂。

### 不定长字节数组

不定长字节数组都不是值类型，而是引用类型。

#### bytes

bytes 是变长字节数组，详见数组。

#### string

UTF-8 编码的变长字节数组，详见数组。







