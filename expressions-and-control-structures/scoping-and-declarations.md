# 作用域与声明

### 声明

声明变量后便有有一个初始值，在比特表现上就都是零。例如：

- bool 默认为 false
- uint/int 默认为 0
- 定长数组、bytes1...bytes32 = 每个元素都是零值的数组
- 动态长度数组，string 的默认值为长度为0的数组或者 string
- enum 类型，其默认值为一个各个成员都是其默认值的 enum



### 作用域

Solidity 的作用域遵循 C99 的规则。变量在从其声明开始一直到其所在最小的 {} 块中有效。有一个例外，在 for 循环中初始化的变量的在 for 循环前一直有效，循环结束则其也失效。

参数形式的变量，例如函数参数、修饰器参数、catch 参数在其后紧接着的代码块内有效。

那些定义在代码块之外的变量，例如函数、合约、自定义类型等，其可见性跟定义的顺序无关，甚至可定义前都是可用的。这意味着你可以在实际声明前使用他们，也可以递归调用函数。

```js
pragma solidity >=0.5.0 <0.9.0;

contract C {
    function minimalScoping() pure public {
        {
            uint same;
            same = 1;
        }
				// 虽然前面已经定义了 same，但是因为在不同的 {} 中，因此编译并不会出错。
        {
            uint same；
            same = 3;
        }
    }
}
```

作为 C99 作用域规则，下面有一个例子

```js
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.5.0 <0.9.0;
// This will report a warning
contract C {
    function f() pure public returns (uint) {
        uint x = 1;
        {
            x = 2; // this will assign to the outer variable
          
          	// 下面这一行代码将要收获一个 warning
          	// Warning: This declaration shadows an existing declaration.
            uint x;
        }
        return x; // x has value 2
    }
}
```

> 在 Solidity 0.5.0 版本之前，作用域规则沿用了 JavaScript 的规则，即一个变量可以声明在函数的任意位置，都可以使他在整个函数范围内可见。