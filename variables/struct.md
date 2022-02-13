# 结构体 Struct

结构体（Structs）提供了一个定义新数据类型的方法。

结构体中不能包含一个它自身类型的成员。结构体可以作为 Mapping 的成员或者数组的基础类型。

## 新建结构体

```js
Funder({ addr: msg.sender, amount: msg.value })
```



```js
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.6.0 <0.9.0;

// Struct 可以定义在合约外，那么就可以被多个合约共享。
struct Funder {
    address addr;
    uint amount;
}

contract CrowdFunding {

    // 定义在合约内的结构体，只能被此合约和子类合约共享
    struct Campaign {
        address payable beneficiary;
        uint fundingGoal; 
        uint numFounders; // 参与众筹的人数
        uint amount; // 众筹的金额 
        mapping(uint => Funder) funders;
    }

    uint numCampaigns;

    // 该 mapping 中的值是一个 struct，这要求该 struct 也必须在 storage 中。
    mapping(uint => Campaign) campaigns;

    // 创建一个众筹
    function newCampaign(address payable beneficiary, uint goal) public returns (uint campaignID){
      	// 这个可以直接不用声明
        campaignID = numCampaigns++;

        // 大家可能想当然会如下这么写，但是这是无法编译通过的。
        // 因为 Campaign(beneficiary, goal, 0, 0) 在内存中创建了一个 Campaign，但是其中的 mapping 不允许存在于内存中。
        // campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0);
        Campaign storage c = campaigns[campaignID];
        c.beneficiary = beneficiary;
        c.fundingGoal = goal;
    }

    // 资助一个众筹
    function contribute(uint campaignID) public payable {
        Campaign storage c = campaigns[campaignID];
        c.funders[c.numFounders++] = Funder({ addr: msg.sender, amount: msg.value });
        c.amount += msg.value;
    }

    // 检查 Goal 是否已经达到
    function checkGoalReached(uint campaignID) public returns (bool reached){
        Campaign storage c = campaigns[campaignID];
        if(c.amount < c.fundingGoal){
            return false;
        }

        uint amount = c.amount;
        c.amount = 0;
        c.beneficiary.transfer(amount);
        return true;
    }
}
```

