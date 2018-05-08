#五、Remix简单使用
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;目前以太坊智能合约的编辑器主要有在线的 http://remix.ethereum.org；由 remix-ide自己搭建的；以及 Mac，Linux 系统上的 Remix-app 三种。三者的使用方式一致，这里以 Mac 平台的 Remix-app 为例。
##1、使用 Remix-app 来编译合约
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;打开安装的 remix-app，在左侧填入以下代码，代码功能很简单，不做过多解释，只是将输入的数字乘以 8，再返回。

```
pragma solidity ^0.4.23;
contract DemoTypes {
    function f(uint a) public pure returns (uint b) {
        uint result = a * 8;
        return result;
    }
}
```

![Remix源码编辑界面][remix-source-code]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;点击 Details 按钮后可以看到智能合约编译后的一些产物，复制其中的 WEB3DEPLOY 中的代码到命令行，且 accounts[0] 账户处于解锁状态即可进行智能合约的部署

![合约编译结果][web3deploy]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在窗体右侧，切换到 settings 选项卡，可以选择 solidity 编译器版本，推荐选择稳定版本。

![编译器版本][solidity-version]

##2、使用 Remix-app 调试合约
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在合约编译通过的前提下，在右侧切换到 Run 选项卡，点击 Create 按钮，可以创建一个合约实例，在合约实例的函数 f 后面输入 100，点击 f，可以看到返回值为 800，说明合约可以工作。

![调试合约][debug-contract]

##3、Solidity 源代码分析
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个最简单的智能合约代码如下：

```
pragma solidity ^0.4.23;
contract DemoTypes {
    function f(uint a) public pure returns (uint b) {
        uint result = a * 8;
        return result;
    }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第一行 ```pragma solidity ^0.4.23;```是必须的，否则编译器将不知道该如何选择编译器，以及编译版本。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第二行，和传统面向对象语言中的类很想像，有构造函数，有继承，有变量，有function，也有抽象类等概念，由 Solidity所写的智能合约，经过编译后会由以太坊虚拟机 EVM 来部署执行。

```
contract DemoTypes {
    ...
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第三行

```
function f(uint a) public pure returns (uint b) {
    ...
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;前面说过，合约中包含方法和变量，```function f(uint a) public pure returns (uint b)```代表定义了一个名为 f 的方法，输入变量为 uint a，输出为 uint b。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;uint 代表无状态整形数字，即大于 0 的整数。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;uint 默认为 uint256，即最大值为 2 的 256次方，这个数字对于绝大多数的数学去处已经足够了。同理还有 uint8 等。
##4、Remix-ide本地部署
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于一些原因，可能还是需要使用在线编译器，而 http://remix.ethereum.org 的服务器在国外，速度较慢。因此，接下来还是补充一下 remix-ide 的本地部署相关内容，可以选择部署在内网服务器上，这样内网人员就可以一起用了。<br/>
<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先需要安装 nodejs 及 npm，且 node 版本需要 > 4.6 & < 9

```
sudo apt-get install -y nodejs nodejs-legacy git
sudo apt-get update
sudo apt-get install -y npm
sudo npm install -g nrm			# 安装 npm 源管理器
sudo npm install -g n			# 安装 node 多版本管理器
nrm use taobao					# 使用 taobao 的源
sudo n v9						# 使用 node v9 版
sudo chown -R $USER:$(id -g $USER) ~/.config
sudo npm install -g npm			# 在 node v9 环境下再次更新 npm
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接下来安装 remix-ide

```
sudo apt-get install -y git
git clone https://github.com/ethereum/remix-ide.git
cd remix-ide
npm install
npm run setupremix
npm start
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;执行成功后，通过浏览器访问 地址+8080（端口号），可打开本地部署的 remix-ide 在线版。

![本地remix-ide][local-remix-ide]

[remix-source-code]:
[web3deploy]:
[solidity-version]:
[debug-contract]:
[local-remix-ide]:












