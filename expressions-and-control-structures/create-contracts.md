# 创建合约

### 通过 `new` 创建合约

可以通过`new`关键词创建一个新合约。因为创建合约时所有的代码应该是明确已知的，所以递归创建是不可行的。

```js
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.7.0 <0.9.0;

contract D {
    uint public x;
    constructor(uint a) payable {
        x = a;
    }
}

contract C {
    D d = new D(4); // 这将在 C 的构造函数中创建

    function createD(uint arg) public {
        D newD = new D(arg);
        newD.x();
    }

    function createAndEndowD(uint arg, uint amount) public payable {

        // 创建合约实例的同时，发送 Ether。
        // 创建合约的同时，只能限定 value 值，不能限定 gas 值。
        D newD = new D{value: amount}(arg);
        newD.x();
    }
}
```

🤔：这里每 `new Contract`都会在区块链上创建一个合约？

### 加盐合约创建 / create2

默认情况下，创建的合约地址是根据 **发起合约地址** 和 **发起合约的 `nonce`**  来确定。

如果你指定了 `salt`值，那么 Solidity 便会根据**发起合约地址**，**salt值**，**合约的创建代码（the creation bytecode）**和**构造函数的参数**计算地址。这种方式中值得注意的是，计数器"`nonce`"并没有被使用。这使得创建合约的时候有更强大的灵活性，并使得新创建的合约地址具有一定的可预见性。Furthermore, you can rely on this address also in case the creating contracts creates other contracts in the meantime.

一个主要的使用场景是**充当链下交互仲裁合约，仅在有争议的时候才需要创建。**

>加盐合约创建有一些比较奇怪的地方。已经销毁的合约可以在同一地址重建。 创建字节码相同（**creation bytecode**），但是部署字节码（**deployed bytecode**）不同是有可能的，因为两次合约创建可能在构造函数中查询了外部状态，这个状态可能在两次创建之间发生了变化。