# 十一、合约安全之 DAO 漏洞分析

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先，这篇文章的较多内容翻译自 [Analysis of the DAO exploit](http://hackingdistributed.com/2016/06/18/analysis-of-the-dao-exploit/)。因为笔者在进行这部分研究时，发现 8比特的《The DAO的漏洞利用分析》应属机器翻译，语句十分不通顺，后笔者虽通过寻找原英文文章，完成了这部分研究，但为方便后来到的小伙伴，这里笔者还是进行了一次全手工翻译。同时，引用了 [The DAO 事件，区块链征途上的一场暴风雨](https://www.jianshu.com/p/431cddaea961) 的部分结构内容，以帮助说明，如关键词这个思路。转载请注明原&译文出处，以下由译文开篇。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我相信大家都已经听说了[关于 DAO 被盗高达1.5亿美元的大新闻](https://news.ycombinator.com/item?id=11921900)，黑客正是利用了[以太坊的递归发送漏洞](https://vessenes.com/more-ethereum-attacks-race-to-empty-is-the-real-deal/)。<br/>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这篇文章将有潜在的可能性，最终成为第一个关于该事件的系列文章，解构和解释在技术层面上的错误，同时提供一个时间表来追踪攻击者在以太坊区块链上的行为。第一篇文章将会关注攻击者如何在DAO中窃取了这么多钱。

## 1、多阶段攻击
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;针对 DAO 的这次攻击显然不是一件小事。现在，我们已经知道，一些具体的编程模式使得 DAO 十分脆弱，同时 DAO 的创建者们在早前对框架代码进行更新时，也修复了这个问题。但讽刺的是，当他们在编写博客并庆祝胜利的时刻，黑客也正在准备部署一项攻击，目标与他们刚刚修复的那个功能相同，该功能漏洞可以耗尽 DAO 中的所有资金。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;让我们重新概览一下这次攻击。首先，攻击者分析了 DAO.sol，并注意到其中的 splitDAO 方法不够严谨，可以利用如我们上面提到的 “递归发送”模式进行攻击。由于这个方法是在最后位置更新用户的余额和总额，所以如果我们能在这个动作发生前，再次调用 splitDAO 方法，我们就可以进行无限次的递归，来转移任何数量的资金。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;完整代码在 [Slockit/DAO](https://github.com/slockit/DAO/blob/v1.0/DAO.sol)，即 github slockit/DAO 代码的 tag v1.0。

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

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;基本思路如下：


## 工





&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;









## 工


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

## 工


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;













[]：