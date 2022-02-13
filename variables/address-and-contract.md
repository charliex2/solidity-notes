# Address 与 Contract


## Address

从 solidity 5.0 版本开始，地址有两种类型，`address` 与 `address payable`。 两者的区别为：`address payable` 可支付地址，具有两个支付相关`transfer` 和 `send` 成员函数；而 `address` 没有这个要求，既可以有也可以没有支付功能。

两者的关系类似`address`是`address payable`的父类，因此两者互相转换的规则是：

- address -> address payable，显式转换。要求该 address 必须有 `receive` 函数或者有 payable 的回退函数；
- address payable -> address，隐式转换。

> 在 solidity 5.0 之前，address 与 address payable 的转化需要通过中间变量装换，例如
>
> ```
> payable(int160(<address>))
> ```

### 运算符

因为地址本质上就是`int160`，因此支持比较运算符，包括 ` <`、`<=`、`==`、`!=`、`\>=`、`\>`。地址进行加减乘数运算应该是没有意义的，因此地址不支持计算运算符。

### 成员函数

####  balance 

`balance`函数用于查询地址 Ether 余额。

```js
<Address>.balance()
```

#### transfer 

`transfer` 函数用于向该地址转账。

```js
<Address>.transfer(<uint>)
```

如果这里的`Adress`是合约地址，那么该合约的`receive`函数会被执行。如果不存在则`payable`修饰的回退函数会被执行。如果`payable`修饰的回退函数也不存在，则此次转账就会失败。如果转账失败会让 transfer 抛出异常，如果没有被处理，将会触发整个合约回滚。

这里需要注意的是，向某一个合约地址转账，有可能会以本合约为发起者（sender），调用和执行该合约的在receive函数或回退函数中定义的代码，但这些代码又是发起合约不可控的，这种机制可能会存在一些风险。

好在 EVM 为 transfer 函数设置了安全措施，即我们在使用 transfer 函数的时候，EVM 仅会给 transfer 函数触发的外部调用分配 2300 gas，这点 gas 并不足以支撑目标合约执行其他复杂的代码。

#### send

send 函数是 transfer 函数的低级版本，意味着其没有 transfer 诸多高级特性。两者的区别是

- send 调用如果失败，不会触发异常，而是返回一个代表成功与否的布尔值；
- send 调用没有特意的 gas 限制；

#### call

call 函数是低级函数，提供了直接控制编码与其他合约进行交互的能力。更底层意味着更加危险，需要更加小心使用。

call 函数接收一个 `bytes memory` 参数，返回一个代表成功与否的`bool`值和包含具体数据的`bytes memory`值。

`bytes memory` 的参数与返回值都可以使用[`abi`包内](../abi.md)的函数进行处理。 

```JS
address nameReg = 0x72ba7d8e73fe8eb666ea66babc8116a41bfb10e2;
bytes memory payload = abi.encodeWithSignature("register(string)", "MyName");
(bool success, bytes memory returnData) = address(nameReg).call(payload);
require(success);
```

#### delegatecall

delegatecall 的使用方式与call一致，区别在于 delegatecall 仅使用被调用合约的相应函数代码，但函数执行的环境还是当前的合约。

delegatecall 被设计用来实现类库代码的功能。即代码储存在一个合约地址上，其他代码可以使用该函数获取这些函数代码在本合约中执行。

#### staticcall

使用方式与 call 函数一致，不过在使用 staticcall 函数的时候，需要开发者明确知道调用的函数不会修改任何储存在区块链上的状态。

> ⚠️ 警告
>
> call、delegatecall、staticcall 都是非常低级的函数，因此使用需要非常小心。任何你使用这些函数调用的其他合约函数都可能是恶意的，有可能会重新回调你当前合约中的函数。这里的风险可以考虑 "**重入**"。

### 修改器

修改器可以修饰上述的 call、delegatecall、staticcall函数，设置调用时的 gas 数量和转账的 ether 数量。

```js
bytes memory payload = abi.encodeWithSignature("register(string)", "MyName");
address(nameReg).call{gas:1000000, value: 1 ether}(payload);
```

其中 delegatecall 不支持有 value 修饰。为什么不能有 value 修饰的原因应该是出于保护调用者资产的考虑。

### 类型转化

Address 可以与以下类互相转化：
1. `address` <-> `address payable`

   - address payable 转 address 为隐式转化；

   - address 转 address payable 为显式转化，用法为 `payable(<address>)`；
   
     🤔 **待确认：address 转为 address payable 似乎没有要求这个地址对应的合约一定要可转账，但是 Contract 类型转为 address payable 有要求。**
     
   - 特例：payable(0) 是允许的；
   
     
   
2. `address` <-> `uint160`

3. `address` <-> `bytes20`

4. `address` <-> `Contract`

## Contract

在 Solidity 0.5.0 版本之前，Contract 是 Address 的子类，但自该版本后 Contract 不再是 Address 的子类。

### 类型转换

1. 子合约转父合约为隐式转化；

2. Address Contract 互转

   两者互转均为显式转化；

3. Contract 与 address payable 互转

   两者互转均为显示转化。

   - 当该合约实现了转账功能，  address payable 转 Contract 类型使用的是 address(x) 函数。

   - 当该合约未实现转账功能，可以借道 address 类型曲线救国，即 `payable(address(x))`

     🤔 但是没明白这么做的意义？

### 类的创建

#### 显式转换赋值

该种方式适用于另一个合约已经部署，可以在本合约中根据合约地址创建出来。

```js
ContractA contractA = Contract(<address>);
```

🤔 这种方式文档中描述不清晰

#### new 创建

该种方式适用于要创建的合约代码是自己写的或者获取到了合约代码的情况，可以在本合约中引入后通过 new 关键词创建出来。

```JS
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.0 <0.9.0;
contract D {
    uint public x;
    constructor(uint a) payable {
        x = a;
    }
}

contract C {
    D d = new D(4); 
}
```

更多通过 new 的方式创建另一个合约对象的知识可以阅读文档 [Create Contract](https://docs.soliditylang.org/en/latest/control-structures.html#creating-contracts)。

### 运算符

合约不支持任何运算符。

### 成员变量与成员函数

合约类型的成员是合约的外部函数及 public 的 状态变量。

