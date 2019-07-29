# 目录

- [概览](#概览)
- [快速入门](#快速入门)
  - [安装或引入](#安装或引入)
  - [初始化代码](#初始化代码)
- [合约](#合约)
  - [合约初始化](#合约初始化)
  - [合约API](#合约API)
  - [StakingContract](#StakingContract)
    - [staking](#staking)
    - [updateStakingInfo](#updateStakingInfo)
    - [unStaking](#unStaking)
    - [addStaking](#addStaking)
    - [GetStakingInfo](#GetStakingInfo)
  - [内置合约](#内置合约)
    - [CandidateContract](#CandidateContract)
    - [TicketContract](#TicketContract)
- [web3](#web3)
  - [web3 eth相关 (标准JSON RPC )](#web3-eth相关-标准json-rpc)
  - [新增的接口](#新增的接口)
    - [contract](#contract)
      - [getPlatONData](#contract.getPlatONData)
      - [decodePlatONCall](#contract.decodePlatONCall)
      - [decodePlatONLog](#contract.decodePlatONLog)

## 概览
> Javascript SDK是PlatON面向js开发者，提供的PlatON公链的js开发工具包

## 快速入门

### 安装或引入

通过node.js引入：

`cnpm i https://github.com/PlatONnetwork/client-sdk-js`

### 初始化代码

然后你需要创建一个web3的实例，设置一个provider。为了保证你不会覆盖一个已有的provider，比如使用Mist时有内置，需要先检查是否web3实例已存在。

```js
if (typeof web3 !== 'undefined') {
    web3 = new Web3(web3.currentProvider);
} else {
    const web3 = new Web3(new Web3.providers.HttpProvider('http://localhost:6789'));
}
```
## 合约
> PlatON对外提供的与链交互的API

### 合约初始化及接口调用

```
const cid = web3.version.network;
web3.[ContractName].init(web3,cid);

web3.[ContractName].[funcName]();
```
#### 示例

```
const cid = web3.version.network;
web3.StakingContract.init(web3,cid);

let params = [];
params.push('0x493301712671Ada506ba6Ca7891F436D29185821');
params.push('a11859ce23effc663a9460e332ca09bd812acc390497f8dc7542b6938e13f8d7');
params.push(0);

params.push(1);
params.push('0x12c171900f010b17e969702efa044d077e868082');
params.push('f71e1bc638456363a66c4769284290ef3ccff03aba4a22fb60ffaed60b77f614bfd173532c3575abe254c366df6f4d6248b929cb9398aaac00cbcc959f7b2b7c');
params.push('111111');
params.push('platon');
params.push('https://www.test.network');
params.push('supper node');
params.push(1000000000000000000000000);
params.push(1792);

web3.stakingContract.staking(...params).then(res=>{
    console.log(res); // {result:true,data:'0x03aba4a22fb60ffaed60b77f614bfd173532c357'}
}).catch(err=>{
    console.log(err);
});
```
### 合约API

#### StakingContract

##### Staking

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|typ|Number  |必选|表示使用账户自由金额还是账户的锁仓金额做质押，0: 自由金额； 1: 锁仓金额|
|benefitAddress|String  |必选|用于接受出块奖励和质押奖励的收益账户|
|nodeId|String  |必选|被质押的节点Id|
|externalId|String  |必选|外部Id(有长度限制，给第三方拉取节点描述的Id)|
|nodeName|String  |必选|被质押节点的名称(有长度限制，表示该节点的名称)|
|website|String  |必选|节点的第三方主页(有长度限制，表示该节点的主页)|
|details|String  |必选|节点的描述(有长度限制，表示该节点的描述)|
|amount|Number  |必选|质押的von|
|programVersion|Number  |必选|程序的真实版本，治理rpc获取|

##### updateStakingInfo

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|typ|Number  |必选|表示使用账户自由金额还是账户的锁仓金额，0: 自由金额； 1: 锁仓金额|
|benefitAddress|String  |必选|用于接受出块奖励和质押奖励的收益账户|
|nodeId|String  |必选|被质押的节点Id|
|externalId|String  |必选|外部Id(有长度限制，给第三方拉取节点描述的Id)|
|nodeName|String  |必选|被质押节点的名称(有长度限制，表示该节点的名称)|
|website|String  |必选|节点的第三方主页(有长度限制，表示该节点的主页)|
|details|String  |必选|节点的描述(有长度限制，表示该节点的描述)|

##### unStaking

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|nodeId|String  |必选|被质押的节点Id|

##### addStaking

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|nodeId|String  |必选|被质押的节点Id|
|typ|Number  |必选|表示使用账户自由金额还是账户的锁仓金额，0: 自由金额； 1: 锁仓金额|
|amount|Number  |必选|质押的von|

##### GetStakingInfo

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|nodeId|String  |必选|被质押的节点Id|


#### NodeContract

##### GetVerifierList

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
无

##### getValidatorList

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
无

##### getCandidateList

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
无


#### DelegateContract

##### delegate

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|typ|Number  |必选|表示使用账户自由金额还是账户的锁仓金额，0: 自由金额； 1: 锁仓金额|
|nodeId|String  |必选|委托的节点Id|
|amount|Number  |必选|委托的金额|

##### withdrewDelegate

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|stakingBlockNum|Number  |必选|质押唯一标识|
|nodeId|String  |必选|委托的节点Id|
|amount|Number  |必选|减持委托的金额|

##### GetRelatedListByDelAddr

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|addr|String  |必选|委托人的账户地址|

##### GetDelegateInfo

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|stakingBlockNum|Number  |必选|发起质押时的区块高度|
|delAddr|String  |必选|委托人的账户地址|
|nodeId|String  |必选|验证人的节点Id|


#### ProposalContract

##### submitText

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|verifier|String  |必选|提交提案的验证人|
|githubID|String  |必选|提案在github上的ID|
|topic|String  |必选|提案主题|
|desc|String  |必选|提案描述|
|url|String  |必选|提案URL|
|endVotingBlock|Number  |必选|提案投票截止块高|

##### submitParam

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|verifier|String  |必选|提交提案的验证人|
|githubID|String  |必选|提案在github上的ID|
|topic|String  |必选|提案主题|
|desc|String  |必选|提案描述|
|url|String  |必选|提案URL|
|endVotingBlock|Number  |必选|提案投票截止块高|
|paramName|String  |必选|参数名称|
|currentValue|String  |必选|当前值|
|newValue|Number  |必选|新的值|

##### submitVersion

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|verifier|String  |必选|提交提案的验证人|
|githubID|String  |必选|提案在github上的ID|
|topic|String  |必选|提案主题|
|desc|String  |必选|提案描述|
|url|String  |必选|提案URL|
|newVersion|String  |必选|升级版本|
|endVotingBlock|Number  |必选|提案投票截止块高|
|activeBlock|String  |必选|生效块高|

##### vote

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|verifier|String  |必选|提交提案的验证人|
|proposalID|String  |必选|提案ID|
|option|String  |必选|投票选项|
|programVersion|Number  |必选|节点代码版本|

##### declareVersion

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|activeNode|String  |必选|声明的节点，只能是验证人/候选人|
|version|String  |必选|声明的版本|

##### getProposal

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|proposalID |String |必选| 提案ID|

##### listProposal

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
无

##### getTallyResult

| 参数名 |类型|属性|参数说明|
|proposalID |String |必选| 提案ID|

##### getActiveVersion

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
无

##### getProgramVersion

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
无

##### listParam

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
无


#### RestrictingPlanContract

##### createRestrictingPlan

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|account|String  |必选|锁仓释放到账账户|
|Plan|Array  |必选|[{Epoch:Number,Amount:Number}],(Epoch：表示结算周期的倍数。Amount：表示目标区块上待释放的金额|

##### getRestrictingInfo

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|account|String  |必选|锁仓释放到账账户|


#### SlashContract

##### reportDoubleSign

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|data|String  |必选|证据的json值|

##### checkDuplicateSign

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|typ|String  |必选|代表双签类型, 1：prepare，2：viewChange|
|addr|String  |必选|举报的节点地址|
|blockNumber|Number  |必选|多签的块高|
