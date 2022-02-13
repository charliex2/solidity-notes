# 引用数据类型

引用数据类型是指有多个名称（引用）可以访问和修改它的值。与值类型不同的是，值类型每次都是一个独立的副本。引用类型包括结构、数组和映射。

## 概述

### 数据位置

**引用类型必须明确指明数据存储所在的位置**，除了合约的成员变量（状态变量，state variable）是默认 `storage`位置的。

> 在 Solidity 0.5.0 之前，数据位置可以省略，因为 EVM 根据数值类型和函数类型等设定了默认数据位置。但是从该版本开始，所有复杂类型都需要显式指定数据位置。
>
> 🤔 这里的”复杂类型“应该指的就是引用类型。

- memory ： 数据存在内存中，因此数据仅在其生命周期内（函数调用期间）有效，不能用于外部调用；

- storage：区块链中，只要合约存在则一直存储；

- calldata：用来保存函数参数的特殊数据位置，该位置为只读不能修改、非持久化的数据位，其行为与 memoery 类似；

  Solidity 0.6.9 版本之前 calldata 仅能用于外部函数的参数，该版本后可以用于任何函数。

### 引用与拷贝

更改数据位置的赋值与类型装换总会自动生成一份拷贝。一般情况下，同一数据位的赋值是生成一个引用，但对应`storage`数据位置的数据在某些场景下也是生成一份拷贝。

下面罗列几条规则：

- storage, memory, calldata 之间的赋值总是生成一份拷贝；
- memory 之间的赋值是引用；
- 将 storage 里的变量赋值给局部 storage 变量，也是引用；
- 其他储存位置向 storage 位置的变量赋值，总是生成一份拷贝。例如向状态变量或者storage位置的 Struct 局部变量（即时这个局部变量只是个引用）赋值。

```js
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.5.0 <0.9.0;

contract C {
    // x 的数据位置是 storage。这是仅有的数据位置可以省略的地方。
    uint[] x;

    //  memoryArray 的数据位置是 memory.
    function f(uint[] memory memoryArray) public {
        x = memoryArray; // 可行，整个数组被拷贝进了 storage
        uint[] storage y = x; // 分配了一个指针（也就是引用）
        y[7]; // fine, returns the 8th element
        y.pop(); // fine, modifies x through y
        delete x; // 清楚了 x， 同时也影响到了 y
        // The following does not work; it would need to create a new temporary /
        // unnamed array in storage, but storage is "statically" allocated:
      	// 🤔 翻译没看懂，应该是 y 指向的 storage 数据已经被删除，y 没有指向任何存储空间了，因此再给 y 赋值会出错。
        // y = memoryArray;
        // delete 会重置指针，但此时 y 已经没有引用了，因此失败。
        // delete y;
        g(x); // calls g, handing over a reference to x
        h(x); // calls h and creates an independent, temporary copy in memory
    }

    function g(uint[] storage) internal pure {}
    function h(uint[] memory) public pure {}
}
```

## 类型

1. [数组 array](variables/array.md)
2. [结构体 struct](variables/struct.md)
3. [映射 mapping](varibales/mapping.md)
