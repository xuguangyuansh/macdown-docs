# 十二、合约安全之 Parity 第一次漏洞分析

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;截止目前，Parity 多重签名钱包共发生过两次安全事件，第一次发生在 2017年07月19日，涉及 Parity 1.5 及以上版本，造成 15万以太币约 3000万美元被盗，第二次发生在 2017年11月07日，致使约 50万枚以太币被锁在合约中无法取出，当时价值大约 1.5亿美元，本篇先对发生于 7月19日的第一次安全漏洞做一下分析，下一篇再分析 2017年11月7日的安全漏洞。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;黑客向每个有漏洞的合约发送了两笔交易：第一笔交易用来获取多重签名钱包的拥有权限，第二笔交易是转移合约上的全部资金。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于官方代码地址 [paritytech/parity](https://github.com/paritytech/parity) 已无法选择 tag 1.5，因此可以 [从这里根据 git 日志ID 进入](https://github.com/paritytech/parity/blob/4d08e7b0aec46443bf26547b17d10cb302672835/js/src/contracts/snippets/enhanced-wallet.sol) 来查看完整代码。

## 攻击分析
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第一步：成为合约的 owner

```js
// enhanced-wallet.sol
// gets called when no other function matches
function() payable {
	// just being sent some cash?
	if (msg.value > 0)
		Deposit(msg.sender, msg.value);
  	else if (msg.data.length > 0)
		_walletLibrary.delegatecall(msg.data);
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过往这个合约地址转账一个`value = 0, msg.data.length > 0`的交易，以执行`_walletLibrary.delegatecall`分支。由于通过 json-rpc 调用以太坊智能合约时，`to`参数为合约地址，而要调用的合约方法会经编码后，放在`data`参数中，因此代码` _walletLibrary.delegatecall(msg.data)`理论上能无条件的调用合约内的任何一个函数，本次安全事件就是黑客调用了一个叫做`initWallet`的函数：

```js
// constructor - just pass on the owner array to the multiowned and
// the limit to daylimit
function initWallet(address[] _owners, uint _required, uint _daylimit) {
	initDaylimit(_daylimit);
	initMultiowned(_owners, _required);
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注意参数列表中的`_owners`，因为是多重签名合约，所以是`address[]`即地址数组，该函数原本的作用是用多重所有者的地址列表来初始化钱包，函数会继续向底层调用`initMultiowned `函数：

```js
// constructor is given number of sigs required to do protected "onlymanyowners" transactions
// as well as the selection of addresses capable of confirming them.
function initMultiowned(address[] _owners, uint _required) {
	m_numOwners = _owners.length + 1;
	m_owners[1] = uint(msg.sender);
	m_ownerIndex[uint(msg.sender)] = 1;
	for (uint i = 0; i < _owners.length; ++i) {
		m_owners[2 + i] = uint(_owners[i]);
		m_ownerIndex[uint(_owners[i])] = 2 + i;
	}
	m_required = _required;
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;经过这一步，合约的所有者就被改变了，相当于获取了 Linux 系统的 root 权限。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;因此，问题的关键就在于，上面的`initWallet`没有检查以防止在合约初始化后再次调用到`initMultiowned`，进而使得合约的所有者被改成黑客。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第二步: 转账，以`owner`身份调用`execute`函数，提取合约余额到黑客的地址：

```js
function execute(address _to, uint _value, bytes _data) external onlyowner returns (bytes32 o_hash) {
    // first, take the opportunity to check that we're under the daily limit.
    if ((_data.length == 0 && underLimit(_value)) || m_required == 1) {
        // yes - just execute the call.
        address created;
        if (_to == 0) {
            created = create(_value, _data);
        } else {
            if (!_to.call.value(_value)(_data))
                throw;
        }
        SingleTransact(msg.sender, _value, _to, _data, created);
    } else {
        // determine our operation hash.
        o_hash = sha3(msg.data, block.number);
        // store if it's new
        if (m_txs[o_hash].to == 0 && m_txs[o_hash].value == 0 && m_txs[o_hash].data.length == 0) {
            m_txs[o_hash].to = _to;
            m_txs[o_hash].value = _value;
            m_txs[o_hash].data = _data;
        }
        if (!confirm(o_hash)) {
            ConfirmationNeeded(o_hash, msg.sender, _value, _to, _data);
        }
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注意函数第一行后面的修改器限制为`onlyowner`，黑客进行上面的动作就是为了突破该限制。

```js
// simple single-sig function modifier.
modifier onlyowner {
    if (isOwner(msg.sender))
        _;
}
```

## 解决方案：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过上面的分析可以看到，核心问题在于越权的函数调用，Parity钱包合约接口必须精心设计和明确定义访问权限，或者更进一步说，Parity钱包的合约的设计必须符合某种成熟的模式，或者标准，Parity钱包合约代码部署前最好交由专业的机构进行评审。否则，一个不起眼的代码就会让你丢掉所有的钱。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

















