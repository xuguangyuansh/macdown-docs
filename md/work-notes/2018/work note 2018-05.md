## 工作日记 2018-05月
### 2018-05-14
完成 什么是区块链 到 用区块链投票 学习文章的发布。已发布到 csdn[《我学区块链》](https://blog.csdn.net/xuguangyuansh/article/category/7662671)。

### 2018-05-05
完成 MyEtherWallet 轻钱包 获取地址余额(setBalance) 及 发送交易(sendTx) 的代码追溯。

### 2018-05-16
- 结合 Ethereum-Wallet，完成 MyEtherWallet 在 Ganache-cli 上的以太坊原币交易；
- 通过交易，完成 MyEtherWallet 的 uiFuncs.js.sendTx 中 web3.eth.sendTransaction 及  ajaxReq.sendRawTx 区别分析。据测，走 web3 的应为代币交易，而后者是以太坊原币交易；
- 更名 etherwallt 为 etherwallet-debug，修改钱包默认连接到 localhost 私链，打印发送交易时 数据格式 日志（各环节日志 1、 2、 3、 4、，具体内容可参看 etherwallet-debug README.md），添加 setBalance 方法的代码追溯 图谱(docs-debug目录)，并将修改后的项目提交到 gitee.com；
- 明日计划：以太坊私链 ganache-cli 发行 ERC20 代币，跟踪代币交易数据格式。

### 2018-05-17
- 分析了纯手写遵循 ERC20 及 使用 zeppelin-solidity 框架开发代币 WTH（Wecash TH），结论后者流程更标准，通用接口的实现更严谨；
- 使用 Truffle 及 zeppelin-solidity 在 ganache-cli 发行了 ERC20 代币，编写了测试脚本 TutorialToken.js 及 测试合约 TestTutorialToken，在测试脚本中向 MyEtherWallet 钱包用户地址发送了用于测试的 WTH 代币；
- 使用 MyEtherWallet 钱包进行了代币（WTH: WecashTH）交易，打印了代币交易期间数据流转日志（各环节日志 1、 2、 3、 4、）内容可参看 etherwallet-debug README.md。
- 明日计划：1）根据交易数据流转日志，对比分析 原币 及 代币 交易编解码过程；2）将代币代码规范化后，提交到 gitee.com。

### 2018-05-18
- 分析原币及代币交易的数据格式，代币交易时 to 为合约地址，data 为 web3.sha3 的方法签名的前 10位 + 补全 64 位的参数列表；
- 方法签名计算法 e.g.: web3.sha3("balanceOf(address,uint256)")，也可将合约粘入 remix，待编译完成时 Details -> FUNCTIONHASHES 来查看；
- etherwallet-debug 补充了几代币交易的日志，修改已提交 gitee.com；
- tutorial-token 添加了测试代码，用于向 MyEtherWallet 钱包账户地址初始化一些 WTH（Wecash TH）代币；
- tutorial-token 已提交 gitee.com。
- 明日计划：1）完成用区块链投票 MetaMask 部分；2）编写 ERC20 代币 MarkDown。





