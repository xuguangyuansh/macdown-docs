# 十一、合约安全之 DAO 漏洞分析

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先，再罗嗦几句 DAO 是什么，DAO 本质上是一个风险投资基金，是一个基于以太坊区块链平台的迄今为止世界上最大的众筹项目。可理解为完全由计算机代码控制运作的类似公司的实体，通过以太坊筹集到的资金会锁定在智能合约中，每个参与众筹的人按照出资数额，获得相应的DAO代币(token)，具有审查项目和投票表决的权利。投资议案由全体代币持有人投票表决，每个代币一票。如果议案得到需要的票数支持，相应的款项会划给该投资项目。投资项目的收益会按照一定规则回馈众筹参与人。<br/>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DAO 于2016年5月28日完成众筹，共募集1150万以太币，在当时的价值达到1.49亿美元。而 DAO 事件就是黑客利用合约中一段不严谨的编码，通过参数攻击，使一个方法的前半部分已经转移了代币，但在方法最后更新客户余额并结束前再次从头部执行本方法，进而不断的将代币转移到黑客的地址。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面结合代码来分析一下 DAO 事件涉及的漏洞，文章部分引用了 [Analysis of the DAO exploit](http://hackingdistributed.com/2016/06/18/analysis-of-the-dao-exploit/) 及 [The DAO 事件，区块链征途上的一场暴风雨](https://www.jianshu.com/p/431cddaea961) 的内容。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;完整代码在 [Slockit/DAO](https://github.com/slockit/DAO/blob/1.0.1/DAO.sol)，即 github slockit/DAO 代码的 tag v1.0.1。

## 先热个身
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;要想搞明白本次事件的漏洞原理，需要先掌握一个 Solidity 的编程技巧，很多人不明白为什么会进入递归，问题也正在于此。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先，Solidity 智能合约中有这样一条原则，Solidity 中签名不匹配任何函数方法时，将会触发回退函数。那么再看一个方法 send()，由于 send() 函数指定了一个空函数签名，所以当 fallback 函数存在时，它总是会调用它，下面是一个例子：

```js
contract Test {
	function() { x = 1; }
	uint x;
}

contract Caller {
	function callTest(address testAddress) {
		Test(testAddress).call('0xabcdefgh'); // hash does not exist
		// results in Test(testAddress).x becoming == 1.
	}
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;类似于 `send()` 的行为，还有一个变种写法 `address.call.value(ether to send)()`，这里比较巧妙，send 方法默认调用空函数签名，而这里直接模拟一个空函数签名调用。这将可以向任意地址发送 ether，如果目标 address 有 fallback 函数，也将会触发。到这里，不明白的小伙伴可以先记下这个流程，我们继续向下走，读完全部内容，相信很多小伙伴会发现，这很像是一条大蛇咬住了自己的尾巴。


## 攻击开始
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;针对 DAO 的这次攻击显然不是一件小事。现在，我们已经知道，一些具体的编程模式使得 DAO 十分脆弱，同时 DAO 的创建者们在早前对框架代码进行更新时，也修复了这个问题。但讽刺的是，当他们在编写博客并庆祝胜利的时刻，黑客也正在准备部署一项攻击，目标与他们刚刚修复的那个功能相同，该功能漏洞可以耗尽 DAO 中的所有资金。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先，攻击者分析了 DAO.sol，并注意到其中的 splitDAO 方法不够严谨，可以利用 “递归发送” 进行攻击。由于这个方法是在最后位置更新用户的余额和总额，所以如果我们能在这个动作发生前，再次调用 splitDAO 方法，就可以进行无限次的递归，来转移任何数量的资金，下面来看几段代码，截自源码 tag v1.0.1。

```js
function splitDAO(uint _proposalID, address _newCurator) noEther onlyTokenholders returns (bool _success) {
    // ...
    // XXXXX Move ether and assign new Tokens. Notice how this is done first!
    uint fundsToBeMoved = (balances[msg.sender] * p.splitData[0].splitBalance) / p.splitData[0].totalSupply;
    if (p.splitData[0].newDAO.createTokenProxy.value(fundsToBeMoved)(msg.sender) == false)
        // XXXXX This is the line the attacker wants to run more than once
        throw;
    // ...
    // Burn DAO Tokens
    Transfer(msg.sender, 0, balances[msg.sender]);
    withdrawRewardFor(msg.sender);  // be nice, and get his rewards
    // XXXXX Notice the preceding line is critically before the next few
    totalSupply -= balances[msg.sender];    // XXXXX AND THIS IS DONE LAST
    balances[msg.sender] = 0;   // XXXXX AND THIS IS DONE LAST TOO
    paidOut[msg.sender] = 0;
    return true;
}
```

```js
function withdrawRewardFor(address _account) noEther internal returns(bool _success) {
    if ((balanceOf(_account) * rewardAccount.accumulatedInput()) / totalSupply < paidOut[_account])
        throw;
    uint reward = (balanceOf(_account) * rewardAccount.accumulatedInput()) / totalSupply - paidOut[_account];
    if (!rewardAccount.payOut(_account, reward))    // XXXXX vulnerable
        throw;
    paidOut[_account] += reward;
    return true;
}
```

```js

function payOut(address _recipient, uint _amount) returns (bool) {
    if (msg.sender != owner || msg.value > 0 || (payOwnerOnly && _recipient != owner))
        throw;
    if (_recipient.call.value(_amount)()) { // XXXXX vulnerable
        PayOut(_recipient, _amount);
        return true;
    } else {
        return false;
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;很明显，从上到下是一个调用链，为了方便展示，省略了部分代码。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DAO 的模式是，如果用户不同意其他用户的投票，为了防止资金损失，他可以选择分裂出去，在分裂之前的投资所得收益他仍然能够得到应得的那一部分，分裂之后原先那个 DAO 的收益就与他无关。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码块第一个方法 `splitDAO` 就是执行分裂，最后一块 `payOut` 就是将资金转移到分裂的目标地址，此次攻击中该地址就是黑客提案的一个合约地址。注意方法内这一句 `_recipient.call.value(_amount)()`，它会转移 `_amount` 数量的代币到 `_recipient` 这个地址，并触发 `_recipient` 的 `fallback` 函数，若黑客在 `fallback` 函数中再去调用 DAO 合约的 `splitDAO` 方法，则最终又会回到黑客部署的 `fallback` 函数，这就成为了一个闭环，从 DAO 的角度 / 堆栈来看，就仿佛 `splitDAO` 在不停的调用自身，像是在递归，并将资金池中的代币不断的转到黑客的合约中。<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;OK，以上就是这次所谓的 “递归发送” 漏洞攻击，觉得绕可以多读几遍。

## 如何修复
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;修复方法也很简单，比如调整代码顺序，先计算可提取额度（由用户余额等信息来计算），之后立即清空用户余额，再进行划转，这样若子合约有反向调用 主 DAO 的 splitDAO 动作时，将重新计算，并得到可提取余额为 "零"，进而方法正常结束，以下为伪码，只说明思路：

```js
function splitDAO() {
	uint canWithdraw = cal...(balances[msg.sender]);
	totalSupply -= balances[msg.sender];
	balances[msg.sender] = 0;
	paidOut[msg.sender] = 0;
	if (canWithdraw > 0) {
		if (!(msg.sender.send(canWithdraw))) {
			throw;
		}
	}
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;或者，也可以设置一个标记位，当发现用户已被标记过时，则不再进行划转，以下同为伪码：

```js
function splitDAO() {
	if (withdrawMutex[msg.sender] == true)
		throw;
	withdrawMutex[msg.sender] = true;
	if (!(msg.sender.send(canWithdraw))) 
		throw;
	totalSupply -= balances[msg.sender];
	balances[msg.sender] = 0;
	paidOut[msg.sender] = 0;
	withdrawMutex[msg.sender] = false;	
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上是类似问题的一种解决思路，但要解决 DAO 中的递归问题，还要结合 DAO 的代码来看，以免影响原本的需求逻辑。DAO 的流程是先 `executeProposal` 再 `splitDAO`，所以 DAO 的修正方法是在 DAO.sol 中调整了标记位的位置，具体可在 slockit/DAO master 分支的 DAO.sol 中搜索 recursive 进行了解。

## 一点感悟
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先要看到，在 14年时国外的智能合约开发能达到这种水平，还是非常棒的，但同时要注意到，如果真能开发出这样水平的合约项目，那像这次事件中的漏洞一点都不难发现，连细心都用不上，若能够再冷静些，初始设计时就完全可以考虑到，一种可能是，当团队设计出这种模式时，太过兴高采烈，忘乎所以，才会放掉这么直白的漏洞，这也不怪其团队中 Green 说：“不要告诉任何人，他(Gun)是非常不负责的”，虽然 Green 仍然非常尊重 Gün。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;再者，区块链 / 智能合约的开发模式可能要区别于其他领域，因为最终产物体量较小，所以并不需要人海战术，但同时内部的严谨性又要求较高。出于不可撤消和不可篡改性，也许这部分工作需要寻找更优秀的人材，甚至大师型人物，搭建小而精的团队，或者一名开发配备 3～5 名白盒评审人员，这样算比较严谨了，但成本问题也是一个不得不考量的因素。

## 总结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;相信很多朋友会觉得那么久的事情了，以太坊一定已经修复了这个递归的 bug 了，但从前面的分析中可以看出，该 bug 本质是由 DAO 合约本身的不严谨造成的，而并非以太坊，以太坊后面发生了软分叉和硬分叉，不过是：一、不承认块高度从 1760000 开始的任何与 DAO 和 child DAO 相关的交易；二、将时间调到 DAO 受攻击以前，之后 DAO 代币持有者可以以 1以太币:100DAO 的汇率提取以太币，以此找回用户损失。但为什么事件过去这么久，没再听到有类似事情发生呢，笔者认为大部分是因为运气，而且大胆做以下猜测：

- 中小公司本着你有我也要有（代币）的思路，其合约普遍并不复杂，只是代币，供用户买入和赎回，且不包含对其他合约的转账，这样就没有凑齐该漏洞发生的场景要素。
- 很多大公司开发区块链产品做为他用，而不是代币，如物流追踪，合同签定，这样的智能合约攻击起来没有经济利益，黑客也就没有去 “光顾”。
- 还一些较大的公司，若同样要开发类似 DAO 这样的合约时，其也具备在合约安全方面进行较高 人材、技术、资金 等方面投入的基础，能够较完善的处理好安全问题，因此类似 DAO 的事件也就不会找上这样的公司。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;也就是说，如果你还按照最初的 DAO 那样编写代码，那么很抱歉，问题依旧，但是否会受到攻击，则是另外一个话题。

## 后记：1、关键词索引

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;内容引自 [The DAO 事件，区块链征途上的一场暴风雨](https://www.jianshu.com/p/431cddaea961)

### TheDAO：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DAO 是Decentralized Autonomous Organization（分布式自治组织）的简称，the DAO是一个基于以太坊区块链平台的迄今为止世界上最大的众筹项目。其目的是让持有theDAO代币的参与者通过投票的方式共同决定被投资项目， 整个社区完全自制， 并且通过代码编写的智能合来实现。 于2016年5月28日完成众筹，共募集1150万以太币，在当时的价值达到1.49亿美元。

### DAOToken
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DAO代币，可以由DAO项目的参与者用以太币兑换得到，拥有Token的参与者通过使用DAO Token，可对DAO项目中发表的提案进行投票与投资。

### ETH
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以太币，以太坊平台发行的数字货币。

### Ethereum
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以太坊，区块链技术平台之一。提供了去中心化的虚拟机环境以及图灵完备的脚本语言。参与者可以基于以太坊的脚本语言编写并且执行智能合约。

### child DAO
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;子DAO。 是指DAO中的Token持有者通过调用DAO智能合约中的split函数，创建的一个小型DAO智能合约。在创建过程中，持有者在原有DAO智能合约中的Token被销毁，存储在原DAO智能合约中对应的以太币被转移到新的DAO智能合约中。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;child DAO的设计是为了保护在DAO投票中处于弱势地位的Token持有者，通过创建child DAO给予他们一个小规模的可提议、投票以及分红的新的分布式自治组织的环境。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在此次the DAO事件中，黑客利用the DAO智能合约中split函数的漏洞，在the DAO Token被销毁前，多次转移以太币到Child DAO智能合约中，从而大规模盗取原the DAO智能合约中的以太币。

### Softfork
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;软分叉，指通过修改区块链中的共识协议，对新生成的区块中的数据进行限制。更新协议的新节点产生的区块会被所有节点认可，而未更新协议的旧节点产生的区块则不一定被所有节点认可。在DAO事件中，通过软分叉的方法可以限制黑客转出被盗的以太币，达到暂时冻结黑客账户的目的。

### Hardfork
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;硬分叉，指通过修改区块链中的共识协议，从而把区块链中的数据恢复到过去某一状态的方法。更新协议的新节点与未更新协议的旧节点对于所产生的区块互相不认可。在the DAO事件中，通过硬分叉，就能时光倒流，让the DAO恢复到事件发生之前的状态。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;要看到，此次针对 DAO 事件的硬分叉已经导致了以太坊直接撕裂成了 ETH 和 ETC（旧版），但最后还是没能解决问题，因为会存在重放攻击。新链上的交易广播到旧链上，交易依然能够成功，因而造成使用混乱。

### Miner
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;矿工，是指在基于PoW共识算法的区块链中通过计算复杂数学问题从而竞争获得记录区块权力以及获得相应电子货币作为回报的节点的参与者。

### Slock.it
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在德国注册的区块链和物联网解决方案提供商，也是DAO项目的发起方。联合创始人为Simon Jentzsch，Christoph Jentzsch以及StephanTual，其中Christoph Jentzsch从2014年起在以太坊项目中担任测试领导者，Stephan Tual在以太坊担任过CCO（首席文化官）。

### VitalikButerin
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以太坊创始人，俄罗斯籍加拿大人。于2013年，Vitalik发布了以太坊项目，2014年他发布了以太坊并且发行了以太币，筹得资金超过1400万美元。

## 后记：2、一些思考
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;内容引自 [The DAO 事件，区块链征途上的一场暴风雨](https://www.jianshu.com/p/431cddaea961)

### “黑客”犯法了吗？
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这是一个有意思的问题。若我们回到 DAO 的主页，其“主旨"页面如是说："The Dao is borne fromimmutable, unstoppable, and irrefutable computer code, operated entirely by itsmembers, and fueled using ETH which Creates DAO tokens." 中文翻译基本上是：DAO 创建于不可伪造、不可虚构、不可纂改的程序代码并完全由其成员自由支配，自主运行，流通的DAO代币是用以太币兑换而来。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这样的主旨若用常见的商业条款上的语言来解释，引用 Slock.it 的注释就是：“与the DAO创建相关的条款已在以他坊的区块链的智能合约上罗列，具体位置为：b9bc244d798123fde783fcc1c72d3bb8c189413。在这里对这些条款的解释及其他相关文献，交流内容都不可能替代或修改 DAO 的代码条款已经包括的义务或担保。但是，这些解释及批注仅仅是以投资者教育为目的，不能替代或改变区块链上的 DAO 代码条款本身所包含的意义。”

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;故常见的解读(包括Slock.it团队过去常用的诠释)是：黑客从其他the DAO用户处盗取资金，这种说法其实违背了the DAO的初衷。因为“被黑”或“被盗”这些字眼本身包含了对于the DAO使用者意图的假设。但是代码本身是不以使用者意图为转移的，它只是一码不差地执行代码化的智能合同。因而代码不可能“被黑”，只是被“正常使用”。 最形象的类比是有人把这个“被黑事件” 称为代码套利行为!

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;确实， the DAO的“三大不可”原则实际是让人类的错误及误差没有可乘之机，就是一个逻辑代码的乌托邦。人类的预期和意愿不作为一个输入变量，除非这些预期和意愿被准确无误地编译成代码。所以，根据the DAO的宗旨， 编程代码的“不可伪造、不可虚构、不可纂改”这三大原则才是对是相关行为是否违背the DAO宗旨的唯一考量。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果真的告到法院，其实很难判断会有怎样的结果， 很有可能法院会认定“黑客”是正常地操作程序，按照代码执行， 而试图改变the DAO原有智能合同代码，保护投资者的举动是在违规。在这种情况下，若"黑客“把Slock.it告上法院， 法院要求Slock.it把 "黑客"所得全部兑现并交付给“黑客”也是有可能的。

### 如何看待 Vitalik 的补救方案
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一旦发现漏洞，在很短时间内 Vitalik 担起责任，提议了软硬分叉两个方案，不能不说是一个非常优秀的程序设计师。但是 DAO 是一个完全去中心化，社区参与者通过编码自理的架构。在危机时刻中心人物的出现，并提议修改 DAO 的智能合同编程，对于一些理想主义者来说应该是一个巨大的失望，也确实有道德风险之疑。若严格遵从 DAO 的 “三个不可” 宗旨，确实 Vitalik 的补救方案有违这个宗旨。

### 还有哪些需要关注
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从法律，社会道德，技术，新技术实践过程等方面我们都有一些事实，然而恰恰是这次事件的未知部分才是值得我们密切关注的。 我们不知道：<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1、DAO 从表面来看是一个去中心化的系统，但是 DAO 的主要投资人是否也是区块链社区的主要领头人? 怎么能保证他们的 "救 DAO” 的行为是公正的，也就是出发点是为了保护大多数投资者的利益而不是把保护自己的利益建立在损害大多数普通投资者的基础上？<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2、另一方面，若没有一个中心的架构和主要负责团体，一个价值一亿多美元的复杂的编程系统，是否能及时在危机的时刻做出做合适的应对措施？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;若以太坊决定用任何分叉的手段作为解决方案，一个由机器来执行的智能合同竟然会撤销，推翻，回滚，那比现在的纸质合同也没有好到哪里去，这样的决定对以太坊的声誉带来的破坏会是深远的。但是什么也不做，严格遵从现有的 DAO 的智能合同，虽然可以避免被指责有道德风险，保护了 DAO 的宗旨，但是现实社会的运作和理念不是和 DAO 的 "黑客” 玩猫和老鼠的游戏，而视大多投资人的利益而不顾。所以不管做出什么决定，都会有不少人伤心。一个应该在实验室里发展试运行的社区管理模式在有太多未知的情况下，太早地投入到现实社会实际上演。














