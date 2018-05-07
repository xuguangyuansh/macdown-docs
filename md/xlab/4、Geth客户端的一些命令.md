#四、Geth客户端的一些命令
##1、进入 Geth 控制台
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在前一篇中我们提到，可以用下面的命令来建立一个新的私有链，现在来对其做一些解释：<br/>

```
geth --datadir "./" --nodiscover console 2>>geth.log
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;–-datadir 代表文件夹地址<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;–-nodiscover 代表该链条不希望被其他节点发现<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;console 代表进入 geth 控制台，当然从控制台退出也很简单，只要打入 exit 即可<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2>>geth.log 表示将相关日志输出到文件 geth.log 中去<br/>
##2、Geth命令行中的 eth.accounts
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;该命令可以查看到当前区块链中共有哪些账号，以及每个账号的公钥地址。每一个以太坊账户都有一对公钥和私钥组成，从私钥到公钥采用单向加密算法。
##3、新增账户
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们可以使用 ```personal.newAccount("123")```来新建一个账户，其中 123 是账户的密码，账户在钱包中默认是以 Account 1、Account 2 等形式显示，用户可以在钱包中随意修改，因为以太坊并不是以该名称来唯一确定一个用户的，而是使用公钥地址。
##4、在两个账户之间转移以太币
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;前面说过，Geth 是一个以太坊客户端，虽然它也能创建新的以太坊网络，因此它具备在账户之前转移以太币的能力。但由于账户地址字符串太长，我们可以设置一个变量 acc0/acc1 分别代表 accounts[0] 及 accounts[1]，另外设置要转移 0.01个以太币

```
> acc0 = eth.accounts[0]
"0x177362b669fba1b7aebdb918c891f658eb3f4c84"
> acc1 = eth.accounts[1]
"0x7cb3d9d8c306fc8fe4d1f2f01b08f44a704e177f"
> amount = web3.toWei(0.01)
"10000000000000000"
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接下来我们可以使用 ```eth.sendTransaction```来将 0.01 个以太币从 acc0 转移到 acc1 中。
> eth.sendTransaction({from: acc0, to: acc1, value: amount})

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此时会出现下面的错误

![交易失败][transaction-fail]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这是以太坊的一个保护机制，每隔一段时间账户会被自动锁定，这个时候的账户只能入不能出，除非把该账户解锁

```
> personal.unlockAccount(acc0)
Unlock account 0x177362b669fba1b7aebdb918c891f658eb3f4c84
Passphrase: 
true
>

```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;之后再次执行命令 ```eth.sendTransaction({from: acc0, to: acc1, value: amount})```，结果如下

```
> eth.sendTransaction({from: acc0, to: acc1, value: amount})
"0x7cb3d9d8c306fc8fe4d1f2f01b08f44a704e177f"
> eth.getBalance(acc1)
10000000000000000
>
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以看到这时账户 acc1 有数值10000000000000000了，其等于 0.01 ether 币，可以使用命令 ```web3.fromWei(10000000000000000,"ether")```进行换算

```
> web3.fromWei(10000000000000000,"ether")
"0.01"
```

##5、Ether币的基本单位
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ether币的最小单位是 Wei，也是命令行默认的单位，每 1000 进一个单位，依次是：
<pre>
kwei (1000 Wei)
mwei (1000 KWei)
gwei (1000 mwei)
szabo (1000 gwei)
finney (1000 szabo)
ether (1000 finney)
</pre>
##6、开始挖矿 & 停止挖矿
```
> miner.start() //开始挖矿
true
> miner.stop() //停止挖矿
true
>
```
##7、部署合约
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;部署合约需要消耗 gas，所以也需要对账户进行解锁，否则不会允许以太币流出，解锁后，将 remix-ide 中合约的编译结果“DEPLOY”的内容直接贴入命令行即可，这部分内容在下一篇中会讲到，这里先部署一个笔者已经编译好的合约，其原码为：

```
pragma solidity ^0.4.23;
contract DemoTypes {
    function f(uint a) public pure returns (uint b) {
        uint result = a * 8;
        return result;
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;编译后的 deploy 内容为：

```
var demotypesContract = web3.eth.contract([{"constant":true,"inputs":[{"name":"a","type":"uint256"}],"name":"f","outputs":[{"name":"b","type":"uint256"}],"payable":false,"stateMutability":"pure","type":"function"}]);
var demotypes = demotypesContract.new(
   {
     from: web3.eth.accounts[0], 
     data: '0x608060405234801561001057600080fd5b5060c08061001f6000396000f300608060405260043610603f576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063b3de648b146044575b600080fd5b348015604f57600080fd5b50606c600480360381019080803590602001909291905050506082565b6040518082815260200191505060405180910390f35b600080600883029050809150509190505600a165627a7a723058207c086fca68eaa0dc5e7ef06ff12187e9d98d193078b802ff524ac430417037d70029', 
     gas: '4700000'
   }, function (e, contract){
    console.log(e, contract);
    if (typeof contract.address !== 'undefined') {
         console.log('Contract mined! address: ' + contract.address + ' transactionHash: ' + contract.transactionHash);
    }
 })
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;进行部署

![命令行部署合约][deploy-contract]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;等待片刻，合约就被部署到挖矿挖出的区块中了, 按下回车代表成功。此时输入合约部署的实例 demotypes, 可以看到demotypes 的详情。

![查看合约详情][demotypes]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;之后就可以使用合约名和函数名调用智能合约。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;


[transaction-fail]:
[deploy-contract]:
[demotypes]:





















