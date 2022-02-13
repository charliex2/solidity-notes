# 函数调用

#### 内部函数调用 Internal Function Calls

当前合约的函数可以直接被调用（internally），也可以递归调用。合约内的函数调用被 EVM 编译成了代码跳转，此时内存中的数据并不会被清除，因此在内部调用的函数之间传递内存引用非常高效。**仅同一个合约的函数可以被内部调用。**

递归调用并不能无限递归。每一次函数内部调用消耗一个堆栈槽，总共 1024 个堆栈槽。

#### 外部函数调用

表达式例如`this.g(8)` 和 `c.g(2)`，其中 c 是合约实例。这种外部调用使用过一种消息调用的机制进行，并不是代码跳转。从一个合约到另一个合约的消息调用并不会创建自己的 transaction，而是整个 transaction 的一部分。

**`this.g()`这种调用不能用于构造函数，因为此时该合约还未被创建。**

如果要调用其他合约的函数，就需要外部调用。对于一个外部调用，所有的函数参数都会被复制进内存。

当调用其他合约的函数的时候，可以使用**修改器**修改发送的 Wei 或者发送的 gas。**但是不建议明确指定 gas，因为各个操作码使用的 gas 消耗可能会在未来发生改变。** 发送给合约的 Wei 会加到合约的余额中。

```js
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.6.2 <0.9.0;

contract InfoFeed {
    function info() public payable returns (uint ret) { return 42; }
}

contract Consuymer {
    InfoFeed feed;
    function setFeed(InfoFeed addr) public { feed = addr; }
    function callFeed() public { feed.info{ value:10, gas: 800 }(); }
  	// 这里的 feed.info{ value:10, gas: 800 } 并没有执行该函数，而是最后的括号执行了该函数。
}
```

因为 EVM 调用一个不存在的合约的方法也会成功，因此 Solidity 提供了 `extcodesize`操作码来检查调用的合约是否整的存在（是否存在代码）。如果不存在则抛出异常。如果返回的数据会被解码，那么该检查就会被跳过好让 ABI 解析器捕获到合约不存在的错误。

低级别的 call 调用不会执行 `extcodesize`检查。

>Be careful when using high-level calls to [precompiled contracts](https://docs.soliditylang.org/en/latest/introduction-to-smart-contracts.html#precompiledcontracts), since the compiler considers them non-existing according to the above logic even though they execute code and can return data.
>
>🤔 啥是 precompiled contracts？

与其他合约做的任何交互都存在潜在的风险，特别是合约的源码你是不知道的。调用对方函数就是将控制权移交给被调用合约，而被调用合约可以做任何事。即使调用的函数是从一个已知的父合约继承而来，继承的合约也只需要实现一个正确的接口，而具体的实现则是完全任意的。

除此之外，小心调用的合约重新调用你所写系统的其他合约或当前的合约。这意味着调用另一个合约，而这个合约有可能回过头来修改发起调用合约的状态。

建议的写法是，先完成合约中各种变量的变更，然后再进行外部函数调用。这样你的合约便不会被轻易进行**重入攻击**（**reentrancy**）。

> 在 Solidity 0.6.2 之前，指定余额和 gas 的方法是使用 f.value(x).gas(g)()。在 0.6.2 中已经弃用，在 0.7.0 中已移除。

#### 具名调用和匿名函数参数

函数支持两种传参方式，一种是具名，即在 `{}` 中写明参数名和参数值，顺序可以任意；一种是匿名，只写值，但是需要保证顺序与声明时一致。

#### 省略函数参数名称

未使用的参数的名称（特别是返回的参数）可以省略。这些参数任然存在于堆栈中，但是无法访问。(🤔 有啥意义？)

```JS
pragma solidity >=0.4.22 <0.9.0;

conntract C {
  function  func(uint k, uint) public pure returns(uint){
    return k;
  }
}
```



