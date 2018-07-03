# XXX、合约交易及费用估算

## 执行交易
- web3.eth.sendSignedTransaction：发送一个已经签名的交易；
- web3.eth.sendRawTransaction：使用当前账户地址，发送一个交易到网络；
- web3.eth.sendTransaction：连接到相邻节点，使用相邻节点账户地址，发送一个交易到网络；
- web3.eth.call：执行交易调用，但数据不会合并到区块链中，适合进行`查看`类方法调用；

## 交易参数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以`web3.eth.sendTransaction`为例，`web3.eth.sendTransaction(transactionObject [, callback])`中大多参数是可选的。

### 参数：
- Object - 要发送的交易对象。
	- from: String - 指定的发送者的地址。如果不指定，使用web3.eth.defaultAccount。
	- to: String - （可选）交易消息的目标地址，如果是合约创建，则不填.
	- value: Number|String|BigNumber - （可选）交易携带的货币量，以wei为单位。如果合约创建交易，则为初始的基金。
	- gas: Number|String|BigNumber - （可选）也叫 gasLimit 默认自动，交易可使用的gas，未使用的gas会退回。
	- gasPrice: Number|String|BigNumber - （可选）默认是自动确定，交易的gas价格，默认是网络gas价格的平均值 。
	- data: String - （可选）或者包含相关数据的字节字符串，如果是合约创建，则是初始化要用到的代码。
	- nonce: Number - （可选）整数，使用此值，可以允许你覆盖你自己的相同nonce的，正在pending中的交易11。
- Function - 回调函数，用于支持异步的方式执行7。

### 返回值
- String - 32字节的交易哈希串。用16进制表示。

如果交易是一个合约创建交易，请使用`web3.eth.getTransactionReceipt()`在交易完成后获取合约的地址。

## 估算费用(gas)
- 1、web3.js
```js
let result = web3.eth.estimateGas({
	to: '0xc4abd0339eb8d57087278718986382264244252f',
	data: '0xc6888fa10000000000000000000000000000000000000000000000000000000000000003'
});
console.log(result);	// '0x0000000000000000000000000000000000000000000000000000000000000015'
```

```js
let Web3 = require('web3');
let web3;

if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
} else {
    // set the provider you want from Web3.providers
    web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
}

let from = web3.eth.accounts[0];

//编译合约
let source = "pragma solidity ^0.4.0;contract Calc{  /*区块链存储*/  uint count;  /*执行会写入数据，所以需要`transaction`的方式执行。*/  function add(uint a, uint b) returns(uint){    count++;    return a + b;  }  /*执行不会写入数据，所以允许`call`的方式执行。*/  function getCount() constant returns (uint){    return count;  }}";
let calcCompiled = web3.eth.compile.solidity(source);

//得到合约对象
let abiDefinition = calcCompiled["info"]["abiDefinition"];
let calcContract = web3.eth.contract(abiDefinition);

//2. 部署合约
//2.1 获取合约的代码，部署时传递的就是合约编译后的二进制码
let deployCode = calcCompiled["code"];

//2.2 部署者的地址，当前取默认账户的第一个地址。
let deployeAddr = web3.eth.accounts[0];

//2.3 异步方式，部署合约
let myContractReturned = calcContract.new({
    data: deployCode,
    from: deployeAddr
}, function(err, myContract) {
	if (!err) {
		// 注意：这个回调会触发两次
		//一次是合约的交易哈希属性完成
		//另一次是在某个地址上完成部署

		// 通过判断是否有地址，来确认是第一次调用，还是第二次调用。
		if (!myContract.address) {
		} else {
			myContract.methods.transfer(recipientAddress,amount).estimateGas({from: tokenHolderAddress}, function(gasAmount) {
				console.log(gasAmount);
			});
		}
	}
});
```

- 2、Geth

```js
> eth.estimateGas({from:eth.accounts[1], to: eth.accounts[2], value:50000000000000})
21001
> eth.gasPrice
20000000000
```

## GasPrice
- https://ethgasstation.info/


## Eth 基本单位
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Ether 币最小的单位是 Wei，也是命令行默认的单位, 每 1000 个进一个单位。	

- kwei (1000 Wei)
- mwei (1000 KWei)
- gwei (1000 mwei)
- szabo (1000 gwei)
- finney (1000 szabo)
- ether (1000 finney)

### 进制转换
eth.toWei(0.001)		// 1000000000000 Wei
eth.toWei(3,'gwei')	// 3000000000 Wei















