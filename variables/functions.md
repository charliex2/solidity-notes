# 函数类型

函数类型的表示形式：

```js
function(<parameter types) {internal|external|private|public} [pure|constant|view|payable] [returns (return types)]
```

若没有返回值，则需要删除整块 returns 部分。

函数类型可以用来

- 赋值给另一个变量；
- 作为参数传递；
- 作为函数返回值；

## 作用域

根据作用域可以分为：

- 内部函数 internal

  在当前代码块内可以调用，包括当前合约、子类合约；

- 外部函数 external

  外部函数用过一个地址和函数签名调用。

### 状态可变性修饰符

状态可变性是指该函数对区块链数据的修改程度。

🤔需要确认： constant、view、pure 修饰的函数告诉编译器该函数不修改状态变量，这样函数执行就不消耗 `gas`，因此不需要矿工来验证。

- pure

  不允许读取变量，也不允许修改变量。否则无法通过编译。

- constant

  在 solidity 4.17 之前，状态可变性修饰符仅有 `constant`。但  `constant` 与变量中的常量混淆，因此拆成了 `view` 和 `pure`。

  `constant` 的作用与 `view` 一模一样。

- view

  允许读取变量，但是不允许修改变量。

- payable

  允许接受 ether。

## 类型转换

两个函数类型可以互相转换，需要满足以下条件

- 参数类型相同；

- 返回类型相同；

- 函数作用域一致（即 internal 或 external 一致）

- 从**状态可变的限制程度****高向低**转化

  - `pure` --> `view`/`non-payable`

  - `view` --> `non-payable`

  - `payable` --> `non-payable`

    🤔 `payable` 函数也支持发送 0 ether，因此它也是 `no-payable`函数。这里有似乎比较牵强 ~

## 成员函数

public 或者 external 函数有下列两个成员函数：

- `.address ` 返回该函数所在的合约的地址
- `.selector ` 返回 ABI 函数选择器 （四字节选择器）

public 或者 external 函数有 `.gas(uint)` 与 `.gas(uint)`两个成员函数，但是已经在 0.6.2 中弃用，在 0.8.0 中移除了。目前使用 `{ gas: ... }` 和 `{value:...}` 两个修改器代替。

