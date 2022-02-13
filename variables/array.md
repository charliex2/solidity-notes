
# 数组 Array

### 声明

数组可以的声明的时候指定长度，也可以动态调整长度。

固定长度数组声明例如 `T[k]`，动态数组声明例如 `T[]`。二维数组声明例如 `T[i][j]`，代表 j 行 i 列，**声明顺序与其他开发语言不同**。

### 元素类型

数组的元素可以是任意类型，包括 mapping 与 struct。数组对类型的限制为：

- 如果数组内元素类型是 mapping  只能存储在 storage 中；
- public 的函数参数必须是 ABI 类型；

### 访问元素

一维数组直接通过下标访问，下标从 0 开始计数。二维数组访问的小标顺序与声明顺序相反，例如 `T[x][y]`代表第 x 行第 y 列的数据。

public 修饰的数组类型状态变量（state variable）Solidity 会自动生成一个 `getter`函数，这个函数有一个代表下标的参数。

### 特殊数组

bytes 和 string 类型的变量都是特殊的数组。

应该尽量使用 bytes 而不是 bytes1[]，因为 bytes 更加便宜，因为 bytes1[] 在内存中会在每个元素之间添加 31 个字节的间隔，不过在 storage 中因为紧打包的原因是没有间隔的。

**基本原则是，对于任意长度的原始字节数据使用`bytes`,对任意长度字符串（UTF-8）数据使用 `string`。对于固定长度的字节数组，则使用对应的 bytes1、bytes2 .... bytes32的具体类型。**

#### string

`string` 与 bytes 一致，不过无法访问其 length ，无法使用下标访问。另外 string 类型也没有操作符，需要借助其他函数实现一些字符串操作，例如

- 比较两个字符串需要借助 `keccak256-hash`函数
- 拼接两个字符串借助 `bytes.contact`

```js
keccak256(abi.encodePacked(s1)) ==  keccak256(abi.encodePacked(s2)))
bytes.concat(bytes(s1), bytes(s2))
```

### 内存数组

可以使用`new`关键字创建动态长度的内存数组。与 storage 函数不同的是，<u>**内存数组的长度是固定的，仅能在创建的时候指定，你无法使用例如 `push()` 函数来改变内存数组的长度。**</u>所以你必须提前计算好所需函数的长度或者直接新建一个数组然后将各个元素拷贝进新数组。

跟其他 Solidity 的数据类型一样，新创建的数组中的每一个元素都会被初始化为其默认值。

```js
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.15 <0.9.0;

contract C {
  function f(uint len) public pure {
    uint[] memory a = new uint[](7);
    bytes memory b = new bytes(len);
    assert(a.length == 7);
    assert(b.length == len);
    a[6] = 8;
  }
}
```

### 数组字面常量

数组字面常量是在方括号内包含一个或者多个逗号分隔的表达式。数组字面常量总是静态固定大小的内存数组。

1. **数组的基本类型即数组第一个元素的类型，其他元素必须都可以隐式地转化为普通类型。** 如果仅仅是有一个数据类型，这个数组的所有元素都可以转换这还不够，必须是第一个元素是这个类型。🤔 原文是   One of the elements has to be of that type，感觉不对。

   例如：`[1,2,3] `的类型是 `uint8[3] memory`，因为每一个常量都是 `uint8`。如果你希望是 `uint[3] memory`，那么需要这样写 `[uint(1), 2, 3]`。

   例如：`[1, -1 ]` 是不对的，因为第一个元素是uint8，第二个元素是 int8，两者无法隐式转换。正确的写法是 `[int8(1), -1]`。

   

   **对于二维数组，需要保证所有内层数组的基本类型一致。因为不同类型的定长内存数组不能互相转化，即使其基本类型可以互相转化。**

   例如：

   ```js
   // 可行，内层数组元素是 uint24, 其中 0xffffff 也是 uint24。
   uint24[2][4] memory x = [[uint24(0x1), 1], [0xffffff, 2], [uint24(0xff), 3], [uint24(0xffff), 4]];
   
   // 不可行，内层数组类型不一致。
   uint[2][4] memory x = [[0x1, 1], [0xffffff, 2], [0xff, 3], [0xffff, 4]];
   
   ```

2. **定长的内存数组不能赋值给变长的内存数组**（该限制可能在未来移除），例如：
	
	```JS
	uint[] x = [uint(1), 2, 3];
	```
	因此如果你想初始化一个动态长度的数组，你必须一个个元素赋值。例如:
	
	```js
	uint[] memory x = new uint[](3);
	x[0] = 1;
	x[1] = 3;
	x[2] = 4;
	```

### 数组的成员

- length

  内存数组一旦创建长度就是固定的。不过在创建是可以任意指定长度。通过 `length`可以访问数组的长度。例如：

  ```JS
  int8[3] = new int
  ```

- push()

  **Storage 中的动态数组或 bytes**(不包括 `string`) 有该成员。使用 `push()`可以在该数组后添加一个默认值。该成员函数返回该新增元素的引用，例如可以如下方法使用：

  ```JS
  x.push().t = 2;
  x.push() = 3;
  ```

  

- push(<element>)

  **Storage 中的动态数组或 bytes**(不包括 `string`) 有该成员。使用 `push()`可以在该数组后添加一个元素。该函数没有返回值。

- pop()

  **Storage 中的动态数组或 bytes**(不包括 `string`) 有该成员。该函数用来移除数组的最后一个元素。调用该函数同时也会隐式的 delete 该元素。

### 注意点

1. 调用 storage 中数组的 `push()` 函数，其 gas 花费总是固定的，因为 storage 数组中的元素都是被初始化其默认值的；

   `pop()`函数的 gas 花费则是不一定的，其花费取决于被删除元素的大小，因为 pop 后还需要执行类似于 `delete`操作的清理动作。

2. 如果要要在 `external` 函数中使用多维数组，那么需要开启` ABI coder V2`。`public` 函数是默认允许使用二维数组。

3. 在拜占庭（Byzantium）模式之前，调用函数返回的动态数组是无法访问的。因此如果您调用的函数返回了动态数组，那么需要确保当前的 EVM 是运行在拜占庭模式。

### 知识点回顾

```js
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.6.0 <0.9.0;

contract ArrayContract {
  uint[2**20] m_aLotOfIntegers;
  // 内层数组是：长度为2基本类型是布尔值；外层数组是动态数组；
  bool[2][] m_pairsOfFlags;

  // storage 中的数组可以直接初始化某一个固定长度
  uint[2][] arrayStorage = new uint[2][](2);

  struct StructType {
    uint[] contents;
    uint moreInfo;
  }
  
  StructType s;

  function f(uint[] memory c) public {
      // 引用拷贝
      StructType storage g = s;
      g.moreInfo = 2; // 改变 g/s 引用的数据

      // 拷贝。文档的理由是 assigns a copy because g.contents is not
      // a local variable, but a member of a local variable.
      g.contents = c;
  }

  function setFlagPair(uint index, bool flagA, bool flagB) public {
      // 如果 index 超限则会报错
      m_pairsOfFlags[index][0] = flagA;
      m_pairsOfFlags[index][1] = flagB;
  }

  function changeFlagArraySize(uint newSize) public {
      // 使用 push 和 pop 是改变 storage 数组长度的唯一方式
      if(newSize < m_pairsOfFlags.length){
          while(m_pairsOfFlags.length > newSize)
            m_pairsOfFlags.pop();
      } else if(newSize > m_pairsOfFlags.length){
          while(m_pairsOfFlags.length < newSize)
            m_pairsOfFlags.push();
      }
  }

  function clear() public {
      delete m_pairsOfFlags;
      delete m_aLotOfIntegers;

      // delete 即将数据重置为初始值，与下述方法有同样的效果
      m_pairsOfFlags = new bool[2][](0);
  }

  bytes m_byteData;

  function byteArrays(bytes memory data) public {
      // 在存储上，bytes各个元素之间是没有 padding 的。但是在行为上，你可以把它当做 uint8[]
    m_byteData = data;
    for(uint i = 0; i < 7; i++)
        m_byteData.push();
    m_byteData[3] = 0x08;
    delete m_byteData[2];
  }

  function addFlag(bool[2] memory flag) public returns (uint) {
      m_pairsOfFlags.push(flag);
      return m_pairsOfFlags.length;
  }

   // 内存数组测试 
  function createMemoryArray(uint size) public pure returns (bytes memory){
      // 内存中的数组需要通过 new 创建并指定固定长度
      uint[2][] memory arrayOfPairs = new uint[2][](size);

      arrayOfPairs[0] = [uint(1), 2];

      bytes memory b = new bytes(200);
      for(uint i = 0; i < b.length; i++){
          b[i] =  bytes1(uint8(i));
      }
      return b;
  }
}

```

### 数组切片

数组切片本身并不是一个数组类型，但是可以隐式转化为对应基本类型的数组。也就是说没有任何变量可以把数组切片当成自己类型，数组切片总是用于中间表达式中。

**目前数组切片仅仅只能用于 calldata 数组。**

#### 切片方法

```JS
x[start:end];
x[:end];
x[start:];
```

- 切片的时候，包括 `x[start]`的元素，不包括 `x[end]`的值；
- 切片的时候，`start` 和 `end` 可以省略，`start` 默认是0， `end` 默认是数组的长度；
- `start`的值最小为 0， `end`的值最大为数组的长度；
- 如果 `start` 、`end` 超限或者 `start` >` end` 将会引发异常；

#### 主要用途

数组切片在 ABI 解码参数数据时非常有用。下面实现了一个代理（Proxy）合约，对传入的参数先进行解码校验，再进行 `delegatecall`：

```JS
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.8.5 <0.9.0

contract Proxy {
    address client;

    contract(address _client){
        client = _client;
    }

    function forward(bytes calldata _payload) external {
        bytes4 sig = bytes4(_payload[:4]);
        // 因为 truncating behaviours， bytes4(_payload) 并不能直接使用

        if(sig == bytes4(keccak256("setOwner(address)"))){
            address owner = abi.decode(_payload[4:], (address));
            require(owner!=address(0),"Address of Owner cannot be zero.");
        }
        (bool status,) = client.delegatecall(_payload);
        require(status, "Foward call failed");
    }
}
```

