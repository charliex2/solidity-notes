# 映射 Mapping

映射只能存在于 storage 数据位置。因此其智能作为状态变量或者作为函数内的 storage 引用或者作为库函数的参数，不能作为合约共有函数的参数和返回值。

基于此，包含映射的数组和结构体也必须是存储与 storage 中的。

映射类型的声明方式为 `mapping(_KeyType => _ValueType)`。

**Mapping 可以当做一个 Hash表，在实际的初始化的过程中，每一个可能的键都别指向了该数据类型的初始值。另外与哈希表不同的是，Solidity mappding 中的键存的是 keccak256 哈希。**

## 数据类型

- KeyType 支持的数据类型

  可以值任何**基本数据类型**，即内建类型、**bytes**、**string** 、合约、枚举类型。但不允许是映射、结构体和除了 bytes、string之外的数组类型，已经用户自定义数据结构。

- ValueType

  任何数据类型。



## Getter

如果将 Mapping 声明为 public，Solidity 将自动创建一个 getter 函数，参数为 _KeyValue，返回对应的 _ValueType。如果 _ValueType 如果是一个函数或者 Mapping，则该 getter 函数的参数需要依次传入该 _KeyType。



## 示例

```js
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.4.22 <0.9.0;

contract SimpleERC20 {
    // 余额，地址 => 余额
    mapping(address => uint) private _balances;

    // 某一个用户 给 另一个用户批准了多少钱的提款权限
    mapping(address => mapping(address => uint)) private _allowances;

    event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);

    function allowance(address owner, address spender) public view returns (uint){
        return _allowances[owner][spender];
    }

    // 提款
    // address sende: 从哪里提款
    // address recipient: 收款账号
    // amount: 提款金额
    function transferFrom(address sender, address recipient, uint amount) public returns (bool){
        require(_allowances[sender][msg.sender] >= amount, "ERC20: Allowance is not enough.");
        _allowances[sender][msg.sender] -= amount;
        _transfer(sender, recipient, amount);
        return true;
    }

    // 批准提款
    function approval(address spender, uint amount) public returns (bool){
        require(spender != address(0), "ERC20: not allowed to approve to the zero address");

        // 这里这么处理肯定是错的，因为没有检查余额；
        _allowances[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    // internal，只允许本类或者子类调用
    function _transfer(address sender, address require, uint amount) internal {
        require(sender != address(0), "ERC20: Not allowed transfer to zero amount");
        require(recipient != address(0), "ERC20: transfer to the zero address");
        require(_balances[spender]>=amount, "ERC20: Not enough funds");

        _balances[sender]-=amount;
        _balances[require]+=amount;
        emit Transfer(sender, recipient, amount);
    }
}
```

