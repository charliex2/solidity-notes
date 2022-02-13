# 枚举类型

Solidity 中的枚举类型至少需要一个成员，**默认值是第一个成员**，最多不超过 256 个成员。

枚举类型可以在合约或者库定义之外的文件级别上声明。

#### 类型转换

枚举类型与整型可以显式装换，但不允许隐式转换。从整数转换为枚举类型时，会检查是否在枚举范围内。

#### 使用方法

```JS
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.16  <0.9.0;

contract Test {
  enum Colors { Red, Blue, Green, Pink }
  Colors favorite;
  
  function select() public {
    favorite = Colors.Red;
  }
  
  function getColor() public view returns (Colors){
    return favorite;
  }
}

```

这里需要说明的是，枚举类型并不能在 ABI 中使用，因此 Solidity 的



