## 工作日记 2018-06月

### 2018-07-05
- NPT 代币合约调试完成，ropsten 测试链 测试通过，测试内容覆盖如下：
- 第一步：
- 由 A 部署合约，由 A 私钥签发交易，mint to A, mint to B 匀成功；
- 由 A 部署合约，由 B 私钥签发交易，mint to B, mint to A 匀失败；
- 第二步：
- 由 A 私钥签发交易 transferOwnership to B，成功；
- owner to B 后，由 A 私钥签发交易 mint to A, mint to B 匀失败；
- owner to B 后，由 B 私钥签发交易 mint to A, mint to B 匀成功；
- 第三步：
- owner to B 后，由 A 私钥签发交易 transferOwnership to A 失败；
- owner to B 后，由 B 私钥签发交易 transferOwnership to A 成功；



