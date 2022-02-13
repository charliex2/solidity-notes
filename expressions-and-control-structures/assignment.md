# 赋值

## 解构赋值与多返回值

Solidity 内部允许元祖 (tuple) 类型，也就是一个在编译时元素数量固定的对象列表，列表中的元素可以使不同类型的对象。这些元祖可以用来给函数同时返回多个值，也可以用来给新声明或者已经存在的变量（或 LValues）赋值。

```js
pragma solidity >=0.5.0 <0.9.0;

contract C {
    uint index;

    function f() public pure returns (uint,  bool, uint) {
        // 多个返回值
        return (7, true, 2);
    }

    function g() public {
        // 基于返回的元祖来声明变量并赋值
        (uint x, bool b, uint y) = f();

        // 交换两个值的窍门 - 但不适用于非值类型的存储 (storage) 变量
        (x, y) = (y, x);

        // 解构可以省略参数。这也适用于变量声明。
        (index,,) = f();
    }
}
```

混合变量声明和非变量声明赋值是不合法的。例如`(x, uint y) = (1, 2)`

### 数组和结构体的复杂性

数组和结构体，包括 bytes 和 string 这类非值类型的赋值语义会比较复杂。详情可以见 [数据位置与赋值行为](https://learnblockchain.cn/docs/solidity/types.html#data-location-assignment)。

```js
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.4.22 <0.9.0;

contract C {
    uint[20] x;

    function f() public {
        // 调用该 g 函数的时候，因为 g 函数会将参数拷贝内存中，
        // 因此仅仅改了内存中的数值，并不会修改 state 值 x
        g(x);

        // h 函数传递了引用，因此 hh 函数会修改 state 值 x
        h(x);
    }

    function g(uint[20] memory y) internal pure {
        y[2] = 3;
    }

    function h(uint[20] storage y) internal {
        y[3] = 4;
    }
}
```

