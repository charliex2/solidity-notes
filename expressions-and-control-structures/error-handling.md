# 错误处理

如果异常在子调用中发生，name异常会自动冒泡到顶层。但是低级别的调用例如：`send`、`call`、`delegatecall`、`staticcall`的调用里发生异常时，他们会返回 `false`。所以如果需要，需要在返回前检查返回值。

错误发生时，会返回 error 实例。内置的错误类型有 Error(string) 和 Panic(uint256) 

- Error 用于常规错误场景
- **Panic is used for errors that should not be present in bug-free code.**

### Panic 错误

`assert` 会产生 `Panic` 错误，函数只能用于测试内部错误，检查表不变量。正确的函数代码永远不会产生 Panic，甚至是基于一个无效的外部输入。如果发生了，那么就意味着产生了一个需要你修复的 bug。语言分析工具可以对你的合约进行评估分析出会引起 Panic 的 条件和函数调用。

下列场景会引发 Panic 错误，其错误码可以指明错误的类型：

- `0x00`: 编译器插入的通用 panic；
- `0x01`: assert() 判断某个参数为 false；
- `0x11`: 在 unchecked{} 代码块中的上、下溢出；
- `0x12`: 除以0 或者 取 0 的模产生的错误；
- `0x21`: 将一个太大或者负数转为枚举类型；
- `0x22`: 范围一个未正确编码的 storage 字节数组；
- `0x31`: 对一个空数组进行 pop 操作；
- `0x32`: 数组、bytesN、或者数组切片下标超限或者下标为负数；
- `0x41`: 分配了太多内存或者创建了太大的数组；
- `0x51`: If you call a zero-initialized variable of internal function type.🤔？

### Error 错误

`require` 函数产生一个带或者不带错误信息的 `Error`。`require` 主要怕用来处理运行时的错误，例如用来判断输入值和调用外部合约函数的返回值的合法性。

目前尚不支持用 require 触发自定义错误类型，不过可以使用 `if(!condition) revert CustomError();`

编译器会在以下场景触发 Error

- `require(x)` 为 false 时；
- 使用 `revert` 函数时；
- 调用一个外部合约的函数，但是这个合约内并没有代码；
- 当你的合约通过 public 函数调用收到了一笔 Ether 转账，但是没有 payable 修饰（包括 constructor 和 fallback 函数）
- 当你的合约通过 getter 函数接收 Ether。

### 错误类型不定的场景

如果错误来自外部调用被转发，这意味着 Error 和 Panic 都有可能被被触发。

- transfer() 失败；

- 通过消息调用（message call）调用一个函数，但是该函数未正确完成。可能有的原因很多，比如 gas 耗尽，没有匹配的函数，自身抛出异常；

  这里说的函数不包括 call、send、 delegatecall、callcode、staticcall 等低级调用，这些函数如果失败仅会返回 false。

- 通过 new 关键词创建一个合约，但是创建过程未完成。



在内部，Solidity 遇到错误会进行一个 revert 操作（操作码为 `0xfd`），该操作码会引起 EVM 回滚本次事务的所有状态变更。

**在所有的场景中调用者（`caller`）可以使用 try/catch 处理错误，但是被调用者 (`callee`) 中的状态变更还是会被回滚。**

> 在 Solidity 0.8.0 版本之前， Panic 错误使用 invalid 操作码，其会消耗掉本次调用所有的 gas。在 Metropolis 本标签。require 触发错误也会消耗掉所有的 gas。



### revert

revert 可以通过 revert 声明或者 revert 函数触发。

```javascript
# revert 指令可以直接抛出一个自定义错误
revert CustomError(arg1, arg2);

# revert function
revert(); //  引发 revert ，不带任何错误信息
revert("description"); // 引发一个 Error(string) 错误
```

使用自定义错误实例会比使用字符串描述的错误更加节约 gas，因为我们可以用自定义错误名描述错误，错误名最终编码只有 4 字节。而长的字符串描述需要通过 NatSpec 来实现，那个这个 gas 花销就无法估计了。

```js
// SPDX-LICENSE-Identifier: GPL-3.0

pragma solidity ^0.8.4;

contract VendingMachine {
    address owner;
    error Unauthorized();

    function buy(uint amount) public payable {
        if(amount > msg.value / 2 ether){
            revert("Not enough Ether provided.");
        }

        // 另一种实现方式
        require(
            amount <= msg.value / 2 ether,
            "Not enough Ether provided"
        )
        // ...
    }

    function withdraw() public {
        if(msg.sender != owner)
            revert Unauthorized();
        payable(msg.sender).transfer(address(this).balance);
    }
}
```

上面例子中提到 `if(!condition revert(...));`和`require(condition, ...)` 只要其传入的参数没有副作用，那么他们就是等效的。

`require()`函数跟其他函数一致，如果传入的参数如果是函数或者表达式，那么他们会在 require 函数执行前执行，即使 require 的条件是 true。例如

```js
require(condition, f());
```

#### Error(string) 错误信息的编码

Error(string) 在 EVM 是 abi 编码的。例如 `revert("not enough Ether provided."); `最终返回的错误数据为`：

```js
0x08c379a0                     // Function selector for Error(string)
0x0000000000000000000000000000000000000000000000000000000000000020 // Data offset
0x000000000000000000000000000000000000000000000000000000000000001a // String length
0x4e6f7420656e6f7567682045746865722070726f76696465642e000000000000 // String data
```

这些错误信息可以被 `try/catch` 取回。

> 在 0.4.13 版本中有一个 关键词 throw 与 revert() 在语义上一致，此后版本已经废除，并在 0.5.0 版本中移除。

### try/catch

外部调用的失败，可以通过 try/catch 语句来捕获；

```js
// SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.1;

interface DataFeed {
    function getData(address token) external returns (uint value);
}

contract FeedConsumer {
    DataFeed feed;
    uint errorCount;

    function rate(address token) public returns (uint value, bool success){
        require(errorCount < 10);

        try feed.getData(token) returns (uint v){
            return (v, true);
        } catch Error(string memory /**reson**/){
            // 如果在 getData 中 revert 被执行，那么会被此处捕获，并提供一个错误字符串
            errorCount++;
            return (0, false);
        } catch Panic(uint /**code**/){
            // 如果 getData 抛出 panic，例如除以 0 错误或者溢出错误。其错误码可以用来判断错误的类型
            errorCount++;
            return (0, false);
        } catch (bytes memory /**lowLevelData**/){
            // 在 revert() 使用时触发
            errorCount ++;
            return (0, false);
        }
    }
}
```



try 关键词后必须跟随着 外部函数调用或者合约创建(`new ContractName()`)语句。该表达式内部的错误将不会被捕获（例如该表达式是一个比较复杂的函数，内部还有调用另一个内部函数），仅仅只能捕获该 external call 自身发生的错误。

returns 不是是可选的，该处声明的需要与这个外部调用返回的值一致。如果没有错误发生，name这些在 returns 声明的值都会被赋值，success 代码块中的代码也会被执行。

目前 Solidity 针对不同的错误类型支持不同的 catch 块。如果错误是由 revert("reasonStringf") 或者 require(false, "reasonString")，或者是由内部错误引起的这些错误，那么就能捕获到 Error(string memory reason)。

It is planned to support other types of error data in the future. The string `Error` is currently parsed as is and is not treated as an identifier.  🤔

当 error 的signature 没有匹配上、当解析错误信息出错、当没有错误信息提供的时候，`catch (bytes memory lowLevelData)`会被执行。该 catch 的声明的变量，可以访问到低级的错误数据；

> 如果是解析 external call 的返回值出错，那么只会在本合约触发一个错误，并不会被 catch 块捕获；如果是解析 `catch Error(string memory reason)` 中解析错误信息出错，那么该错误会被 catch 块捕获；

> 如果执行到了 catch-block，外部调用引起的 state-changing 会被回滚。如果成功执行到了 success 块，那么就不会被回滚。如果被回滚后，将会继续执行 catch 块，或者 try/catch 自身也 revert 了（例如解析错误信息出错）。

> 引起错误的原因多种多样。不要假定所有的错误信息直接来自于合约。这个错误可能是调用链深处引起的，你调用的合约仅仅是转发了这个错误。
>
> 错误可能是 gas 用完和not a deliberate error condition：调用者（caller）总是在调用外部合约的时候保留 1/64 的gas，以保证外包合约用光了 gas 还能有部分 gas 保留。
