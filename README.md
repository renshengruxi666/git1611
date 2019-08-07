# 目录

- [概述](#概述)
- [引用](#引用)
  - [安装或引入](#安装或引入)
  - [初始化代码](#初始化代码)
- [rpc接口](#rpc接口)
- [wasm智能合约](#wasm智能合约)
  - [合约示例](#合约示例)
  - [部署合约](#部署合约)
  - [合约call调用](#合约call调用)
  - [合约sendRawTransaction调用](#合约sendrawtransaction调用)
  - [编码解码](#编码解码)
      - [getPlatONData](#contract.getPlatONData)
      - [decodePlatONCall](#contract.decodePlatONCall)
      - [decodePlatONLog](#contract.decodePlatONLog)
- [内置合约](#内置合约)
  - [整体架构图](#整体架构图)
  - [合约初始化及接口调用](#合约初始化及接口调用)
  - [合约API](#合约API)
    - [StakingContract](#StakingContract)
      - [staking](#staking-发起质押)
      - [updateStakingInfo](#updateStakingInfo-修改质押信息)
      - [unStaking](#unStaking-撤销质押)
      - [addStaking](#addStaking-增持质押)
      - [GetStakingInfo](#GetStakingInfo-获取质押信息)
    - [NodeContract](#NodeContract)
      - [GetVerifierList](#GetVerifierList-查询当前结算周期的验证人队列)
      - [getValidatorList](#getValidatorList-查询当前共识周期的验证人列表)
      - [getCandidateList](#getCandidateList-查询所有实时的候选人列表)
    - [DelegateContract](#DelegateContract)
      - [delegate](#delegate-发起委托)
      - [withdrewDelegate](#withdrewDelegate-撤销委托)
      - [GetRelatedListByDelAddr](#GetRelatedListByDelAddr-查询当前账户地址所委托的节点的NodeID和质押Id)
      - [GetDelegateInfo](#GetDelegateInfo-查询当前单个委托信息)
    - [ProposalContract](#ProposalContract)
      - [submitText](#submitText-提交文本提案)
      - [submitParam](#submitParam-提交参数提案)
      - [submitVersion](#submitVersion-提交升级提案)
      - [vote](#vote-给提案投票)
      - [declareVersion](#declareVersion-版本声明)
      - [getProposal](#getProposal-查询提案)
      - [listProposal](#listProposal-查询提案列表)
      - [getTallyResult](#getTallyResult-查询提案结果)
      - [getActiveVersion](#getActiveVersion-查询节点的链生效版本)
      - [getProgramVersion](#getProgramVersion-查询节点代码版本)
    - [RestrictingPlanContract](#RestrictingPlanContract)
      - [createRestrictingPlan](#createRestrictingPlan-创建锁仓计划)
      - [getRestrictingInfo](#getRestrictingInfo-获取锁仓信息)
    - [SlashContract](#SlashContract)
      - [reportDoubleSign](#reportDoubleSign-举报多签)
      - [checkDuplicateSign](#checkDuplicateSign-查询接口是否已经被举报多签)

## 概述
> Javascript SDK是PlatON面向js开发者，提供的PlatON公链的js开发工具包

## 引用

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
## rpc接口

<!-- ### web3.version.api

返参： String - The ethereum js api version

示例
```
var version = web3.version.api;
console.log(version); // "0.2.0"
```
### web3.version.node
```
web3.version.node
// or async
web3.version.getNode(callback(error, result){ ... })
```

返参： String - The client/node version.

示例
```
var version = web3.version.node;
console.log(version); // "Mist/v0.9.3/darwin/go1.4.1"

``` -->

## wasm智能合约
wasm智能合约的编写及其ABI(json文件)和BIN(wasm文件)生成方法请参考 [WASM合约指南](/zh-cn/development/%5BChinese-Simplified%5D-Wasm%E5%90%88%E7%BA%A6%E5%BC%80%E5%8F%91%E6%8C%87%E5%8D%97.md)。

#### 合约示例

```
    namespace platon {
        class ACC : public token::Token {
        public:
            ACC(){}
            void init(){
                Address user(std::string("0xa0b21d5bcc6af4dda0579174941160b9eecb6916"), true);
                initSupply(user, 199999);
            }

            void create(const char *addr, uint64_t asset){
                Address user(std::string(addr), true);
                Token::create(user, asset);
            }

            uint64_t getAsset(const char *addr) const {
                Address user(std::string(addr), true);
                return Token::getAsset(user);
            }

            void transfer(const char *addr, uint64_t asset) {
                Address user(std::string(addr), true);
                Token::transfer(user, asset);
            }
        };
    }

    PLATON_ABI(platon::ACC, create)
    PLATON_ABI(platon::ACC, getAsset)
    PLATON_ABI(platon::ACC, transfer)
    //platon autogen begin
    extern "C" {
        void create(const char * addr,unsigned long long asset) {
            platon::ACC ACC_platon;
            ACC_platon.create(addr,asset);
        }
        unsigned long long getAsset(const char * addr) {
            platon::ACC ACC_platon;
            return ACC_platon.getAsset(addr);
        }
        void transfer(const char * addr,unsigned long long asset) {
            platon::ACC ACC_platon;
            ACC_platon.transfer(addr,asset);
        }
        void init() {
            platon::ACC ACC_platon;
            ACC_platon.init();
        }
    }
    //platon autogen end
```

#### 部署合约

##### 接口声明

    MyContract.deploy(deployData [, callback])

##### 参数说明

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|deployData |String |必选| 16进制格式的签名交易数据|
|callback|Funciton  |可选|回调函数，用于支持异步的方式执行|

##### 示例

````js
const Web3 = require('web3'),
    fs = require('fs'),
    path = require('path'),
    Tx = require('ethereumjs-tx'),
    abi = require('./multisig.cpp.abi.json') //应用程序二进制接口
    ;

const provider = 'http://localhost:6789',
    privateKey='fd098d39e56115762cf28d7d223a4303a42a42451da6e7da37cb40a81a246d98',//私钥
    address='0xf216d6e4c17097a60ee2b8e5c88941cd9f07263b'

const web3 = new Web3(new Web3.providers.HttpProvider(provider))

const MyContract  = web3.platon.contract(abi)

const source = fs.readFileSync(path.join(__dirname, './demo.wasm'));//WebAssembly的二进制代码 bin

const platONData = MyContract.deploy.getPlatONData(source)

const params = {
    nonce: web3.platon.getTransactionCount(address),
    gas: "0x506709",
    gasPrice: "0x8250de00",
    data: platONData,
    from:address
}

const daployData=sign(privateKey,params)

let myContractReturned = MyContract.deploy(daployData, (err, myContract)=> {
    if (!err) {
        if (!myContract.address) {
            console.log("contract deploy transaction hash: " + myContract.transactionHash) // 部署合约的交易哈希值
        } else {
            // 合约发布成功
            console.log("contract deploy transaction address: " + myContract.address)// 部署合约的地址
            const receipt = web3.platon.getTransactionReceipt(myContract.transactionHash);
            console.log(`contract deploy receipt:`, receipt);
        }
    } else {
        console.log(`contract deploy error:`, err)
    }
});

function sign(privateKey, data) {
    const key = new Buffer(privateKey, 'hex')
    const tx = new Tx(data)
    tx.sign(key)

    const serializeTx = tx.serialize()
    const result = '0x' + serializeTx.toString('hex')
    return result
}
````
#### 合约call调用

> 调用合约函数，返回常量值

##### 接口声明

    web3.platon.call(Object [, defaultBlock] [, callback])

##### 参数说明

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|Object |String |必选| 返回一个交易对象，同`web3.platon.sendTransaction`。与`sendTransaction`的区别在于，`from`属性是可选的|
|defaultBlock|Number/String |可选|如果不设置此值使用`web3.platon.defaultBlock`设定的块，否则使用指定的块|
|callback|Funciton  |可选|回调函数，用于支持异步的方式执行|

##### 示例

````js
const data = contract.getOwners.getPlatONData()

const result = web3.platon.call({
    from: web3.platon.accounts[0],
    to: contract.address,
    data: data
});

console.log('call result:', result);

contract.decodePlatONCall(result)
````

#### 合约sendRawTransaction调用

> 发送通过私钥签名的交易

##### 接口声明

    web3.platon.sendRawTransaction(signedTransactionData [, callback])

##### 参数说明

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|signedTransactionData |String |必选|16进制格式的签名交易数据|
|callback|Funciton  |可选|回调函数，用于支持异步的方式执行|

##### 示例

````js
const Tx = require('ethereumjs-tx');

const privateKey='4484092b68df58d639f11d59738983e2b8b81824f3c0c759edd6773f9adadfe7',
    param_from = web3.platon.accounts[0],
    param_to = wallet.address,
    param_assert = 4
    ;

const data = myContractInstance.transfer02.getPlatONData(param_from, param_to, param_assert);

const hash = web3.platon.sendRawTransaction(sign(privateKey,getParams(data)))
console.log('hash', hash);//0xb1335d4db521ddc0b390448f919e5b5af1258b29e7ab4e0d68b0ef315af0cf5f

const data=web3.platon.getTransactionReceipt(hash);

let res = myContractInstance.decodePlatONLog(data.logs[0]);

function sign(privateKey, data) {
    const key = new Buffer(privateKey, 'hex')
    const tx = new Tx(data)
    tx.sign(key)

    const serializeTx = tx.serialize()
    const result = '0x' + serializeTx.toString('hex')
    return result
}

function getParams(data = '', value = "0x0") {
    const nonce = web3.platon.getTransactionCount(wallet.address)
    value = web3.toHex(web3.toVon(value, 'lat'));

    const params = {
        from: '0xf216d6e4c17097a60ee2b8e5c88941cd9f07263b',
        gasPrice: 22 * 10e9,
        gas: 80000,
        to: myContractInstance.address,
        value,
        data,
        nonce
    }

    return params
}
````
### 编码解码

<a name="contract.getPlatONData"></a>

#### contract.getPlatONData

##### 接口声明

    MyContract.myMethod.getPlatONData(param1 [, param2, ...])

##### 参数

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|param1/param2/...|String/Number|必选|零或多个函数参数|

**返回值 或 回调**

`String` - 十六进制字符串

##### 示例

MyContract.myMethod.getPlatONData([param1,param2,param3]);

<a name="contract.decodePlatONCall"></a>

#### contract.decodePlatONCall

> 根据abi中fnName下的outputs类型解析call返回结果

##### 接口声明

    MyContract.decodePlatONCall( callResult, [fnName])

##### 参数

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|callResult|String|必选|应用程序二进制接口对象|
|fnName|String|可选|不传，callResult将string解析callResult,如果传了，会找到abi中name等于fnName下的outputs，按照outputs.type解析|

**返回值 或 回调**

`code` - 0表示成功
`data` - 解析出来的数据

##### 示例

````js
    MyContract.decodePlatONCall( '0x',)
````
<a name="contract.decodePlatONLog"></a>

#### contract.decodePlatONLog

> 解析交易回执中logs的某一项

##### 接口声明

    MyContract.decodePlatONLog(log)

##### 参数

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|log||Object|交易回执中logs的某一项|

**返回值 或 回调**

`Array` - 解析后的日志数组

##### 示例

````js
const data=web3.platon.getTransactionReceipt('0xb1335d4db521ddc0b390448f919e5b5af1258b29e7ab4e0d68b0ef315af0cf5f');

let res = myContractInstance.decodePlatONLog(data.logs[0]);
````
## 内置合约
> PlatON对外提供的与链交互的API

### 整体架构图

<!-- ![Image text](https://raw.githubusercontent.com/hongmaju/light7Local/master/img/productShow/20170518152848.png) -->

> 此架构图是对内置合约API的一个汇总，方便用户更直观的弄清API的使用，及各API之间的关系。

### 合约初始化及接口调用

```js
const cid = web3.version.network;
web3.[ContractName].init(web3,cid);

web3.[ContractName].[funcName]();
```
#### 示例

```js
const cid = web3.version.network;
web3.StakingContract.init(web3,cid);

web3.stakingContract.staking({
  from:'0x493301712671Ada506ba6Ca7891F436D29185821',
  privateKey:'a11859ce23effc663a9460e332ca09bd812acc390497f8dc7542b6938e13f8d7',
  value:0,
  typ:1,
  benifitAddress:'0x12c171900f010b17e969702efa044d077e868082',
  nodeId:'f71e1bc638456363a66c4769284290ef3ccff03aba4a22fb60ffaed60b77f614bfd173532c3575abe254c366df6f4d6248b929cb9398aaac00cbcc959f7b2b7c',
  externalId:'111111',
  nodeName:'platon',
  website:'https://www.test.network',
  details:'supper node',
  amount:1000000000000000000000000,
  processVersion:1792,
}).then(res=>{
    console.log(res); // {result:true,data:'0x03aba4a22fb60ffaed60b77f614bfd173532c357'}
}).catch(err=>{
    console.log(err);
});
```
<a name="合约API"></a>

### 合约API

#### 接口说明

以下合约接口未列出返回参数的即为交易接口，其它为查询接口

交易接口统一返回参数形式如下：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|data |String |必选| 交易hash|

查询接口统一返回参数形式如下：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|result |Boolean |必选| true代表成功，false代表查询出错|
|data |String或json |必选| 返回的结果|
|err |String |必选| 查询出错的错误信息|

<a name="StakingContract"></a>

#### StakingContract

<a name="staking-发起质押"></a>

##### staking-发起质押

入参：

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

示例
```js
web3.stakingContract.staking({
  from:'0x493301712671Ada506ba6Ca7891F436D29185821',
  privateKey:'a11859ce23effc663a9460e332ca09bd812acc390497f8dc7542b6938e13f8d7',
  value:0,
  typ:1,
  benifitAddress:'0x12c171900f010b17e969702efa044d077e868082',
  nodeId:'f71e1bc638456363a66c4769284290ef3ccff03aba4a22fb60ffaed60b77f614bfd173532c3575abe254c366df6f4d6248b929cb9398aaac00cbcc959f7b2b7c',
  externalId:'111111',
  nodeName:'platon',
  website:'https://www.test.network',
  details:'supper node',
  amount:1000000000000000000000000,
  processVersion:1792,
}).then(res=>{
    console.log(res); // {data:'0x03aba4a22fb60ffaed60b77f614bfd173532c357'}
}).catch(err=>{
    console.log(err);
});
```

<a name="updateStakingInfo-修改质押信息"></a>

##### updateStakingInfo-修改质押信息

入参：

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

示例
```js
web3.stakingContract.updateStakingInfo({
  from:'0x493301712671Ada506ba6Ca7891F436D29185821',
  privateKey:'a11859ce23effc663a9460e332ca09bd812acc390497f8dc7542b6938e13f8d7',
  value:0,
  typ:1,
  benifitAddress:'0x12c171900f010b17e969702efa044d077e868082',
  nodeId:'f71e1bc638456363a66c4769284290ef3ccff03aba4a22fb60ffaed60b77f614bfd173532c3575abe254c366df6f4d6248b929cb9398aaac00cbcc959f7b2b7c',
  externalId:'111111',
  nodeName:'platon',
  website:'https://www.test.network',
  details:'supper node',
}).then(res=>{
    console.log(res); // {data:'0x03aba4a22fb60ffaed60b77f614bfd173532c358'}
}).catch(err=>{
    console.log(err);
});
```

<a name="unStaking-撤销质押"></a>

##### unStaking-撤销质押

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|nodeId|String  |必选|被质押的节点Id|

示例
```js
web3.stakingContract.unStaking({
  from:'0x493301712671Ada506ba6Ca7891F436D29185821',
  privateKey:'a11859ce23effc663a9460e332ca09bd812acc390497f8dc7542b6938e13f8d7',
  value:0,
  nodeId:'f71e1bc638456363a66c4769284290ef3ccff03aba4a22fb60ffaed60b77f614bfd173532c3575abe254c366df6f4d6248b929cb9398aaac00cbcc959f7b2b7c',
}).then(res=>{
    console.log(res); // {data:'0x03aba4a22fb60ffaed60b77f614bfd173532c359'}
}).catch(err=>{
    console.log(err);
});
```
<a name="addStaking-增持质押"></a>

##### addStaking-增持质押

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|nodeId|String  |必选|被质押的节点Id|
|typ|Number  |必选|表示使用账户自由金额还是账户的锁仓金额，0: 自由金额； 1: 锁仓金额|
|amount|Number  |必选|质押的von|

示例
```js
web3.stakingContract.addStaking({
  from:'0x493301712671Ada506ba6Ca7891F436D29185821',
  privateKey:'a11859ce23effc663a9460e332ca09bd812acc390497f8dc7542b6938e13f8d7',
  value:0,
  nodeId:'f71e1bc638456363a66c4769284290ef3ccff03aba4a22fb60ffaed60b77f614bfd173532c3575abe254c366df6f4d6248b929cb9398aaac00cbcc959f7b2b7c',
  typ:1,
  amount:1000000000000000000000000,
}).then(res=>{
    console.log(res); // {data:'0x03aba4a22fb60ffaed60b77f614bfd173532c360'}
}).catch(err=>{
    console.log(err);
});
```

<a name="GetStakingInfo-获取质押信息"></a>

##### GetStakingInfo-获取质押信息

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|nodeId|String  |必选|被质押的节点Id|

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

示例
```js
web3.stakingContract.GetStakingInfo({
  nodeId:'f71e1bc638456363a66c4769284290ef3ccff03aba4a22fb60ffaed60b77f614bfd173532c3575abe254c366df6f4d6248b929cb9398aaac00cbcc959f7b2b7c',
}).then(res=>{
    console.log(res); // {result:true,data:[...],err:''}
}).catch(err=>{
    console.log(err);
});
```
<a name="NodeContract"></a>

#### NodeContract

<a name="GetVerifierList-查询当前结算周期的验证人队列"></a>

##### GetVerifierList-查询当前结算周期的验证人队列

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
无

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

示例
```js
web3.NodeContract.GetVerifierList().then(res=>{
    console.log(res); // {result:true,data:[...],err:''}
}).catch(err=>{
    console.log(err);
});
```

<a name="getValidatorList-查询当前共识周期的验证人列表"></a>

##### getValidatorList-查询当前共识周期的验证人列表

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
无

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

示例
```js
web3.NodeContract.getValidatorList().then(res=>{
    console.log(res); // {result:true,data:[...],err:''}
}).catch(err=>{
    console.log(err);
});
```

<a name="getCandidateList-查询所有实时的候选人列表"></a>

##### getCandidateList-查询所有实时的候选人列表

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
无

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

示例
```js
web3.NodeContract.getCandidateList().then(res=>{
    console.log(res); // {result:true,data:[...],err:''}
}).catch(err=>{
    console.log(err);
});
```
<a name="DelegateContract"></a>

#### DelegateContract

<a name="delegate-发起委托"></a>

##### delegate-发起委托

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|typ|Number  |必选|表示使用账户自由金额还是账户的锁仓金额，0: 自由金额； 1: 锁仓金额|
|nodeId|String  |必选|委托的节点Id|
|amount|Number  |必选|委托的金额|

示例
```js
web3.DelegateContract.delegate({
  from:'0x493301712671Ada506ba6Ca7891F436D29185821',
  privateKey:'a11859ce23effc663a9460e332ca09bd812acc390497f8dc7542b6938e13f8d7',
  value:0,
  nodeId:'f71e1bc638456363a66c4769284290ef3ccff03aba4a22fb60ffaed60b77f614bfd173532c3575abe254c366df6f4d6248b929cb9398aaac00cbcc959f7b2b7c',
  typ:1,
  amount:1000000000000000000000000,
}).then(res=>{
    console.log(res); // {data:'0x03aba4a22fb60ffaed60b77f614bfd173532c360'}
}).catch(err=>{
    console.log(err);
});
```

<a name="withdrewDelegate-撤销委托"></a>

##### withdrewDelegate-撤销委托

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|stakingBlockNum|Number  |必选|质押唯一标识|
|nodeId|String  |必选|委托的节点Id|
|amount|Number  |必选|减持委托的金额|

示例
```js
web3.DelegateContract.withdrewDelegate({
  from:'0x493301712671Ada506ba6Ca7891F436D29185821',
  privateKey:'a11859ce23effc663a9460e332ca09bd812acc390497f8dc7542b6938e13f8d7',
  value:0,
  nodeId:'f71e1bc638456363a66c4769284290ef3ccff03aba4a22fb60ffaed60b77f614bfd173532c3575abe254c366df6f4d6248b929cb9398aaac00cbcc959f7b2b7c',
  stakingBlockNum:1000,
  amount:1000000000000000000000000,
}).then(res=>{
    console.log(res); // {data:'0x03aba4a22fb60ffaed60b77f614bfd173532c360'}
}).catch(err=>{
    console.log(err);
});
```

<a name="GetRelatedListByDelAddr-查询当前账户地址所委托的节点的NodeID和质押Id"></a>

##### GetRelatedListByDelAddr-查询当前账户地址所委托的节点的NodeID和质押Id

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|addr|String  |必选|委托人的账户地址|

返参： 列表

|名称|类型|说明|
|---|---|---|
|Addr|20bytes|验证人节点的地址|
|NodeId|64bytes|验证人的节点Id|
|StakingBlockNum|uint64(8bytes)|发起质押时的区块高度|

示例
```js
web3.DelegateContract.GetRelatedListByDelAddr({
    addr:'0x12c171900f010b17e969702efa044d077e868082',
}).then(res=>{
    console.log(res); // {result:true,data:[...],err:''}
}).catch(err=>{
    console.log(err);
});
```

<a name="GetDelegateInfo-查询当前单个委托信息"></a>

##### GetDelegateInfo-查询当前单个委托信息

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|stakingBlockNum|Number  |必选|发起质押时的区块高度|
|delAddr|String  |必选|委托人的账户地址|
|nodeId|String  |必选|验证人的节点Id|

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

示例
```js
web3.DelegateContract.GetDelegateInfo({
    stakingBlockNum:1000,
    delAddr:'0x12c171900f010b17e969702efa044d077e868082',
    nodeId:'a11859ce23effc663a9460e332ca09bd812acc390497f8dc7542b6938e13f8d7'
}).then(res=>{
    console.log(res); // {result:true,data:[...],err:''}
}).catch(err=>{
    console.log(err);
});
```
<a name="ProposalContract"></a>

#### ProposalContract

<a name="submitText-提交文本提案"></a>

##### submitText-提交文本提案

入参：

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

示例
```js
web3.ProposalContract.submitText({
  from:'0x493301712671Ada506ba6Ca7891F436D29185821',
  privateKey:'a11859ce23effc663a9460e332ca09bd812acc390497f8dc7542b6938e13f8d7',
  value:0,
  verifier:'f71e1bc638456363a66c4769284290ef3ccff03aba4a22fb60ffaed60b77f614bfd173532c3575abe254c366df6f4d6248b929cb9398aaac00cbcc959f7b2b7c',
  githubID:'GithubID',
  topic:'Topic',
  desc:'Desc',
  url:'http://www.test.inet',
  endVotingBlock:1000,
}).then(res=>{
    console.log(res); // {data:'0x03aba4a22fb60ffaed60b77f614bfd173532c360'}
}).catch(err=>{
    console.log(err);
});
```

<a name="submitParam-提交参数提案"></a>

##### submitParam-提交参数提案

入参：

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

示例
```js
web3.ProposalContract.submitParam({
  from:'0x493301712671Ada506ba6Ca7891F436D29185821',
  privateKey:'a11859ce23effc663a9460e332ca09bd812acc390497f8dc7542b6938e13f8d7',
  value:0,
  verifier:'f71e1bc638456363a66c4769284290ef3ccff03aba4a22fb60ffaed60b77f614bfd173532c3575abe254c366df6f4d6248b929cb9398aaac00cbcc959f7b2b7c',
  githubID:'GithubID',
  topic:'Topic',
  desc:'Desc',
  url:'http://www.test.inet',
  endVotingBlock:1000,
  paramName:'ParamName',
  currentValue:'0.85',
  newValue:'1.02',
}).then(res=>{
    console.log(res); // {data:'0x03aba4a22fb60ffaed60b77f614bfd173532c360'}
}).catch(err=>{
    console.log(err);
});
```

<a name="submitVersion-提交升级提案"></a>

##### submitVersion-提交升级提案

入参：

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

示例
```js
web3.ProposalContract.submitVersion({
  from:'0x493301712671Ada506ba6Ca7891F436D29185821',
  privateKey:'a11859ce23effc663a9460e332ca09bd812acc390497f8dc7542b6938e13f8d7',
  value:0,
  verifier:'f71e1bc638456363a66c4769284290ef3ccff03aba4a22fb60ffaed60b77f614bfd173532c3575abe254c366df6f4d6248b929cb9398aaac00cbcc959f7b2b7c',
  githubID:'GithubID',
  topic:'Topic',
  desc:'Desc',
  url:'http://www.test.inet',
  endVotingBlock:1000,
  newVersion:1,
  activeBlock:1000,
}).then(res=>{
    console.log(res); // {data:'0x03aba4a22fb60ffaed60b77f614bfd173532c360'}
}).catch(err=>{
    console.log(err);
});
```

<a name="vote-给提案投票"></a>

##### vote-给提案投票

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|verifier|String  |必选|提交提案的验证人|
|proposalID|String  |必选|提案ID|
|option|String  |必选|投票选项|
|programVersion|Number  |必选|节点代码版本|

示例
```js
web3.ProposalContract.vote({
  from:'0x493301712671Ada506ba6Ca7891F436D29185821',
  privateKey:'a11859ce23effc663a9460e332ca09bd812acc390497f8dc7542b6938e13f8d7',
  value:0,
  verifier:'f71e1bc638456363a66c4769284290ef3ccff03aba4a22fb60ffaed60b77f614bfd173532c3575abe254c366df6f4d6248b929cb9398aaac00cbcc959f7b2b7c',
  proposalID:'0x12c171900f010b17e969702efa044d077e86808212c171900f010b17e969702e',
  option:1,
  programVersion:1,
}).then(res=>{
    console.log(res); // {data:'0x03aba4a22fb60ffaed60b77f614bfd173532c360'}
}).catch(err=>{
    console.log(err);
});
```

<a name="declareVersion-版本声明"></a>

##### declareVersion-版本声明

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|activeNode|String  |必选|声明的节点，只能是验证人/候选人|
|version|String  |必选|声明的版本|

示例
```js
web3.ProposalContract.declareVersion({
  from:'0x493301712671Ada506ba6Ca7891F436D29185821',
  privateKey:'a11859ce23effc663a9460e332ca09bd812acc390497f8dc7542b6938e13f8d7',
  value:0,
  activeNode:'f71e1bc638456363a66c4769284290ef3ccff03aba4a22fb60ffaed60b77f614bfd173532c3575abe254c366df6f4d6248b929cb9398aaac00cbcc959f7b2b7c',
  version:1,
}).then(res=>{
    console.log(res); // {data:'0x03aba4a22fb60ffaed60b77f614bfd173532c360'}
}).catch(err=>{
    console.log(err);
});
```

<a name="getProposal-查询提案"></a>

##### getProposal-查询提案

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|proposalID |String |必选| 提案ID|

返参：Object

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|ProposalID|String  |提案ID|
|Proposer|String  |提案节点ID|
|ProposalType|String  |提案类型， 0x01：文本提案； 0x02：升级提案；0x03参数提案|
|SubmitBlock|String  |提交提案的块高|
|EndVotingBlock|String  |提案投票结束的块高|
|ActiveBlock|String  |提案生效块高，proposalType:0x02才有|
|NewVersion|String  |升级版本，proposalType:0x02才有|
|currentValue|String  |参数当前值，proposalType:0x03才有|
|newValue|String  |参数新值，proposalType:0x03才有|
|ParamName|String  |参数新值，proposalType:0x03才有|

示例
```js
web3.ProposalContract.getProposal({
    proposalID:'0x12c171900f010b17e969702efa044d077e86808212c171900f010b17e969702e'
}).then(res=>{
    console.log(res); // {result:true,data:{...},err:''}
}).catch(err=>{
    console.log(err);
});
```

<a name="listProposal-查询提案列表"></a>

##### listProposal-查询提案列表

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
无

返参：列表

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|ProposalID|String  |提案ID|
|Proposer|String  |提案节点ID|
|ProposalType|String  |提案类型， 0x01：文本提案； 0x02：升级提案；0x03参数提案|
|SubmitBlock|String  |提交提案的块高|
|EndVotingBlock|String  |提案投票结束的块高|
|ActiveBlock|String  |提案生效块高，proposalType:0x02才有|
|NewVersion|String  |升级版本，proposalType:0x02才有|
|currentValue|String  |参数当前值，proposalType:0x03才有|
|newValue|String  |参数新值，proposalType:0x03才有|
|ParamName|String  |参数新值，proposalType:0x03才有|

示例
```js
web3.ProposalContract.listProposal().then(res=>{
    console.log(res); // {result:true,data:[...],err:''}
}).catch(err=>{
    console.log(err);
});
```

<a name="getTallyResult-查询提案结果"></a>

##### getTallyResult-查询提案结果

入参：

| 参数名 |类型|属性|参数说明|
|proposalID |String |必选| 提案ID|

返参：Object

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|ProposalID|String|提案ID|
|yeas|Number|赞成票|
|nays|String|反对票|
|abstentions|String|弃权票|
|accuVerifiers|String|在整个投票期内有投票资格的验证人总数|
|status|String|状态|

示例
```js
web3.ProposalContract.getTallyResult({
    proposalID:'0x12c171900f010b17e969702efa044d077e86808212c171900f010b17e969702e'
}).then(res=>{
    console.log(res); // {result:true,data:{...},err:''}
}).catch(err=>{
    console.log(err);
});
```

<a name="getActiveVersion-查询节点的链生效版本"></a>

##### getActiveVersion-查询节点的链生效版本

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
无

返参： String

|名称|类型|说明|
|---|---|---|
|无|String|版本号|

示例
```js
web3.ProposalContract.getActiveVersion().then(res=>{
    console.log(res); // {result:true,data:'1792',err:''}
}).catch(err=>{
    console.log(err);
});
```

<a name="getProgramVersion-查询节点代码版本"></a>

##### getProgramVersion-查询节点代码版本

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
无

返参： String

|名称|类型|说明|
|---|---|---|
|无|String|版本号|

示例
```js
web3.ProposalContract.getProgramVersion().then(res=>{
    console.log(res); // {result:true,data:'1792',err:''}
}).catch(err=>{
    console.log(err);
});
```

<a name="listParam-查询可治理参数列表"></a>

##### listParam-查询可治理参数列表

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
无

返参： 列表

|名称|类型|说明|
|---|---|---|
|无|String|json字符串|

示例
```js
web3.ProposalContract.listParam().then(res=>{
    console.log(res); // {result:true,data:[...],err:''}
}).catch(err=>{
    console.log(err);
});
```
<a name="RestrictingPlanContract"></a>

#### RestrictingPlanContract

<a name="createRestrictingPlan-创建锁仓计划"></a>

##### createRestrictingPlan-创建锁仓计划

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|account|String  |必选|锁仓释放到账账户|
|Plan|Array  |必选|[{Epoch:Number,Amount:Number}],(Epoch：表示结算周期的倍数。Amount：表示目标区块上待释放的金额|

示例
```js
web3.RestrictingPlanContract.createRestrictingPlan({
  from:'0x493301712671Ada506ba6Ca7891F436D29185821',
  privateKey:'a11859ce23effc663a9460e332ca09bd812acc390497f8dc7542b6938e13f8d7',
  value:0,
  account:'0x493301712671Ada506ba6Ca7891F436D29185821',
  Plan:[{
            "Epoch":5,
            "Amount":1000000000000000000000000
        },{
            "Epoch":6,
            "Amount":2000000000000000000000000
        }],
}).then(res=>{
    console.log(res); // {data:'0x03aba4a22fb60ffaed60b77f614bfd173532c360'}
}).catch(err=>{
    console.log(err);
});
```

<a name="getRestrictingInfo-获取锁仓信息"></a>

##### getRestrictingInfo-获取锁仓信息

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|account|String  |必选|锁仓释放到账账户|

返参：Object

| 名称    | 类型            | 说明                                                         |
| ------- | --------------- | ------------------------------------------------------------ |
| balance | *big.Int(bytes) | 锁仓余额                                                     |
| debt    | *big.Int(bytes) | symbol为 true，debt 表示欠释放的款项，为 false，debt 表示可抵扣释放的金额 |
| symbol  | bool            | debt 的符号                                                  |
| info    | bytes           | 锁仓分录信息，json数组：[{"blockNumber":"","amount":""},...,{"blockNumber":"","amount":""}]。其中：<br/>blockNumber：\*big.Int，释放区块高度<br/>amount：\*big.Int，释放金额 |

示例
```js
web3.RestrictingPlanContract.getRestrictingInfo({
    account:'0x493301712671Ada506ba6Ca7891F436D29185821'
}).then(res=>{
    console.log(res); // {result:true,data:{...},err:''}
}).catch(err=>{
    console.log(err);
});
```
<a name="SlashContract"></a>

#### SlashContract

<a name="reportDoubleSign-举报多签"></a>

##### reportDoubleSign-举报多签

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|from |String |必选| 发送交易的账户地址|
|privateKey|String  |必选|发送交易的账户私钥|
|value|Number  |必选|发送交易的金额|
|data|String  |必选|证据的json值，格式为RPC接口Evidences的返回值|

示例
```js
web3.SlashContract.reportDoubleSign({
  from:'0x493301712671Ada506ba6Ca7891F436D29185821',
  privateKey:'a11859ce23effc663a9460e332ca09bd812acc390497f8dc7542b6938e13f8d7',
  value:0,
  data:''
}).then(res=>{
    console.log(res); // {data:'0x03aba4a22fb60ffaed60b77f614bfd173532c360'}
}).catch(err=>{
    console.log(err);
});
```

<a name="checkDuplicateSign-查询接口是否已经被举报多签"></a>

##### checkDuplicateSign-查询接口是否已经被举报多签

入参：

| 参数名 |类型|属性|参数说明|
| :------: |:------: |:------: | :------: |
|typ|Number  |必选|代表双签类型, 1：prepare，2：viewChange|
|addr|String  |必选|举报的节点地址|
|blockNumber|Number  |必选|多签的块高|

返参：String

示例
```js
web3.SlashContract.checkDuplicateSign({
    typ:1,
    addr:'0x493301712671Ada506ba6Ca7891F436D29185821',
    blockNumber:1000
}).then(res=>{
    console.log(res); // {result:true,data:'0x03aba4a22fb60ffaed60b77f614bfd173532c360',err:''}
}).catch(err=>{
    console.log(err);
});
```

