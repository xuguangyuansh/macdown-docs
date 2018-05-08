#6、Mist、Ethereum-Wallet简单使用
##1、安装 Ethereum-Wallet
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Mist 是以太坊官方提供的以太坊浏览器，是因为其除了可以用于账户及以太币操作之外，还可浏览当前以太坊网络算力，已挖掘区块数，账户数，还可输入地址，访问 web 页面。Mist 与 Ethereum-Wallet 都可以使用发布包安装，而不必从源码自行构建，预编译版的址为```https://github.com/ethereum/mist/releases```。
<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Mist 在安装后，首次启动时会从网络下载一个自己的 geth 核心，这需要消耗一点时间，之后还要从网络同步最新的以太坊区块链状态，否则直接 launch application，会导致程序页面空白，所以还是不要急，耐心一点，等其跑完自己需要执行的内容。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于 Mist 与 Ethereum-Wallet 功能大体相同，只是比后者多了浏览以太坊状态的功能。而像创建账户，转移以太币，部署调用合约这些功能 Ethereum-Wallet 也全都具备，而且从笔者自己的经验来看，其相比 Mist 会少很多奇奇怪怪的问题，因此本次我们直接来看 Ethereum-Wallet。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Mac 环境执行以下命令可以连接到我们自己的私有链上：
```
/Applications/Ethereum\ Wallet.app/Contents/MacOS/Ethereum\ Wallet --rpc http://10.40.152.132:8545/
```

![Ethereum-Wallet首页][wallet-index]

##2、Ethereum-Wallet 转移以太币
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在 Wallet 界面，点击 Account 1 账户。

![选择账户1][select-account1]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在接下来的界面右侧，点击 Copy address 或 Transfer Ether & Tokens。

![账户界面][account-page]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;设置 from, to, ammount 并点击下方的 Send。

![转移以太币][send-transaction]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接下来会提示输入转出账户的密码，为防止自动计算的 gas 数量不够，导致交易失败，此处我们也可以修改最大允许消耗的 gas值。

![确认转出][transaction-gas]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;密码输入完成后，点击 Send Transaction，然后切换到 wallet，此时可在页面下部查看这笔交易的进度，11 of 12 是指目前已经得到 11 个区块的确认。

![交易进度][transaction-rate]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;点击进度条，可查看该笔交易的详细信息。

![交易详情][transaction-detail]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;稍等片刻，可以看到 Account 1 的金融增加了 100 以太币。

![交易完成][transaction-over]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;到此为止，Ethereum-Wallet 的账户间转账功能就介绍完了。

##3、Ethereum-Wallet 部署合约
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

##4、调用智能合约
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;再次回到合约管理界面，选择刚刚部署的合约。

![选择运行合约][run-contract-select]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

[wallet-index]:
[select-account1]:
[account-page]:
[send-transaction]:
[transaction-gas]:
[transaction-rate]:
[transaction-detail]:
[account1-100]:
[contract-page]:
[contract-account]:
[contract-code]:
[contract-gas]:
[contract-passwd]:
























