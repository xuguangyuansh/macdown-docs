# XXX、用于生产环境的合约
## 1、代理调用
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;存在一种特殊的消息调用，叫做代理调用。合约可以在运行时动态的加载其他地址的代码，“存储”、“当前地址”、“余额”都是和调用合约挂钩的，只有代码是从被调用方中获取。这就使得我们可以在 Solidity 中使用库。比如为了实现复杂的数据结构，可重用的代码可以封装于库合约中。

## 2、日志
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们可以把数据存储在一个特殊索引的数据结构中。这个结构映射到区块层面的各个地方。在 Solidity 中把这个特性称为日志。合约在被创建出来后是不可以访问日志数据的。但是他们可以从区块链外面有效的访问这些数据。因为日志的部分数据是存储在 bloom filters 上的。我们可以用有效并且安全加密的方式来查询这些数据。即使不用下载整个区块链数据（轻客户端）也能找到这些日志。

## 3、创建
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;合约可以通过特殊的指令来创建其他合约。这些创建指令和普通的消息调用唯一区别是：负载数据被执行，结果作为代码被存储，调用者在栈里收到了新合约的地址。

## 4、自毁
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从区块链中移除代码的唯一方法是合约在它的地址上执行了`selfdestruct`操作。这个账号下剩余的以太币会发送给指定的目标，存储和代码从栈中删除。

```js
function destroy() {
	if (msg.sender == owner) {
		suicide(owner)
	}
}
```


## 智能合约的5种设计模式
[智能合约的5种设计模式](https://my.oschina.net/u/3790537/blog/1808773)











&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/><br/>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;另外 randomId：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最终向网络发送交易

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最终向网络发送交易


```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;基于以上两种方式，就可以在以太坊上开发出支持原币及ERC20代币的多币种钱包，兼容了 ERC20后，应该可以支持到不下50种币了。这里只是带大家理一个以太坊钱包的基本思路，理解了以上原理以后，以太坊的其他 api 也就可以自行摸索使用了，因为我们已经知道了以太坊 api 的大概流程是什么样的。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;























