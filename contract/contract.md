# Contract

Contract 类似于面向对象编程语言中的类， Contract 具有持久化的状态变量和函数可以来修改这些变量。如果在合约中调用另一个合约的函数，那么会执行一个 EVM 函数调用，这个操作会切换执行时的上下文，在这个函数内不能访问这个函数发起合约中的状态变量。

**合约和函数必须是被主动触发的，没有“cron”定时任务概念和也没有被事件自动触发的机制。**

### Contract 创建

1. contract 创建的途径有两种

- 通过 Ethereum transaction 链外部创建；
  - IDES，例如 Remix
  - Web3.js 或者 ether.js
- Solidity contract 创建

2. Constructor 

   构造函数数可选的，并且必须只能一个。因此覆写（overloading）是不允许的。

3. 构造函数被执行后，最终的代码才会被存储在区块链上，代码包括 public 和 external 函数这类允许通过 function call 访问的函数。部署后的代码（deployed code）并不包括构造函数代码和那些仅能被构造函数访问的代码。

4. 在内部，constructor 的参数在合约自身代码后经过 ABI encoded 后传递；但是你使用 web3.js，那么你就不需要关系这个。

   Internally, constructor arguments are passed [ABI encoded](https://docs.soliditylang.org/en/latest/abi-spec.html#abi) after the code of the contract itself, but you do not have to care about this if you use `web3.js`.

   🤔 原文是这个，不大清楚具体指什么？

5. 如果通过一个合约创建另一个合约，那么这个合约的源码（二级制码）必须是已知的。因此循环创建依赖的Contract。



示例：

```js
// SPDX-License_Identifier: GPL-3.0
pragma solidity >=0.4.12 <0.9.0;

contract OwnedToken {

    TokenCreator creator;
    address owner;
    bytes32 name;

    constructor(bytes32 _name){

        owner = msg.sender;

        // 这里进行了 address 到 TokenCreator 的显式转换
        // 这里需要确保调用的合约就是 TokenCreator 类型的，但是并不存在方法去验证这一点。
        // 这里并不是要创建一个合约
        creator = TokenCreator(msg.sender);
        name = _name;
    }

    // 只有 creator 能修改 name
    function channgeName(bytes32 newName) public {
        if(msg.sender == address(creator))
            name = newName;
    }

    function transfer(address newOwner) public {
        if(msg.sender != owner) return;

        if(creator.isTokenTransferOk())
            owner = newOwner;
    }
}

contract TokenCreator {
    function createToken(bytes32 name) public returns (OwnedToken tokenAddress){
        // 创建了一个新的 Token 合约实例。
        // 因为此处是外部调用，因此获取到的是 address，因为在 ABI 中 address 是最接近于 Contract 的类
        return new OwnedToken(name);
    }

    // 该函数需要传入的类型是 OwnedToken，但是在 ABI 中只需要传入 address 类型的值即可
    function changeName(OwnedToken OwnedToken, bytes32 name) public {
        OwnedToken.changeName(name);
    }

    function isTokenTransferOk(address currentOwner, address newOwner)
        public
        pure
        returns (bool ok)
    {
        // 这里应该是任意写了一个判断条件
        return keccak256(abi.encodePacked(currentOwner, newOwner))[0]==0x7f);
    }
}
```

