# XXX、Truffle 高级技巧

## 部署带参构造函数合约
```js
var NPToken = artifacts.require("./NPToken");

module.exports = function(deployer) {
    deployer.deploy(NPToken, arg1, arg2, arg3);
}
```

## 父合约初始化
父合约构造函数无参时会自动初始化

## 父合约带参初始化
constructor() public CappedToken(CAP_SUPPLY)













