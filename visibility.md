# 可见性

### 函数的可见性
函数的调用主要来源有以下几种
1. Contract 内部调用
2. 子 Contract 调用
3. 合约间调用。合约间调用通过 EVM internal call 来实现，因此其实合约自身也可以通过 `this.someMethod()` 来实现 internal call。
  

为了对上述 3 种调用方式进行不同策略的限制，solidity 定义了四个函数可见性的关键词。
1. public
   范围最大，支持任何形式的调用。
2. external
   只能在合约间调用。例如直接使用 `someMethod()` 来调用会失败，而 `this.someMethod()` 则可以正常使用。
3. internal
   可以在合约内调用，也可以供子合约调用。   
4. private
   范围最小，仅本合约可以调用，子合约也不可以调用。


### 状态变量
状态变量的作用域修饰符与函数的类似，但是没有 external。另外还有几需要注意：
- 状态变量的默认修饰符为 internal；
- public 修饰的变量，编译器会自动生成一个同名的 getter 函数；


# Getter 函数
- 普通数据类型的 getter 函数，调用后会返回该数据的值；
- 数组类型的 getter 函数，调用需要传入 index，返回该数组中对应下标的值；
- struct 类型的 getter 函数，调用返回该 struct 值；


下面有一个例子：

```js

// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.7.0 <0.9.0;

contract GetterTest {
    struct Data {
        uint age;
        address id;
    }

    Data public data;
    uint[] public myArray;
    uint public age = 25;
}
```

生成的 abi 为：
```json
[
	{
		"inputs": [],
		"name": "age",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [],
		"name": "data",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "age",
				"type": "uint256"
			},
			{
				"internalType": "address",
				"name": "id",
				"type": "address"
			}
		],
		"stateMutability": "view",
		"type": "function"
	},
	{
		"inputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"name": "myArray",
		"outputs": [
			{
				"internalType": "uint256",
				"name": "",
				"type": "uint256"
			}
		],
		"stateMutability": "view",
		"type": "function"
	}
]

```