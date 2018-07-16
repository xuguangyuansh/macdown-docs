# 十三、合约安全之 Parity 第二次漏洞分析

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Parity 多签钱包的第二次漏洞发生于 2017年11月07日，不同于 17年7月19日那次，本次不是资产被黑客盗走，而是合约底层被破坏，导致资产就在那，但却永远也取不出，就像驾船到太平洋最深处，投下一枚硬币，硬币就在那，但你可能再也无法找到它。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本次漏洞导致约 50万枚以太币被锁在多签智能合约里，当时价值大约 1.5亿美元。下面我们来分析下这次漏洞的原理。

## 漏洞回放
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2017年11月7日，漏洞发现者在Github 的 Parity 项目网站上建了一条题为 “anyone can kill your contract” 的问题，说自己意外地杀死了 Parity 多签库合约，并贴出他执行的交易信息。他在问题的跟随评论里进一步解释发生了什么：他本来不是合约的所有者（owner），但由于自己的合约未初始化，再次初始化时，他把自己变成了 “库合约” 的 owner，然后作为 owner 他调用了合约的 kill 方法，把库合约里的代码都抹去了。

<center>![parity-github-1][]</center>
<center>![parity-github-2][]</center>

## 漏洞分析
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Parity 钱包提供了一个多签合约的模板，用户使用模板可以用很少的代码，很容易生成自己的多签智能合约。实际业务逻辑都通过`delegatecall`内嵌式地交给库合约。这样做的一个主要好处是：多签合约的主逻辑（代码量较大）作为库合约只需在以太坊上部署一次，而不会作为用户多签合约的一部分重复部署，因此可以为用户节省部署多签合约所耗费的大量Gas。

问题代码：<br/>
```
git checkout ddc7b588dca4a8c6fa1ffe19a824e5b2344e41b5
```

```js
// gets called when no other function matches
function() payable {
	// just being sent some cash?
	if (msg.value > 0)
		Deposit(msg.sender, msg.value);
	else if (msg.data.length > 0)
		_walletLibrary.delegatecall(msg.data);
}
```
```js
// constructor - just pass on the owner array to the multiowned and
// the limit to daylimit
function initWallet(address[] _owners, uint _required, uint _daylimit) only_uninitialized {
	initDaylimit(_daylimit);
	initMultiowned(_owners, _required);
}

// constructor is given number of sigs required to do protected "onlymanyowners" transactions
// as well as the selection of addresses capable of confirming them.
function initMultiowned(address[] _owners, uint _required) only_uninitialized {
	m_numOwners = _owners.length + 1;
	m_owners[1] = uint(msg.sender);
	m_ownerIndex[uint(msg.sender)] = 1;
	for (uint i = 0; i < _owners.length; ++i) {
		m_owners[2 + i] = uint(_owners[i]);
		m_ownerIndex[uint(_owners[i])] = 2 + i;
	}
	m_required = _required;
}

// throw unless the contract is not yet initialized.
modifier only_uninitialized { if (m_numOwners > 0) throw; _; }
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参考上面几段代码，为了修复 2017年7月19日的那个 bug，确保初始化逻辑只能被执行一次，几段函数都增加了`only_uninitialized`。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里补充一个知识点，即当库合约的函数被调用时，它是运行在调用方的上下文里。而为了能被用户合约调用，本次事件中的库合约初始化函数都是 public 的。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这次的问题就出现在黑客直接调用了库合约的初始化函数，由于库合约本质上也不过是另一个智能合约，这次攻击调用使用的就是库合约本身的上下文，对于调用者而言，这个库合约是未经初始化的，而黑客通过初始化参数把自己设置的成了 owner，接下来又作为 owner 调用了 kill 函数，抹除了库合约的所有代码，这样所有依赖这个库合约的用户多签合约就都无法执行，而合约中的代币全部被锁在合约内无法转移。


## 修复方法
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;显然`only_uninitialized`的限制仍是不严谨的，而若想不发生本次的安全事件，可对进一步对`initWallet`、`initDaylimit`及`initMultiowned`添加`internal`限定类型，以禁止外部调用：

```js
// constructor - just pass on the owner array to the multiowned and
// the limit to daylimit
function initWallet(address[] _owners, uint _required, uint _daylimit) internal only_uninitialized {
	initDaylimit(_daylimit);
	initMultiowned(_owners, _required);
}

// constructor - stores initial daily limit and records the present day's index.
function initDaylimit(uint _limit) internal only_uninitialized {
	m_dailyLimit = _limit;
	m_lastDay = today();
}

// constructor is given number of sigs required to do protected "onlymanyowners" transactions
// as well as the selection of addresses capable of confirming them.
function initMultiowned(address[] _owners, uint _required) internal only_uninitialized {
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

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上只是一种修正方案，但 Parity 团队也许出于一些其他原因，并没有采用类似的方案进行修正，而且在 v1.10.0 版本后干脆移除了包含`initwallet`的库合约。

[parity-github-1]:
[parity-github-2]:
















