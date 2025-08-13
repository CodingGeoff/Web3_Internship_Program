---
timezone: UTC+8
---

# Wayland

**GitHub ID:** evelive3

**Telegram:** @oxwayland

## Self-introduction

Web2从业者；全栈偏后端开发，主力语言Python、Golang，学习Rust中；爱好矿晶、化石、游戏

## Notes

<!-- Content_START -->
# 2025-08-13

## 知识分享会

### 以太坊的PoS架构

- 执行层EL，通过JSON-RPC接口与用户通信
- 共识层CL，与Validator通信
- 验证者Validator

### 节点类型

- 轻节点
- 完整节点

### 快速启动一个以太坊测试节点

[参考](https://lxdao.notion.site/24bdceffe40b80148b07fa28483662cf)

#### 硬件要求

- 如果需要同步主网数据，需要准备2T SSD

#### 执行层EL

- [Geth](https://geth.ethereum.org/)，基于`Go`
- [Reth](https://reth.rs/)，基于`Rust`
- [Foundry Anvil](https://getfoundry.sh/anvil/overview/)，基于`Rust`
- [Hardhat Network](https://hardhat.org/hardhat-network/docs/overview)，基于`nodejs`

#### 共识层CL

- [Lighthouse](https://github.com/sigp/lighthouse)，基于`Rust`

#### 辅助工具

- [kurtosis](https://docs.kurtosis.com/)，基于`Go`编写，开发环境搭建工具，

### 启动节点可观测性

- Prometheus
- Grafana

### 课后练习

- [ ] 完成可观测环境的搭建

# 2025-08-12

## 知识分享

### 什么是Defi

#### 特点

- 去中心化
- 无许可
- 可验证
- 可组合性

#### 代表产品

- UniSwap：以太坊上最大DEX，通过自动化做市商（AMM）模型实现无需订单簿的代币交易
  - 业界前沿，一定要读它的白皮书
- SuShiSwap：最初是UniSwap V2的分叉
  - 吸血鬼攻击是一种非常好的项目启动方式
- MakerDAO
- DAI

#### 启示

- 可组合性：
  - DeFi乐高，协议间可无缝集成
  - 创新加速：直接调用现有协议
- 吸血鬼攻击
  - 风险点为不可持续，一定要发展出自己的特色，占领独特的生态位
- 代币标准化
  - 减少重复造轮子
- 治理与团队稳定性
- 长期主义VS短期投机
  - UniSwap：长期专注，最终脱颖而出
  - SuShiSwap：产品线混乱

### 链上金融

> 对比传统金融业，有许多优点
- 公开透明
- 24小时可用
- 市场自动调节
- 个人可以真正参与到社区决策共建中

## 相关网站

- [web3.career](https://web3.career)，可以在该网站查看招聘要求、业内薪资水平等

# 2025-08-11

## 以太坊中文周会

## 技术向：DApp架构概览

### 通过项目介绍DApp架构

- 智能合约
    - Solidity：智能合约开发语言
    - OpenZeppelin：安全、经过审计的智能合约库和开发工具集（基于Solidity）
    - Hardhat：EVM智能合约开发环境和任务运行器
    - Hardhat Ignition：Hardhat官方推出的模块化部署工具
- 区块链网络交互
    - wagmi：用于在React项目中和以太坊（EVM 兼容链）交互
    - viem：TypeScript原生的区块链交互工具库
- 前端
    - NextJS：React技术栈的再封闭
    - RainbowKit：React技术栈的钱包连接UI组件库
    - TailwindCSS：通用样式库

### 开发流程

1. 合约开发&部署
2. 前端集成&测试

### 思考&验证&下一步计划

- 为什么说合约是不可变的？这种不可变带来什么样的特性？
- 是否可以在前端与链上交互期间实现中间人攻击？
- 前端->区域链，后端在Web3可以扮演怎样的角色？
- 开发智能合约需要充分ERC和对OpenZeppelin，接下来需要收集和学习相关资料

## 运营向：Web3运营概览

### 与Web2的区别

- 远程工作较多
- 跨国协作机会较多
- 英语作为主要沟通语言
- 工作单位相对不固定，流动性较大

### 关注点

- 沟通能力
- 积极性
- 学习能力
- 主动改进

# 2025-08-10

## 阅读`Geth`源码

### RPC

#### 协议编解码接口

```go
type ServerCodec interface {
	peerInfo() PeerInfo
	readBatch() (msgs []*jsonrpcMessage, isBatch bool, err error)
	close()

	jsonWriter
}
```

#### 协议编解码实现

分别在`HTTP`和`Websocket`和`IPC`协议上实现了编解码接口

- [http](https://github.com/ethereum/go-ethereum/blob/master/rpc/http.go)
- [websocket](https://github.com/ethereum/go-ethereum/blob/master/rpc/websocket.go)
- [json](https://github.com/ethereum/go-ethereum/blob/master/rpc/json.go)

#### 核心调度结构

```go
type Server struct {
	services serviceRegistry
	idgen    func() ID

	mutex              sync.Mutex
	codecs             map[ServerCodec]struct{}
	run                atomic.Bool
	batchItemLimit     int
	batchResponseLimit int
	httpBodyLimit      int
}
```

#### 核心调度方法

- [RegisterName](https://github.com/ethereum/go-ethereum/blob/c3ef6c77c24956ebe1156205869259ffb8892486/rpc/server.go#L96C1-L98C2)实现了RPC注册
- [serveSingleRequest](https://github.com/ethereum/go-ethereum/blob/c3ef6c77c24956ebe1156205869259ffb8892486/rpc/server.go#L144)处理单个连接的请求生命周期

# 2025-08-09

## 阅读`Geth`源码

### 节点表管理

> 源码见[/p2p/discover/node.go](https://github.com/ethereum/go-ethereum/blob/c3ef6c77c24956ebe1156205869259ffb8892486/p2p/discover/node.go)

#### 核心结构体

- K桶结构体为`BucketNode`，使用了`json`标签，应该是需要序列化为JSON格式与外部交互
- 节点表结构体为`tableNode`，主要保存`enode.Node`的指针，有一些控制/记录类的属性成员
- `unwrapNodes`方法用于转换`[]*tableNode`为`[]*enode.Node`
- `nodesByDistance`结构体核心属性`entries []*enode.Node`，该列表保存了所有通信的节点信息，使用`Kademlia`算法XOR算出的逻辑距离决定保存顺序

#### 核心方法

- `nodesByDistance`结构体的核心方法`push`，用于维护节点表，如果节点表元素已经满员，则使用`copy(h.entries[ix+1:], h.entries[ix:])`将节点元素整体前移一位，再append新节点信息至表末尾

#### 其它方法

比较意外的是还有一段使用了Go泛型的代码

```go
func containsID[N nodeType](ns []N, id enode.ID) bool {
    for _, n := range ns {
        if n.ID() == id {
            return true
        }
    }
    return false
}
```

作用是搜索入参`id enode.ID`是否存在于泛节点类型`[]nodeType`(包含`T.ID()`方法的类型)中

# 2025-08-08

## 内容1：优秀笔记分享

> 群除我佬啊
> 群除我佬啊
> 群除我佬啊

- 前端和合约强无敌的`Segment7`
- 熟悉密码学的`阿哲`
    - 抗量子计算很重要，因为随着量子计算机技术的成熟，现有加密算法存在被攻破的风险

## 内容2：geth源码学习

### Node Discovery
    
节点发现上，采用了非常经典的[类Kademlia算法](https://github.com/ethereum/go-ethereum/blob/c3ef6c77c24956ebe1156205869259ffb8892486/p2p/discover/table.go)，对比[Kademlia](https://pdos.csail.mit.edu/~petar/papers/maymounkov-kademlia-lncs.pdf)的算法主要有以下不同

- ID长度不同，`Kademlia`使用 160 bit ID (SHA1)，`Geth`使用 256 bit ID (Keccak-256/SHA3)
- K桶数量不同，`Kademlia`有 160 个桶，`Geth`只有 17 个，并且只维护与自己距离`hashBits - nBuckets`外的节点
- K桶容量，与原算法一致，为16
- 仅实现了节点发现功能（合理裁剪）

```go
// Constant

// https://github.com/ethereum/go-ethereum/blob/c3ef6c77c24956ebe1156205869259ffb8892486/common/types.go#L39

const (
	// HashLength is the expected length of the hash
	HashLength = 32
	// AddressLength is the expected length of the address
	AddressLength = 20
)

// https://github.com/ethereum/go-ethereum/blob/c3ef6c77c24956ebe1156205869259ffb8892486/p2p/discover/table.go/#L48C33-L48C37
const (
	alpha           = 3  // Kademlia concurrency factor
	bucketSize      = 16 // Kademlia bucket size
	maxReplacements = 10 // Size of per-bucket replacement list

	// We keep buckets for the upper 1/15 of distances because
	// it's very unlikely we'll ever encounter a node that's closer.
	hashBits          = len(common.Hash{}) * 8
	nBuckets          = hashBits / 15       // Number of buckets
	bucketMinDistance = hashBits - nBuckets // Log distance of closest bucket

	// IP address limits.
	bucketIPLimit, bucketSubnet = 2, 24 // at most 2 addresses from the same /24
	tableIPLimit, tableSubnet   = 10, 24

	seedMinTableTime = 5 * time.Minute
	seedCount        = 30
	seedMaxAge       = 5 * 24 * time.Hour
)
```


### 执行层

> 消化中，有点费劲 [Execution Layer](https://epf.wiki/#/wiki/EL/el-specs)

  - 区块状态转换函数$\sigma_{t+1} \equiv \Pi(\sigma_t, B)$
  - 账户存储 Trie 根$\mathrm{TRIE}(L_I^{*}(\sigma[a]_s)) \equiv \sigma[a]_s$，用于将合约存储压缩成一个 32 字节的根，放进账户的 `storageRoot` 字段

# 2025-08-07

## 内容1：知识分享会

嘉宾: 饶炜彤@曼昆事务所

### 监管红线

- 虚拟货币的发行和募资
- 交易所玩杠杆（开设赌场）
- 开设交易所（非法集资）

### 风险要点：资金汇集

- 是否涉及法币（非常少项目持有完善的经济经营牌照）
- 交易所与虚拟货币基金
- 海外的合规性是否完备（AML与KYC）
- 技术上是否排除中国大陆用户

#### 传销与赌场

- 扩张过程的合规：回避多级返佣（传销特征）和团队奖励（变相多级返佣）（跟保险公司发展销售员的套路好像）
- 非典型安全：合约交易（可能被某些地方公安以此为理由抓捕）
- NFT的合规玩法设计（如果玩法设计有随机性，且投入现金有机率超过获得现金，就会被定性为开设赌场）

** 求职时注意观察自己在雇主是否满足以上特征 **

#### 洗钱与外汇管制

- 是洗钱链条的一环
- 是外汇兑换的媒介（被用于突破国家外汇管制）

> 出现不清楚的情况时，提高自身警惕

**难点: 资金的来龙去脉对于非管理层来说难以知晓**

#### 判断是否是洗钱

- 以明显高于市场的价格购买NFT（可能会查不到当事人，但会为了图方便直接把平台端了）

#### 判断是否突破外汇管制

- 出入的结算法币不同？比如美元进，人民币出

#### 判断是否违规的依据

- 项目是否有明确的负责组织，是否在明确、受监管的司法辖区内注册
- 是否为了取得合规资质，经过专业机构的第三方审计或安全测试
- 是否具备KYC、AML等反洗钱与用户身份识别制度
- 是否对外公开项目负责人、团队背景、资金来源路径等基本信息
- 是否具备高风险模块（比如混币器）
- 是否存在高风险的扩张方式（多级返佣）
- 收外汇要小心，需要汇款方是合规的

### 责任越大，风险越大

- 核心管理层
- 宣传负责人或社区负责人（甚至风险高于核心管理层，因为直接接触用户）
- 核心链条
- 纯粹技术

### 法律盲区：知情推断

- 即使是纯粹的技术提供者，也必须保持审慎判断
- 明知可疑而不询问，知情推断，**只有完全不知情才能免责**
- 免责的前提是尽力防范

**币安在最难取得资质的日本都持牌了，考虑尽快从OKX迁移到币安**

### Q&A

外贸使用USDT收款，在做KYC的前提下，只有税务风险
使用USDT结算工资时，因为国内还不支持USDT，所以存在税务风险
但由于如果对USDT收入征税的话，等于变相承认数字货币的合法性，与中央精神冲突，所以他们一般选择不管
USDT收入换回法币：小额C2C 大额去香港换 但磨损很大

### 知识科普

KYC（Know Your Customer）
- 目的：确认客户身份、评估风险等级、防止匿名或虚假账户。
- 关键动作：
- 身份核验：护照、驾照、人脸识别、活体检测等。
- 地址验证：近期水电账单、银行对账单等。
- 资金来源与用途：了解客户资金从哪来、用来干什么。
- 持续监控：账户行为发生异常时重新评估风险

AML（Anti-Money Laundering）
- 目的：发现并阻止洗钱、恐怖融资等金融犯罪。
- 关键动作：
- 客户风险评级（CDD/EDD）。
- 交易监控：对大额、频繁、异常交易实时预警。
- 制裁名单筛查：与OFAC、EU、UN 等制裁名单实时比对。
- 可疑交易报告（SAR）：一旦触发阈值，必须向监管机构报告。
- 员工培训与内部审计：确保制度长期有效。

## 内容2. 铸造自己的第一个NFT

- 参考[My first NFT](https://nft.myfirst.io/)，完成了NFT铸造
- 使用etherscan查看了智能合约源码，确定该NFT使用的`ERC-721`标准

# 2025-08-06

## 内容1：知识分享会

- [Marcus@X](https://x.com/Bitzack_01)

### V与BM

- Satoshi Nakamoto[^1]
- Vitalik buterin(V神)
- Dan Larimer(ByteMaster)

#### BM与BTS

- 特点
  - 第一个Dex
  - DPOS[^2]治理
- 失败原因
  - 没有智能合约
  - 订单簿？难以对抗传统交易所
  - 稳定币是区块链的刚需

#### BM与EOS

- 特点
    - 史上最大规模[^3]
    - DPOS，21个超级节点，中心化程度过高
- 失败原因
    - 未兑现技术承诺
    - DApp生态发展迟缓
    - 治理机制设计失败
    - 竞争压力大

### 课后题

> 去中心化和高性能，应该优先选择谁？
- 个人认为应该优先考虑去中心化，参考EOS的失败经验，如果纯考虑高性能，那区块链永远也干不过Web2
- 去中心化程度足够高的前提下，也应该重试优化性能

> 公链的不同治理模型有哪些？对比异同
- 因为之前只关注技术，不知道治理模型，AI了一下奇怪的知识又增加了🎉

| 分类                            | 描述                          | 示例                    |
| ----------------------------- | --------------------------- | --------------------- |
| 🔹 链下治理（Off-chain Governance） | 治理活动在链下进行，由开发者、基金会、社区讨论达成共识 | Bitcoin、Ethereum      |
| 🔸 链上治理（On-chain Governance）  | 所有提案与投票通过智能合约在链上执行，自动生效     | Tezos、Polkadot、Decred |
| 🔹 委托治理（Delegated Governance） | 社区将治理权委托给代表，由代表负责表决与提案      | EOS（DPoS）、Cosmos（部分）  |
| 🔸 混合治理（Hybrid Governance）    | 链上+链下结合，既有链上投票，又保留链下协调机制    | Ethereum 2.0、Polkadot |

> 应该优先激活哪些生态用户
- 先激活开发者，实现生态的建设，就像玩RTS游戏先造农民
- 再激活中间层，即基础设施提供者
- 然后激活内容创作者
- 最后是终端用户

> 公链如何管理资金来获取用户信任
- 传统方式通过成立基金会进行管理，类似Blender Foundation, Apache Foundation等
- Web3特色方式为链上国库 + DAO治理
- 不论是传统还是链上，都需要透明、公开，让用户多参与

## 内容2：尝试自行编写智能合约铸造NFT

### 使用工具&网站

- [foundry](https://github.com/foundry-rs/foundry) is a blazing fast, portable and modular toolkit for Ethereum application development written in Rust. #Rust #Tools #Opensource #Web3
- [bun](https://github.com/oven-sh/bun) is an ^^all-in-one^^ toolkit for JavaScript and ^^TypeScript^^ apps. It ships as ^^a single executable^^ called `bun`. #Zig #Opensource
- [OpenSea](https://opensea.io/)
- [etherscan](https://optimistic.etherscan.io)
- [reth](https://reth.rs/)
- [Pinata](https://pinata.cloud/)，使用IPFS技术的图床
- [MetaMask](https://metamask.io/)，小狐狸钱包
- WSL，由于在Windows下

### 安装工具

 ```sh
 # 由于其它工具已经安装，此处仅记录Foundry的安装，记得先准备魔法
 curl -L https://foundry.paradigm.xyz | bash
 source ~/.zshenv
 
 # It will automatically install the latest version of the precompiled binaries to system 
 foundryup
 ```

### 编写智能合约

#### 初始化智能合约项目

```sh
forge init sepolia-nft
```

- 非常方便，执行后即在当前目录创建了`sepolia-nft`目录，克隆好了[forge-std](https://github.com/foundry-rs/forge-std")仓库，组织好了项目文件
- 打开`src/Counter.sol`文件，就可以看到默认生成的智能合约了

#### 来都来了，先试试默认项目如何运行

```sh
# 本地测试
forge test

# 在reth网上测试
forge test --fork-url https://reth-ethereum.ithaca.xyz/rpc
```

#### 运行本地开发节点

```sh
anvil
# 如果运行正常，可以看到`Listening on 127.0.0.1:8545`的信息
```

#### 部署智能合约

- 部署至某节点只需要更改`private key`与`rpc-url`，私钥来源于钱包

```sh
# 确认本地节点运行后，往上翻可以看到private key，取一个进行测试
# **不建议将私钥写在命令行、代码中**
export PRIVATE_KEY="0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80"

# 部署合约至本地节点
forge script script/Counter.s.sol --rpc-url http://127.0.0.1:8545 --broadcast --private-key $PRIVATE_KEY
```

### 目前为止还没干正事儿

- [ ] 再理解下Solidity和ERC提案，编写一个`ERC-721`的智能合约
- [ ] 将`Mabinogi 2015年跨年线上音乐会游戏截图`铸造为NFT

### 查看我所获得的NFT的铸造代码

- OpenSea查看NFT详情，`Details` -> `Blockchain details`，可以看到`Contract Address`，点击后会跳转到etherscan
- 点击`Contract`即可查看智能合约源码

### 小收获

NFT协议主要有两种
- `ERC-721`，具有天然唯一性
- `ERC-1155`，可以指定数量，可以批量铸造，我所获得的`Casual Hackathon`奖牌就是通过`ERC-1155`的智能合约铸造

[^1]: Satoshi Nakamoto (中本聪)
[^2]: DPOS (Delegated Proof-of-Stake) 委托权益证明
[^3]: ICO (Initial Coin Offering) 首次代币发行

# 2025-08-05

# Day 2

## Web3安全

- 分享嘉宾: [0xRory](https://x.com/0x_rory)@[DefiHackLabs](https://defihacklab.io)

### 攻击事件记录

- 在[DeFiHackLabs](https://github.com/SunWeb3Sec/DeFiHackLabs)上收录了大量攻击事件的PoC

### 攻击特点

1. 钓鱼攻击量非常高
2. 主要目的是偷资产
3. 被盗后难以追回

### Unphishable反钓鱼训练平台

- 沉浸式体验钓鱼手法
- 重现和体验历史案例
- 邀请安全研究员贡献新的安全教育案例

### 攻击基础模式

1. 建立信任
2. 制造状况
3. 诱导操作
4. 窃取资产

### 常见套路

- #### 剪切板攻击

    > 🎯
    > - 构造相似的地址，利用人的直觉替换真实地址
    > - 监听剪切板，替换内存中的钱包地址

    > 🛡️
    > - 使用工具核对目标地址，转帐付款时复制、粘贴和发送时都要检查，不要相信直觉和眼睛
    > - 安装可信的防毒软件，养成良好习惯，接收到的文件使用联合防毒扫描引擎扫描

- #### 钓鱼网站&应用&链接&应用消息

    > 🎯
    > - 高仿网站&应用
    > - 利用AI快速生成高仿钓鱼网站&应用

    > 🛡️
    > - 细心观察甄别，或者通过本地DNS白名单过滤钓鱼网站
    > - 放慢操作速度，不要着急，认真确认每一步操作

- #### 恶意RPC提供者

    > 🎯
    > - 诱导用户切换使用恶意RPC端点

    > 🛡️
    > - 对一切保持怀疑

- #### AI生成音频&视频&网站

    > 🎯
    > - 利用朋友、上级视频或音频，诱导用户放松警惕

    > 🛡️
    > - 切换不同的认证方式，主动二次或多次确认

- #### 源码投毒

    > 🎯
    > - 利用招聘、社区协作机会，在源码内投毒，诱导用户执行恶意程序

    > 🛡️
    > - 仔细检查构建过程，避免运行恶意程序

- #### 模拟交易

    > 🎯
    > - 将恶意攻击动作插入到正常交易流程之中，以正常行为掩盖攻击行为

    > 🛡️
    > - 熟悉交易环节，对异常事件（如交易时间变长、多余的动作&动画等）

- #### 多签钱包

    > 🎯
    > - 提供一个有余额的钱包（包括助记词都给了），需要提供Gas费转出内部资金
    > - 其实该钱包是一个没有转账权限的子钱包
    > - 等用户提供Gas费后，Gas费被转走

    > 🛡️
    > - 别贪心

- #### 线下收U
    > 🎯
    > - 中间人撮合线下双方交易，双方把资产都打到中间人账户
    > - 中间人把交易双方的钱都骗走，或者双方联手骗用户

    > 🛡️
    > - 非法活动不受法律保护，不参与线下交易

## 助记词、钱包地址、私钥公钥和签名

- [助记词提案 BitCoin:BIP-39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki)
- [种子与密钥树 BitCoin:BIP-32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)
- [钱包结构 BitCoin:BIP-44](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki)
- [椭圆曲线算法](https://zh.wikipedia.org/wiki/%E6%A4%AD%E5%9C%86%E6%9B%B2%E7%BA%BF%E6%95%B0%E5%AD%97%E7%AD%BE%E5%90%8D%E7%AE%97%E6%B3%95)

### 生成顺序

    ```txt
    助记词 (Mnemonic)
        ↓（通过 BIP-39）
    种子 (Seed)
        ↓（通过 BIP-32/BIP-44 路径派生）
    私钥 (Private Key)
        ↓（通过椭圆曲线算法生成公钥，再哈希）
    地址 (Address)
    ```

### 签名与验证

- 签名过程
    - 椭圆曲线算法(私钥,  哈希(明文)) => 摘要
    - 发送摘要与明文给接收方
- 验证过程：
    - 接收摘要与明文
    - 椭圆曲线算法(公钥,  哈希(明文)) => '摘要
    - 摘要 == '摘要 ? 通过 ：失败

### 应用

- 一切从助记词开始
- 相同的助记词可以生成相同的密钥树
- 一棵密钥数可以包含无数个密钥
- 一般钱包只开放了`m/44'/60'/0'/0/0`，即第一个私钥

# 2025-08-04

# 正式打卡第一天

## 尝试 

- 昨天从[faucet](https://cloud.google.com/application/web3/faucet/ethereum/sepolia)领取了0.05个Sepolia
- 今天尝试从[sepolia-faucet.pk910.de](https://sepolia-faucet.pk910.de/)挖了2.5个Sepolia
- 给同学发了0.03个Sepolia
- 参加了以太坊中文周会并做了笔记
- 完成了[unphishable](https://unphishable.io/)钓鱼攻防挑战初级和中级内容
- 完成了[Web3 实习手册](https://web3intern.xyz/)内容的学习（其实前几天也在断断续续的看），拟了下近期学习目标
- 写了一篇关于[sepolia-faucet.pk910.de](https://sepolia-faucet.pk910.de/)的分析笔记

## 分析

- 网站
	- [sepolia-faucet.pk910.de](https://sepolia-faucet.pk910.de/)
- 行为分析
	- Sepolia是eth的测试网，已经完成PoS改造，不可能通过PoW获取到代币奖励
	- 运行该网站的mine程序后，CPU占用上升，最高可达100%，界面上可以看到Hash rate
	- 结束mine后，会向用户配置的钱包地址汇入Sepolia代币
	- 结合以上表现，猜测该网站可能以Mine Sepolia为诱饵，使用户自行执行可CPU mine的币种挖矿程序
- 官方网站声明
	- [github:pk910/PoWFaucet](https://github.com/pk910/PoWFaucet?tab=readme-ov-file)
		- For clarification: This faucet does NOT generate new coins with the "mining" process. It's just one of the protection methods the faucet uses to prevent anyone from requesting big amount of funds and draining the faucet wallet. If you want to run your own instance you need to transfer the funds you want to distribute to the faucet wallet yourself!
	- [How Our Faucet Protects Itself and You](https://github.com/pk910/PoWFaucet/wiki#how-our-faucet-protects-itself-and-you)
	- 该网站声明，PoW确实不能产生新币，mine过程只是为了防止恶意请求快速消耗faucet钱包余额，而设置的自我保护措施
- 代码分析
	- 该项目主要由三部分组成
		- 静态资源
		- typescript
			- 核心为faucet-client，实现了程序入口、三种worker和CSS外观
		- wasm
			- 包含CryptoNight, Scrypt, Argon2等抗ASIC加密算法的wasm实现，及一个sqlite3实现
	- client与server间通信通过WebSocket，端点为`/ws/pow`，通过`session id`参数进行会话管理
	- 框架使用`React`
- 运行中ws数据流监听
	- 端点
		- `wss://sepolia-faucet.pk910.de/ws/pow?session=0e17686e-05d8-41f4-a2f9-312cfb49d0d4&cliver=2.4.0`
	- Sent
		- 初始任务
			-
			  ```json
			  {
			      "id": 1,
			      "action": "foundShare",
			      "data": {
			          "nonce": 71,
			          "data": null,
			          "params": "argon2|0|13|4|4096|1|16|13",
			          "hashrate": 491.471512640656
			      }
			  }
			  ```
		- 后续任务
			-
			  ```json
			  {
			      "id": 44,
			      "action": "verifyResult",
			      "data": {
			          "shareId": "902358ad-899d-4592-bc4e-890d322525fd",
			          "params": "argon2|0|13|4|4096|1|16|13",
			          "isValid": true
			      }
			  }
			  ```
	- Received
		- 任务确认
			-
			  ```json
			  {
			      "action": "ok",
			      "data": null,
			      "rsp": 1
			  }
			  ```
		- 任务验证
			-
			  ```json
			  {
			      "action": "verify",
			      "data": {
			          "shareId": "46096c68-9f81-4ce8-b73d-a6b9ae68852b",
			          "preimage": "tMSRZ3ecDcE=",
			          "nonce": 2430725,
			          "data": null
			      }
			  }
			  ```
		- 本地状态更新
			-
			  ```json
			  {
			      "action": "updateBalance",
			      "data": {
			          "balance": "1986000000000000",
			          "reason": "valid share (reward: 1986000000000000)"
			      }
			  }
			  ```
		- 每次完成验证后，`rsp`+1开启下一个任务
- 结论
	- 使用了抗ASIC矿机的专用算法，与门罗等CPU币使用的算法高度重合，可能是为了自我保护，也可能是真实挖矿行为
	- 如果将计算部分完全剥离，用户侧只代理计算过程，与矿池通信完全放在server端，用户无法直接发现异常
	- 如果只针对钱包地址做防护，为每个地址设置24小时的冷却时间，无法防止通过大量钱包地址发起的恶意领取，使用CPU PoW确实是有效的防御手段
	- 无法证伪官方的声明，不排除使用用户CPU挖矿的可能，建议使用不需要挖矿的faucet，警惕需要额外授权的行为，注意CPU使用率是否异常升高


# 2025.07.29


<!-- Content_END -->
