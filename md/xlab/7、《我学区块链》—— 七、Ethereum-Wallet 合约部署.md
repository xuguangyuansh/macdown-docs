#7、Ethereum-Wallet合约部署

##1、Ethereum-Wallet 部署合约
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在 Ethereum-Wallet 主界面中点击 Contracts，可以进入到合约管理界面，点击 Deploy New Contract 可以部署一个新合约。

![合约界面][contract-page]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;选择一个账户用来部署合约，账户需要有以太币，因为部署合约时需要消耗 gas。

![选择账户部署合约][contract-account]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在界面下部，贴入下面的代码

```
pragma solidity ^0.4.21;
contract Token {
    mapping(address => uint) public balancesOf;
    address public owner;

    function Token() public {
        owner = msg.sender;
        balancesOf[msg.sender] = 10000;
    }

    function transfer(address _to, uint _value) public {
        if (balancesOf[msg.sender] < _value) return; // 避免转移出去的代币超过当前的存货
        if (balancesOf[_to] + _value < balancesOf[_to]) return; // 避免自己调用自己，或者递归调用
        balancesOf[msg.sender] -= _value;
        balancesOf[_to] += _value;
    }

    function mint(uint _amount) public {
        balancesOf[owner] += _amount;
    }
}
```
![贴入合约代码][contract-code]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;选择为部署合约准备提供的 gas 数量。
![部署合约gas值][contract-gas]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;点击界面底部的 Send，之后会提示输入密码。
![合约部署密码][contract-passwd]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;等待挖矿后，合约部署就会完成。

##2、调用智能合约
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;再次回到合约管理界面，选择刚刚部署的合约。

![选择运行合约][run-contract-select]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;选择要执行的合约方法，这里执行 mint，相当于对本代币进行了挖矿。

![执行合约方法][run-contract-mint]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;点击 execute 后，会提示输入调用该合约方法的账户密码。

![执行合约密码][run-contract-mint-passwd]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;等待一会，待本次调用被挖矿以后，再次回到合约管理界面，选择刚才的合约，Balance of 中输入合约所有人的地址，可以看到其已经具有了代币。

![显示合约主账户][run-contract-show-balance]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;到此 Ethereum-Wallet 部署和调用和合约的部分就介绍完了，这里只调用了 mint 方法，另一个方法 transfer，其作用是将代币从一个账户转移到其他账户，感兴趣的小伙伴可自行尝试。 

[contract-page]:
[contract-account]:
[contract-code]:
[contract-gas]:
[contract-passwd]:
[run-contract-select]:
[run-contract-mint]:
[run-contract-mint-passwd]:
[run-contract-show-balance]:





















