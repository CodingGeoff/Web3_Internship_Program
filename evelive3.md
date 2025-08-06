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
