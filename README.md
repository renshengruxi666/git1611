## 交易接口定义
交易接口沿用以太坊消息结构，对于特定类型的交易(如举报)通过扩展tx.data字段来实现与以太坊交易接口的兼容
以太坊交易结构如下：

| **字段名**  | **长度**  |  **说明**  |
| ------------ | ------------ | ------------ |
| nonce | 8bytes | 当前账户发送的第几笔交易 |
| gasPrice | 32bytes | 当前交易的gas价格 |
| gas | 8bytes | 当前交易允许消耗的最大gas |
| to | 20bytes | 交易的目标地址(为nil时表示创建合约) |
| value | 32bytes | 当前交易转账的金额 |
| input | bytes | 当前交易携带的扩展数据，前2个字节为funcTypeid，后面是参数列表 |
| value | 32bytes | 当前交易转账的金额 |
| value | 32bytes | 当前交易转账的金额 |
| v | 32bytes | 签名v值 |
| r | 32bytes | 签名r值 |
| s | 32bytes | 签名s值 |

## tx.input扩展说明

- 字段说明

| **字段名**  | **位置**  |  **说明**  |
| ------------ | ------------ | ------------ |
| funcType | tx.input前2个字节 | 命令码 |

- funcType取值

<a name="top"></a>

## staking接口

1000: [发起质押](#staking_contract_1)

1001: [修改质押信息](#staking_contract_2)

1002: [增持质押](#staking_contract_3)

1003: [撤销质押](#staking_contract_4)

1004: [发起委托](#staking_contract_5)

1005: [减持/撤销委托](#staking_contract_6)

1100: [查询当前结算周期的验证人队列](#staking_contract_7)

1101: [查询当前共识周期的验证人列表](#staking_contract_8)

1102: [查询所有实时的候选人列表](#staking_contract_9)

1103: [查询当前账户地址所委托的节点的NodeID和质押Id](staking_contract_10)

1104: [查询当前单个委托信息](#staking_contract_11)

1105: [查询当前节点的质押信息](#staking_contract_12)

## 治理接口

2000: [提交文本提案](#submitText)

2001: [提交升级提案](#submitVersion)

2002: [提交参数提案](#submitParam)

2003: [给提案投票](#vote)

2004: [版本声明](#declareVersion)

2100: [查询提案](#getProposal)

2101: [查询提案结果](#getTallyResult)

2102: [查询提案列表](#listProposal)

2103: [查询生效版本](#getActiveVersion)

2104: [查询节点代码版本](#getProgramVersion)

2105: [查询可治理参数列表](#listParam)

## 举报惩罚接口

3000: [举报多签](#ReportDuplicateSign)

3001: [查询节点是否有多签过](#CheckDuplicateSign)

## 锁仓计划接口

4000: [创建锁仓计划](#CreateRestrictingPlan)

4100: [获取锁仓信息](#GetRestrictingInfo)

- 编解码协议

1. tx.input字段以及查询结果的data字段统一使用RLP编解码协议，各个接口参数列表详见以下表格
2. 查询结果沿用[以太坊的结果返回格式](query_result)

## staking接口

**以下staking相关接口的合约地址为:** 0x1000000000000000000000000000000000000002

<a name="staking_contract_1"></a>
<code style="color: purple;">1000. createStaking() </code>: 发起质押

[回到接口目录](#top)

|参数|类型|说明|
|---|---|---|
|funcType|uint16(2bytes)|代表方法类型码(1000)|
|typ|uint16(2bytes)|表示使用账户自由金额还是账户的锁仓金额做质押，0: 自由金额； 1: 锁仓金额|
|benefitAddress|20bytes|用于接受出块奖励和质押奖励的收益账户|
|nodeId|64bytes|被质押的节点Id(也叫候选人的节点Id)|
|externalId|string|外部Id(有长度限制，给第三方拉取节点描述的Id)|
|nodeName|string|被质押节点的名称(有长度限制，表示该节点的名称)|
|website|string|节点的第三方主页(有长度限制，表示该节点的主页)|
|details|string|节点的描述(有长度限制，表示该节点的描述)|
|amount|*big.Int(bytes)|质押的von|
|programVersion|uint32|程序的真实版本，治理rpc获取|

<a name="staking_contract_2"></a>
<code style="color: purple;">1001. editCandidate() </code>: 修改质押信息

[回到接口目录](#top)

|参数|类型|说明|
|---|---|---|
|funcType|uint16(2bytes)|代表方法类型码(1001)|
|benefitAddress|20bytes|用于接受出块奖励和质押奖励的收益账户|
|nodeId|64bytes|被质押的节点Id(也叫候选人的节点Id)|
|externalId|string|外部Id(有长度限制，给第三方拉取节点描述的Id)|
|nodeName|string|被质押节点的名称(有长度限制，表示该节点的名称)|
|website|string|节点的第三方主页(有长度限制，表示该节点的主页)|
|details|string|节点的描述(有长度限制，表示该节点的描述)|



<a name="staking_contract_3"></a>
<code style="color: purple;">1002. increaseStaking() </code>: 增持质押

[回到接口目录](#top)

入参：

|参数|类型|说明|
|---|---|---|
|funcType|uint16(2bytes)|代表方法类型码(1002)|
|nodeId|64bytes|被质押的节点Id(也叫候选人的节点Id)|
|typ|uint16(2bytes)|表示使用账户自由金额还是账户的锁仓金额做质押，0: 自由金额； 1: 锁仓金额|
|amount|*big.Int(bytes)|增持的von|






<a name="staking_contract_4"></a>
<code style="color: purple;">1003. withdrewStaking() </code>: 撤销质押(一次性发起全部撤销，多次到账)

[回到接口目录](#top)

|参数|类型|说明|
|---|---|---|
|funcType|uint16(2bytes)|代表方法类型码(1003)|
|nodeId|64bytes|被质押的节点的NodeId|

<a name="staking_contract_5"></a>
<code style="color: purple;">1004. delegate() </code>: 发起委托

[回到接口目录](#top)

|参数|类型|说明|
|---|---|---|
|funcType|uint16(2bytes)|代表方法类型码(1004)|
|typ|uint16(2bytes)|表示使用账户自由金额还是账户的锁仓金额做委托，0: 自由金额； 1: 锁仓金额|
|nodeId|64bytes|被质押的节点的NodeId|
|amount|*big.Int(bytes)|委托的金额(按照最小单位算，1LAT = 10**18 von)|

<a name="staking_contract_6"></a>
<code style="color: purple;">1005. withdrewDelegate() </code>: 减持/撤销委托(全部减持就是撤销)

[回到接口目录](#top)

|参数|类型|说明|
|---|---|---|
|funcType|uint16(2bytes)|代表方法类型码(1005)|
|stakingBlockNum|uint64(8bytes)|代表着某个node的某次质押的唯一标示|
|nodeId|64bytes|被质押的节点的NodeId|
|amount|*big.Int(bytes)|减持委托的金额(按照最小单位算，1LAT = 10**18 von)|

<a name="staking_contract_7"></a>
<code style="color: purple;">1100. getVerifierList() </code>: 查询当前结算周期的验证人队列

[回到接口目录](#top)

入参：

|名称|类型|说明|
|---|---|---|
|funcType|uint16(2bytes)|代表方法类型码(1100)|

<a name="query_result">查询结果统一格式</a>

|名称|类型|说明|
|---|---|---|
|result|bool| true:表示结果OK，false:表示查询遇到异常，当为false时，err不能为空|
|data|string| json字符串的查询结果，具体格式参见以下查询相关接口返回值 |
|err|string| 当result为false时的错误提示信息|

> 注：以下查询接口（platon_call调用的接口）如无特殊声明，返回参数都按照上述格式返回

返参： 列表

|名称|类型|说明|
|---|---|---|
|NodeId|64bytes|被质押的节点Id(也叫候选人的节点Id)|
|StakingAddress|20bytes|发起质押时使用的账户(后续操作质押信息只能用这个账户，撤销质押时，von会被退回该账户或者该账户的锁仓信息中)|
|BenefitAddress|20bytes|用于接受出块奖励和质押奖励的收益账户|
|StakingTxIndex|uint32(4bytes)|发起质押时的交易索引|
|ProgramVersion|uint32|被质押节点的PlatON进程的真实版本号(获取版本号的接口由治理提供)|
|StakingBlockNum|uint64(8bytes)|发起质押时的区块高度|
|Shares|*big.Int(bytes)|当前候选人总共质押加被委托的von数目|
|ExternalId|string|外部Id(有长度限制，给第三方拉取节点描述的Id)|
|NodeName|string|被质押节点的名称(有长度限制，表示该节点的名称)|
|Website|string|节点的第三方主页(有长度限制，表示该节点的主页)|
|Details|string|节点的描述(有长度限制，表示该节点的描述)|
|ValidatorTerm|uint32(4bytes)|验证人的任期(在结算周期的101个验证人快照中永远是0，只有在共识轮的验证人时才会被有值，刚被选出来时也是0，继续留任时则+1)|

<a name="staking_contract_8"></a>
<code style="color: purple;">1101. getValidatorList() </code>: 查询当前共识周期的验证人列表

[回到接口目录](#top)

入参：

|名称|类型|说明|
|---|---|---|
|funcType|uint16(2bytes)|代表方法类型码(1101)|

返参： 列表

|名称|类型|说明|
|---|---|---|
|NodeId|64bytes|被质押的节点Id(也叫候选人的节点Id)|
|StakingAddress|20bytes|发起质押时使用的账户(后续操作质押信息只能用这个账户，撤销质押时，von会被退回该账户或者该账户的锁仓信息中)|
|BenefitAddress|20bytes|用于接受出块奖励和质押奖励的收益账户|
|StakingTxIndex|uint32(4bytes)|发起质押时的交易索引|
|ProgramVersion|uint32(4bytes)|被质押节点的PlatON进程的真实版本号(获取版本号的接口由治理提供)|
|StakingBlockNum|uint64(8bytes)|发起质押时的区块高度|
|Shares|*big.Int(bytes)|当前候选人总共质押加被委托的von数目|
|ExternalId|string|外部Id(有长度限制，给第三方拉取节点描述的Id)|
|NodeName|string|被质押节点的名称(有长度限制，表示该节点的名称)|
|Website|string|节点的第三方主页(有长度限制，表示该节点的主页)|
|Details|string|节点的描述(有长度限制，表示该节点的描述)|
|ValidatorTerm|uint32(4bytes)|验证人的任期(在结算周期的101个验证人快照中永远是0，只有在共识轮的验证人时才会被有值，刚被选出来时也是0，继续留任时则+1)|

<a name="staking_contract_9"></a>
<code style="color: purple;">1102. getCandidateList() </code>: 查询所有实时的候选人列表

[回到接口目录](#top)

入参：

|名称|类型|说明|
|---|---|---|
|funcType|uint16(2bytes)|代表方法类型码(1102)|

返参： 列表

|名称|类型|说明|
|---|---|---|
|NodeId|64bytes|被质押的节点Id(也叫候选人的节点Id)|
|StakingAddress|20bytes|发起质押时使用的账户(后续操作质押信息只能用这个账户，撤销质押时，von会被退回该账户或者该账户的锁仓信息中)|
|BenefitAddress|20bytes|用于接受出块奖励和质押奖励的收益账户|
|StakingTxIndex|uint32(4bytes)|发起质押时的交易索引|
|ProgramVersion|uint32(4bytes)|被质押节点的PlatON进程的真实版本号(获取版本号的接口由治理提供)|
|Status|uint32(4bytes)|候选人的状态(状态是根据uint32的32bit来放置的，可同时存在多个状态，值为多个同时存在的状态值相加0: 节点可用 (32个bit全为0)； 1: 节点不可用 (只有最后一bit为1)； 2： 节点出块率低(只有倒数第二bit为1)； 4： 节点的von不足最低质押门槛(只有倒数第三bit为1)； 8：节点被举报双签(只有倒数第四bit为1))|
|StakingEpoch|uint32(4bytes)|当前变更质押金额时的结算周期|
|StakingBlockNum|uint64(8bytes)|发起质押时的区块高度|
|Shares|*big.Int(bytes)|当前候选人总共质押加被委托的von数目|
|Released|*big.Int(bytes)|发起质押账户的自由金额的锁定期质押的von|
|ReleasedHes|*big.Int(bytes)|发起质押账户的自由金额的犹豫期质押的von|
|RestrictingPlan|*big.Int(bytes)|发起质押账户的锁仓金额的锁定期质押的von|
|RestrictingPlanHes|*big.Int(bytes)|发起质押账户的锁仓金额的犹豫期质押的von|
|ExternalId|string|外部Id(有长度限制，给第三方拉取节点描述的Id)|
|NodeName|string|被质押节点的名称(有长度限制，表示该节点的名称)|
|Website|string|节点的第三方主页(有长度限制，表示该节点的主页)|
|Details|string|节点的描述(有长度限制，表示该节点的描述)|

<a name="staking_contract_10"></a>
<code style="color: purple;">1103. getRelatedListByDelAddr() </code>: 查询当前账户地址所委托的节点的NodeID和质押Id

[回到接口目录](#top)

入参：

|名称|类型|说明|
|---|---|---|
|funcType|uint16(2bytes)|代表方法类型码(1103)|
|addr|common.address(20bytes)|委托人的账户地址|

返参： 列表

|名称|类型|说明|
|---|---|---|
|Addr|20bytes|验证人节点的地址|
|NodeId|64bytes|验证人的节点Id|
|StakingBlockNum|uint64(8bytes)|发起质押时的区块高度|


<a name="staking_contract_11"></a>
<code style="color: purple;">1104. getDelegateInfo() </code>: 查询当前单个委托信息

[回到接口目录](#top)

入参：

|名称|类型|说明|
|---|---|---|
|funcType|uint16|代表方法类型码(1104)|
|stakingBlockNum|uint64(8bytes)|发起质押时的区块高度|
|delAddr|20bytes|委托人账户地址|
|nodeId|64bytes|验证人的节点Id|

返参： 列表

|名称|类型|说明|
|---|---|---|
|Addr|20bytes|委托人的账户地址|
|NodeId|64bytes|验证人的节点Id|
|StakingBlockNum|uint64(8bytes)|发起质押时的区块高度|
|DelegateEpoch|uint32(4bytes)|最近一次对该候选人发起的委托时的结算周期|
|Released|*big.Int(bytes)|发起委托账户的自由金额的锁定期委托的von|
|ReleasedHes|*big.Int(bytes)|发起委托账户的自由金额的犹豫期委托的von|
|RestrictingPlan|*big.Int(bytes)|发起委托账户的锁仓金额的锁定期委托的von|
|RestrictingPlanHes|*big.Int(bytes)|发起委托账户的锁仓金额的犹豫期委托的von|
|Reduction|*big.Int(bytes)|处于撤销计划中的von|

<a name="staking_contract_12"></a>
<code style="color: purple;">1105. getCandidateInfo() </code>: 查询当前节点的质押信息

[回到接口目录](#top)

入参：

|名称|类型|说明|
|---|---|---|
|funcType|uint16|代表方法类型码(1105)|
|nodeId|64bytes|验证人的节点Id|

返参： 列表

|名称|类型|说明|
|---|---|---|
|NodeId|64bytes|被质押的节点Id(也叫候选人的节点Id)|
|StakingAddress|20bytes|发起质押时使用的账户(后续操作质押信息只能用这个账户，撤销质押时，von会被退回该账户或者该账户的锁仓信息中)|
|BenefitAddress|20bytes|用于接受出块奖励和质押奖励的收益账户|
|StakingTxIndex|uint32(4bytes)|发起质押时的交易索引|
|ProgramVersion|uint32(4bytes)|被质押节点的PlatON进程的真实版本号(获取版本号的接口由治理提供)|
|Status|uint32(4bytes)|候选人的状态(状态是根据uint32的32bit来放置的，可同时存在多个状态，值为多个同时存在的状态值相加0: 节点可用 (32个bit全为0)； 1: 节点不可用 (只有最后一bit为1)； 2： 节点出块率低(只有倒数第二bit为1)； 4： 节点的von不足最低质押门槛(只有倒数第三bit为1)； 8：节点被举报双签(只有倒数第四bit为1))|
|StakingEpoch|uint32(4bytes)|当前变更质押金额时的结算周期|
|StakingBlockNum|uint64(8bytes)|发起质押时的区块高度|
|Shares|*big.Int(bytes)|当前候选人总共质押加被委托的von数目|
|Released|*big.Int(bytes)|发起质押账户的自由金额的锁定期质押的von|
|ReleasedHes|*big.Int(bytes)|发起质押账户的自由金额的犹豫期质押的von|
|RestrictingPlan|*big.Int(bytes)|发起质押账户的锁仓金额的锁定期质押的von|
|RestrictingPlanHes|*big.Int(bytes)|发起质押账户的锁仓金额的犹豫期质押的von|
|ExternalId|string|外部Id(有长度限制，给第三方拉取节点描述的Id)|
|NodeName|string|被质押节点的名称(有长度限制，表示该节点的名称)|
|Website|string|节点的第三方主页(有长度限制，表示该节点的主页)|
|Details|string|节点的描述(有长度限制，表示该节点的描述)|



## 治理接口

**以下 治理相关接口的合约地址为:** 0x1000000000000000000000000000000000000005

<a name="submitText"></a>
<code style="color: purple;">2000. submitText() </code>: 提交文本提案

|参数|类型|说明|
|---|---|---|
|funcType|uint16|代表方法类型码(2000)|
|verifier|64bytes|提交提案的验证人|
|githubID|string|提案在github上的id|
|topic|string|提案主题，长度不超过128|
|desc|string|提案描述，长度不超过512|
|url|string|提案URL，长度不超过512|
|endVotingBlock|uint64|提案投票截止块高（投票截止高度设置为当前共识周期后第N个共识周期的第230个块，即： N * 250 - 20，其中0 < N <= 4840（约为2周），且为整数）|

<a name="submitVersion"></a>
<code style="color: purple;">2001. submitVersion() </code>: 提交升级提案

[回到接口目录](#top)

|参数|类型|说明|
|---|---|---|
|funcType|uint16|代表方法类型码(2001)|
|verifier|64bytes|提交提案的验证人|
|githubID|string|提案在github上的id|
|topic|string|提案主题，长度不超过128|
|desc|string|提案描述，长度不超过512|
|url|string|提案URL，长度不超过512|
|newVersion|uint|升级版本|
|endVotingBlock|uint64|提案投票截止块高（投票截止高度设置为当前共识周期后第N个共识周期的第230个块，即： N * 250 - 20，其中0 < N <= 4840（约为2周），且为整数）|
|activeBlock|uint64|（如果投票通过）生效块高（endVotingBlock+ 4*250 < 生效块高 <= endVotingBlock+ 10*250）|

<a name="submitParam"></a>
<code style="color: purple;">2002. submitParam() </code>: 提交参数提案

[回到接口目录](#top)

|参数|类型|说明|
|---|---|---|
|funcType|uint16|代表方法类型码(2002)|
|verifier|64bytes|提交提案的验证人|
|githubID|string|提案在github上的id|
|topic|string|提案主题，长度不超过128|
|desc|string|提案描述，长度不超过512|
|url|string|提案URL，长度不超过512|
|endVotingBlock|uint64|提案投票截止块高（投票截止高度设置为当前共识周期后第N个共识周期的第230个块，即： N * 250 - 20，其中0 < N <= 4840（约为2周），且为整数）|
|paramName|string|参数名称|
|currentValue|string|当前值|
|newValue|string|新的值|


<a name="vote"></a>
<code style="color: purple;">2003. vote() </code>: 给提案投票

[回到接口目录](#top)

|参数|类型|说明|
|---|---|---|
|funcType|uint16|代表方法类型码(2003)|
|verifier|64bytes|投票验证人|
|proposalID|common.Hash(32bytes)|提案ID|
|option|VoteOption|投票选项|
|programVersion|uint32|节点代码版本，有rpc接口获取|

<a name="declareVersion"></a>
<code style="color: purple;">2004. declareVersion() </code>: 版本声明

[回到接口目录](#top)

|参数|类型|说明|
|---|---|---|
|funcType|uint16|代表方法类型码(2004)|
|activeNode|64bytes|声明的节点，只能是验证人/候选人|
|version|uint|声明的版本|

<a name="getProposal"></a>
<code style="color: purple;">2100. getProposal() </code>: 查询提案

[回到接口目录](#top)

入参：

|名称|类型|说明|
|---|---|---|
|funcType|uint16|2100)|
|proposalID|common.Hash(32bytes)|提案ID|

返参：

Proposal接口实现对象的json字符串，参考： [Proposal接口 提案定义](#Proposal)

<a name="getTallyResult"></a>
<code style="color: purple;">2101. getTallyResult() </code>: 查询提案结果

[回到接口目录](#top)

入参：

|名称|类型|说明|
|---|---|---|
|funcType|uint16|代表方法类型码(2101)|
|proposalID|common.Hash(32bytes)|提案ID|

返参：
TallyResult对象的json字符串，参考：[TallyResult 投票结果定义](#TallyResult)

<a name="listProposal"></a>

<code style="color: purple;">2102. listProposal() </code>: 查询提案列表

[回到接口目录](#top)

入参：

|名称|类型|说明|
|---|---|---|
|funcType|uint16(2bytes)|代表方法类型码(2102)|

返参：

Proposal接口实现对象列表的json字符串，参考： [Proposal接口 提案定义](#Proposal)



<a name="getActiveVersion"></a>

<code style="color: purple;">2103. getActiveVersion() </code>: 查询节点的链生效版本

[回到接口目录](#top)

入参：

|名称|类型|说明|
|---|---|---|
|funcType|uint16(2bytes)|代表方法类型码(2103)|

返参：

版本号的json字符串，如{65536}，表示版本是：1.0.0。
解析时，需要把ver转成4个字节。主版本：第二个字节；小版本：第三个字节，patch版本，第四个字节。


<a name="getProgramVersion"></a>

<code style="color: purple;">2104. getProgramVersion() </code>: 查询节点代码版本

[回到接口目录](#top)

入参：

|名称|类型|说明|
|---|---|---|
|funcType|uint16(2bytes)|代表方法类型码(2104)|

返参：

版本号的json字符串，如{65536}，表示版本是：1.0.0。
解析时，需要把ver转成4个字节。主版本：第二个字节；小版本：第三个字节，patch版本，第四个字节。


<a name="listParam"></a>

<code style="color: purple;">2105. listParam() </code>: 查询可治理参数列表

[回到接口目录](#top)

入参：

|名称|类型|说明|
|---|---|---|
|funcType|uint16(2bytes)|代表方法类型码(2105)|

返参：
map[string]string的json字符串

<a name="ProposalType"></a>

##  <span style="color: red;">**ProposalType 提案类型定义**</span> ：[回到模块目录](#top)

|类型|定义|说明|
|---|---|---|
|TextProposal|0x01|文本提案|
|VersionProposal|0x02|升级提案|
|ParamProposal|0x03|参数提案|

<a name="ProposalType"></a>

##  <span style="color: red;">**ProposalStatus 提案状态定义**</span> ：[回到模块目录](#top)

对文本提案来说，有：0x01,0x02,0x03三种状态；
对升级提案来说，有：0x01,0x03,0x04,0x05四种状态。
对参数提案来说，有：0x01,0x02,0x03三种状态；

|类型|定义|说明|
|---|---|---|
|Voting|0x01|文本提案|
|Pass|0x02|投票通过（文本提案）|
|Failed|0x03|投票失败|
|PreActive|0x04|升级版本预生效（升级提案投票通过）|
|Active|0x05|升级版本生效|

<a name="VoteOption"></a>

##  <span style="color: red;">**VoteOption 投票选项定义**</span> ：[回到模块目录](#top)

|类型|定义|说明|
|---|---|---|
|Yeas|0x01|支持|
|Nays|0x02|反对|
|Abstentions|0x03|弃权|

<a name="Proposal"></a>

##  <span style="color: red;">**Proposal接口 提案定义**</span> ：[回到模块目录](#top)

子类TextProposal：文本提案

- 字段说明：

|字段|类型|说明|
|---|---|---|
|ProposalID|common.Hash(32bytes)|提案ID|
|Proposer|common.NodeID(64bytes)|提案节点ID|
|ProposalType|byte|提案类型， 0x01：文本提案； 0x02：升级提案；0x03参数提案。|
|GithubID|string|githubID|
|Topic|string|提案标题|
|Desc|string|提案描述|
|SubmitBlock|8bytes|提交提案的块高|
|EndVotingBlock|8bytes|提案投票结束的块高|
|ActiveBlock|8bytes|提案生效块高，proposalType:0x02才有|
|NewVersion|uint|升级版本，proposalType:0x02才有|


子类VersionProposal：升级提案

- 字段说明：

|字段|类型|说明|
|---|---|---|
|ProposalID|common.Hash(32bytes)|提案ID|
|Proposer|common.NodeID(64bytes)|提案节点ID|
|ProposalType|byte|提案类型， 0x01：文本提案； 0x02：升级提案；0x03参数提案。|
|GithubID|string|githubID|
|Topic|string|提案标题|
|Desc|string|提案描述|
|SubmitBlock|8bytes|提交提案的块高|
|EndVotingBlock|8bytes|提案投票结束的块高|
|ActiveBlock|8bytes|提案生效块高，proposalType:0x02才有|
|NewVersion|uint|升级版本，proposalType:0x02才有|



子类ParamProposal：参数提案

- 字段说明：

|字段|类型|说明|
|---|---|---|
|ProposalID|common.Hash(32bytes)|提案ID|
|Proposer|common.NodeID(64bytes)|提案节点ID|
|ProposalType|byte|提案类型， 0x01：文本提案； 0x02：升级提案；0x03参数提案。|
|GithubID|string|githubID|
|Topic|string|提案标题|
|Desc|string|提案描述|
|SubmitBlock|8bytes|提交提案的块高|
|EndVotingBlock|8bytes|提案投票结束的块高|
|ParamName|string|参数名称，proposalType:0x03才有|
|CurrentValue|string|参数当前值，proposalType:0x03才有|
|NewValue|string|参数新值，proposalType:0x03才有|


<a name="Vote"></a>

##  <span style="color: red;">**Vote 投票定义**</span> ：[回到模块目录](#top)

|字段|类型|说明|
|---|---|---|
|voter|64bytes|投票验证人|
|proposalID|common.Hash(32bytes)|提案ID|
|option|VoteOption|投票选项|

<a name="TallyResult"></a>

##  <span style="color: red;">**TallyResult 投票结果定义**</span> ：[回到模块目录](#top)

|字段|类型|说明|
|---|---|---|
|proposalID|common.Hash(32bytes)|提案ID|
|yeas|uint16(2bytes)|赞成票|
|nays|uint16(2bytes)|反对票|
|abstentions|uint16(2bytes)|弃权票|
|accuVerifiers|uint16(2bytes)|在整个投票期内有投票资格的验证人总数|
|status|byte|状态|


## 举报惩罚接口

**以下 slashing相关接口的合约地址为:** 0x1000000000000000000000000000000000000004

<a name="ReportDuplicateSign"></a>
<code style="color: purple;">3000. ReportDuplicateSign() </code>: 举报双签

[回到接口目录](#top)

| 参数     | 类型   | 描述                                    |
| -------- | ------ | --------------------------------------- |
| funcType | uint16(2bytes) | 代表方法类型码(3000)                    |
| data     | string | 证据的json值，格式为RPC接口Evidences的返回值 |

<a name="CheckDuplicateSign"></a>
<code style="color: purple;">3001. CheckDuplicateSign() </code>: 查询节点是否已被举报过多签

入参：

| 参数        | 类型           | 描述                                                         |
| ----------- | -------------- | ------------------------------------------------------------ |
| funcType    | uint16(2bytes) | 代表方法类型码(3001)                                         |
| typ         | uint32         | 代表双签类型，<br />1：prepare，2：viewChange，3：TimestampViewChange |
| addr        | 20bytes        | 举报的节点地址                                               |
| blockNumber | uint64         | 多签的块高                                                   |

回参：

| 类型   | 描述           |
| ------ | -------------- |
| 16进制 | 举报的交易Hash |



## 锁仓接口

**以下 锁仓相关接口的合约地址为:** 0x1000000000000000000000000000000000000001

<a name="CreateRestrictingPlan"></a>
<code style="color: purple;">4000. CreateRestrictingPlan() </code>: 创建锁仓计划

[回到接口目录](#top)

入参：

| 参数    | 类型           | 说明                                                         |
| ------- | -------------- | ------------------------------------------------------------ |
| account | 20bytes | `锁仓释放到账账户`                                           |
| plan    | []RestrictingPlan | plan 为 RestrictingPlan 类型的列表（数组），RestrictingPlan 定义如下：<br>type RestrictingPlan struct { <br/>    Epoch uint64<br/>    Amount：\*big.Int<br/>}<br/>其中，Epoch：表示结算周期的倍数。与每个结算周期出块数的乘积表示在目标区块高度上释放锁定的资金。Epoch \* 每周期的区块数至少要大于最高不可逆区块高度。<br>Amount：表示目标区块上待释放的金额。 |

<a name="GetRestrictingInfo"></a>
<code style="color: purple;">4100. GetRestrictingInfo() </code>: 获取锁仓信息。

注：本接口支持获取历史数据，请求时可附带块高，默认情况下查询最新块的数据。

[回到接口目录](#top)

入参：

| 参数    | 类型    | 说明               |
| ------- | ------- | ------------------ |
| account | 20bytes | `锁仓释放到账账户` |

返参：

返回参数为下面字段的 json 格式字符串

| 名称    | 类型            | 说明                                                         |
| ------- | --------------- | ------------------------------------------------------------ |
| balance | *big.Int(bytes) | 锁仓余额                                                     |
| debt    | *big.Int(bytes) | symbol为 true，debt 表示欠释放的款项，为 false，debt 表示可抵扣释放的金额 |
| symbol  | bool            | debt 的符号                                                  |
| info    | bytes           | 锁仓分录信息，json数组：[{"blockNumber":"","amount":""},...,{"blockNumber":"","amount":""}]。其中：<br/>blockNumber：\*big.Int，释放区块高度<br/>amount：\*big.Int，释放金额 |
