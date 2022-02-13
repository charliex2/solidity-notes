# 单位与全局变量

## 以太币单位

以太币单位之间的换算就是在数字后加上 wei、gwei 或者 ether 来实现。如果没有单位，缺省是 wei。

gwei 在 Solidity 0.6.11 中新增。

##  时间单位

`seconds`、`minutes`、`hours`、`days` 与 `weeks`。其中 `years` 已经在 0.5.0 中因为闰年的问题已经被移除。

时间单位只能应用于字面量，不能应用于变量。如果要在函数的参数中使用，可以按下述示例展示：

```js
function f(uint start, uint daysAfter) public {
  if(block.timestamp >= start + daysAfter * 1 days) {
    // ....
  }
}
```

## 特殊变量和函数

Solidity 语言层面的原生 API。

### Block 和 Transaction 的属性

- blockhash(uint blockNumber) returns (bytes32)

  指定区块的区块哈希。仅可获取最新的 256 个区块但不包括当前区块；

- lock.basefee(uint): 当前区块的 base fee (EIP-3198 and EIP-1559)；

- block.chainid(uint): 当前区块的ID (EIP-3198 and EIP-1559)；

- block.coinbase (address payable): 当前区块的矿工地址；

- block.difficulty(uint): 当前区块挖矿困难程度；

- block.gaslimit(uint): 当前区块的 gas 限额；

- gasleft() returns (uint256): 剩余 gas；

- msg.data (bytes calldata):  全部 calldata；

- msg.sender(address): 消息的发送者；

- msg.sig (bytes4): calldata的前 4 bytes；

- msg.value(uint): 随 message 发送的 wei；

- tx.gasprice(uint): 该 transaction 的 gas 价格；

- tx.origin(address)： transaction 的发布者；

注意点：

- `msg`的成员`msg.sender`和`msg.value`会随着`external function` 的调用而改变。其中也包括`library functions`。

- 当合约是在链下进行评估的，那么你不能假设 block.* 和 tx.* 对应的值会来自于某一个特定的区块（Block）或者交易（Transaction）。因为这些值是由 EVM 在执行这个合约的时候提供的，这一过程可以说是随机的。

- 除非你知道在做实名，否则不要依赖于 `block.timestamp` 和 `blockhash` 作为随机源。因为无论是时间戳还是 block hash都会受到矿工某种程度上的影响。例如，挖矿社区的恶意矿工可以用某个给定的哈希来运行赌场合约的 `payout` 函数，当没有受到钱的时候，还可以尝试用不同的哈希进行尝试。

- `block hash` 只能访问最近的 256 个区块，其他区块的值都是 0。

- 函数 `blockhash` 的前身是 `block.blockhash,`  其已经在 Solidity 0.4.22 废弃并在 0.5.0 版本移除。

- ###### 函数 `gaslet` 的前身是 `msg.gas`，其已经在 Solidity 0.4.21 废弃并在 0.5.0 版本中移除。

- 从 Solidity 0.7.0 开始，`now` 函数（block.timestamp）已经被移除。

### ABI Encoding 和 Decoding 函数

- `abi.decode(bytes memory encodedData, (...)) returns (...)`

  ABI 解码指定的 bytes 数据，该数据的类型由第二个参数指定。例如

  ```js
  (uint a, uint[2] memory b, bytes memory c) = abi.decode(data, (uint, uint[2], bytes))
  ```

- `abi.encode(...) returns (bytes memory)`

  ABI-编码指定的参数。

- `abi.encodePacked(...) returns (bytes memory)`

  对指定参数执行[**紧打包编码**](https://learnblockchain.cn/docs/solidity/abi-spec.html#abi-packed-mode)。注意，可以不明确打包编码。

- `abi.encodeWithSelector(bytes4 selector, ...) returns (bytes)`

  对给定的第二个开始参数进行编码，并以给定的函数选择器作为起始的4字节数据

- `abi.encodeWithSignature(string signature, ...) returns (bytes)`

  等价于 `abi.encodeWithSelector(bytes4(keccak256(signature)), ...)`

> 上述的 encoding 函数可以用来手动组装调用外部函数（**external function**）的数据，而不用真正调用这些外部数据。
>
> 除此之外，`keccak256(abi.encodePacked(a, b))`是计算结构数据哈希值的一种方式，但是请注意在使用不同的函数参数类型的时候，有可能会引起哈希碰撞。
>
> 了解更多计算结构化数据的哈希，可以参考 [EIP712](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md)，或者阅读中文文章 [《理解 EIP712 - 类型结构化数据 Hash与签名》](https://learnblockchain.cn/2019/04/24/token-EIP712/)

### bytes 的成员函数

- `bytes.contact(...) returns (bytes memory)`

  将动态大小的 bytes 和 固定长度的 bytes1、bytes2、... bytes32 合并为单个 byte 数组。

### string 的成员函数

- string.contact(....) returns (string memory)

  将多个字符串合并为一个字符串。

### 错误处理

- `assert(bool condition)`

  用于处理内部错误。如果条件不符，即触发 Panic 错误并让状态回滚。

- `require(bool condition)`

  用于处理输入或者外部组件的错误。如果条件不符，则回滚。

- `require(bool condition, string memory message)`

  作用同上，但可以自定义错误信息。

- `revert()`

  终止执行，并回滚状态。

- `revert(string memory reason)`

  作用同上，但是可以自定义错误信息。

### 数学与加密学方法

- `addmod(uint x, uint y, uint k) returns (uint)`

  计算 `(x + y) % k`时加法可以无限精度进行，即使超过了`2**256`也不会被截断。Solidity 0.5.0 版本编译器新增了 `k != 0` 校验；
  
- `mulmod(unit x, uint y, uint k) returns (uint)` 

  计算`(x*y) % y`时乘法可以无限精度进行，即使超过了`2**256`也不会被截断。Solidity 0.5.0 版本编译器新增了 `k != 0` 校验；

- `keccak256(bytes memory) returns (bytes32)`

  计算输入值的 Keccak-256 哈希值；在 Solidity 0.5.0 之前，`sha3`是`keccak256`的别名，现已删除；
  
- `sha256(bytes memory) returns (bytes32)`
  
  计算输入值的 SHA-256 哈希值；
  
- `ripemd160(bytes memory) returns (bytes20)`
  
  计算输入值的 RIPEMD-160 哈希值；
  
- `ecrecover(bytes32 hash, uint8 v, bytes32 r, bytes32 s) returns (address)`
	
  利用椭圆区县签名恢复公钥对应的 address，若出错则返回零值。
  
  函数参数对应于 ECDSA 签名的值：
  
  - r =  签名的前 32 字节；
  - s = 签名的第2个32字节；
  - v = 签名的最后一个字节；
  
  ecrecover 返回一个 address，而不是 address payable。
  
  > 🤔 以下内容没看懂
  >
  > If you use `ecrecover`, be aware that a valid signature can be turned into a different valid signature without requiring knowledge of the corresponding private key. In the Homestead hard fork, this issue was fixed for _transaction_ signatures (see [EIP-2](https://eips.ethereum.org/EIPS/eip-2#specification)), but the ecrecover function remained unchanged.
  >
  > This is usually not a problem unless you require signatures to be unique or use them to identify items. OpenZeppelin have a [ECDSA helper library](https://docs.openzeppelin.com/contracts/2.x/api/cryptography#ECDSA) that you can use as a wrapper for `ecrecover` without this issue.
  
  >在私有链上执行`sha256`,`ripemd160`或`ecreover`可能会遇到 Out-of-Gas。因为这些函数被实现为预编译合约（Precompiled Contracts），等他们收到第一条消息之后他们才会真实存在（尽管这些合约的代码是硬编码的）。
  >
  >给不存在的合约发送消息更加昂贵，因此会发生 Out-of-Gas 错误。
  >
  >为了解决该问题，在你真正的合约中使用这些函数前，你可以可以先给每一个合约发送点点 Wei。
  >
  >在主链或者测试链上不存在该问题。

### Address 类型的成员

- `<address>.balance    (uint256)`

  该地址的余额（单位 Wei）。

- `<address>.code  (bytes memory)`

  该地址上的代码。（非合约地址可能为空）

- `<address>.codehash   (bytes32)`

  该地址上的 codehash。

- `<address payable>.transfer(uint256 amount)`

  向某一个地址发送一定量的 Wei。失败时抛出异常，使用固定不可调节的 2300 gas的矿工费。

- `<address payable>.send(uint256 amount) returns (bool)`

  向某一地址发送一定了的Wei，若失败返回`false`。使用固定不可调节的 2300 gas。

  🤔 这里好像以前不是固定数量 gas？

- `<address>.call(bytes memory) returns (bool, bytes memory)`

  用给定的有效载荷（payload）发出低级 CALL 调用，返回成功状态和数据。call 会传递所有可用 gas ，可以手动调节。

- `<address>.delegatecall(bytes memory) returns (bool, bytes memory)`

  用给定的有效载荷（payload）发出低级 `DELEGATECALL`调用，返回成功状态和数据，发送所有可用的 gas，也可以调节gas。失败会返回 `false`。

- `<address>.staticcall(bytes memory) returns (bool, bytes memory)`

  用给定的有效载荷发出`STATICCALL`调用，返回成功状态和数据，发送所有可用 gas，也可以调节 gas。

  > 在执行另一个合约函数时，应该尽量避免使用 `.call()`,因为它绕过了类型检查，函数存在检查和参数打包。

  > 使用 `send`函数有一些危险的点：
  >
  > - 当调用栈深度大道 1024 时（这可以被调用者强制指定），调用会失败；
  > - 当接收者用光了 gas ，转账会失败；
  > - `send`函数需要检查返回值。
  >
  > 建议使用 `transfer`或者用接收者取回钱的模式。

	>调用一个不存在的合约总是会成功。
	>
	>Solidity 提供了一个额外的`extcodesize`操作码用于检查调用的合约是否存在代码（code），不存在则会抛出错误。
	>
	>这些作用在地址上而不是合约实例上的低级 calls，例如`.call()`、`.delegatecall()`、`.staticcall()`、`.send()`和`.transfer()`都不会进行该项检查，因为不检查更节约 gas 但是相应的更加不安全。
	
	> 在 Solidit 0.5.0 之前，允许直接在合约实例上访问上述address 的成员，但是现在不允许了，必须先要进行显示转化：
	>
	> ````js
	> address(this).balancece
	> ````
	> 

	>在 Solidity 0.5.0 之前，`.call`,`.delegatecall`,`.staticcall`都仅仅返回成功状态，但是并不返回数据，。
	
	> 🤔 这个没看懂
	>
	> If state variables are accessed via a low-level delegatecall, the storage layout of the two contracts must align in order for the called contract to correctly access the storage variables of the calling contract by name. This is of course not the case if storage pointers are passed as function arguments as in the case for the high-level libraries.
	
	> 在 Solidity 0.5.0 之前，有一个与 `delegatecall`类似但在语义上又有所不同的函数 `callcode`，现在已经废弃。

### 合约相关

- `this` 当前的合约类型

  指当前的合约类型，可以显式地转为 Address 类型。

- selfdestruct(address payable recipient)

  销毁当前合约，并将其资金转移到指定的地址，然后结束执行。在 Solidity 0.5.0 之前叫 `suicide`，现在已更名。

  请注意 `selfdestruct` 从 EVM 中继承了一些特性：

  - **接收余额的合约，其 `receive` 函数并不会执行**。
  - 在这次销毁 transaction 执行完了才会真正销毁该合约。`revert`会取消该次销毁。

当然，合约中中定义的函数都可以被调用，包括本函数。

### Type 信息

`type`表达式可以被用来获取某一对象的类型信息，目前仅仅支持合约类型和整数类型。在未来，`type`表达式还可以被扩展到其他类型。

下列属性是在**合约类型**信息支持的属性：

- `type(C).name`

  获取合约名。

- `type(C).creationCode`

  获取包含创建该合约字节码的内存字节数组。This can be used in inline assembly to build custom creation routines, especially by using the `create2` opcode. 🤔？

  该属性不能在本合约或者其他继承自该合约的合约中使用，因为可能会造成循环引用。

- `type(C).runtimecode`

  获取合约运行是字节码的内存字节数组。这些代码通常是合约构造函数创建的代码。如果该合约有一个使用内联汇编的够着函数，那么这些代码可能与实际不是的字节码不同。

下列是接口类型（type `I`）的属性：

- `type(I).interfaceId`

  A `bytes4` value containing the [EIP-165](https://eips.ethereum.org/EIPS/eip-165) interface identifier of the given interface `I`. This identifier is defined as the `XOR` of all function selectors defined within the interface itself - excluding all inherited functions.

  这个我没看懂 🤔？

下列是整型类型 T 的属性：

- `type(T).min`

  类型 `T`的最小值；

- `type(t).max`

  类型`T`的最大值；



