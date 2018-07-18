# XXX、ERC165、ERC721与以太坊养猫

## 1、什么是ERC165
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ERC165 要求合约提供其实现了哪些接口，这样在与合约交互时可以先调用些接口进行查询。

```js
/**
 * @title ERC165
 * @dev https://github.com/ethereum/EIPs/blob/master/EIPS/eip-165.md
 */
interface ERC165 {
    /**
     * @notice Query if a contract implements an interface
     * @param _interfaceId The interface identifier, as specified in ERC-165
     * @dev Interface identification is specified in ERC-165. This function
     * uses less than 30,000 gas.
     */
    function supportsInterface(bytes4 _interfaceId) external view returns (bool);
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;没错，ERC165 就只做了这一项提案。

## 2、什么是ERC721
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ERC721 继承了 ERC165，是 2017年9月20日由加拿大温哥华新创公司 Axiom Zen 正式推出，该公司的技术总监 Dieter Shirley 是 ERC721 标准的作者和主要贡献者，ERC721 协议建立的是一套非同质的「不可替代币」（Non Fungible Token，简称 NFT）标准。与 ERC20 类似，其定义了一套通用接口，方便第三方钱包或应用集成，与 ERC20 不同，在 ERC721 合约中，你的一单位代币不等于我的一单位代币，每枚代币都有自己的元数据属性，由于属性的不同，你我的一单位代币对以太币价格可能相差悬殊。

## 3、一款火暴的应用
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2017年下半年，有一阵子，以太坊网络突然特别拥堵，原因是兴趣了一款以太坊养猫的 Dapp 游戏，超级可爱的猫形象，再加上配种，繁殖和拍卖等丰富的玩法，风靡了币圈。 一时间币圈大大小小的人都在撸猫，以太坊网络不堪负荷。

<center>![erc721-cat][]</center>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;后续又出现了很多的类似的游戏，如网易招财猫，百度莱茨狗和加密鱼等等，不过玩法套路都是差不多。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这些游戏的核心就是遵循 ERC721 协议的以太坊合约。

## ERC721	未来发展与应用
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以往，基于 ERC20 协议，开发者可以发行同质代币，以进行 ICO、众筹 或用作其他应用产品的记账。现在，有了 ERC721 协议标准，将为以太坊应用领域带来新的局面，如加密收藏器、游戏装备、房屋、股票、债券所有权、交易审计等，如此，一个新时代的产权，所有权市场呼之欲出。


## 一般读者为什么需要了解 ERC721
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;主要是很多民众对投资 ICO 有兴趣，在评估是否投资时，需要花时间详细阅读该 ICO 项目的白皮书。这时，若某个 ICO 项目是发行的 ERC721 代币，若不了解 ERC721 协议所代表的涵意，有可能导致不清楚 ICO 的卖法，背后如何动作，甚至白皮书的真假也难以判断。因此，若想从事加密货币的投资，应该试着理解 ERC721 协议，提高视野，以降低投资风险。


[erc721-cat]:





















