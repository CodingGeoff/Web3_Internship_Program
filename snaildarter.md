---
timezone: UTC+8
---

# Neal Lee

**GitHub ID:** snaildarter

**Telegram:** @Neal_li

## Self-introduction

学习，使用，重复，反思，升华。

## Notes

<!-- Content_START -->
# 2025-08-18

已经开始编写 wishingTree 合约。

# 2025-08-17

今日开始开发许愿树项目，刚开始安装时 v3 版本，遇到有一些问题，hh 不能用，暂时退回到 V2 版本。开始设计写合约部分。

# 2025-08-16

### 快速上手：Hardhat 中使用 Viem

1. 快速开始

   - 新项目：创建时选“create a TypeScript project (with Viem)”
   - 已有项目：安装`@nomicfoundation/hardhat-viem`，添加到配置（与 ethers 兼容）

2. 核心操作（附示例）

   - 客户端交互：

     1. 新建`scripts/clients.ts`，写入代码（查余额、转账）：

        ```typescript
        import { parseEther, formatEther } from 'viem';
        import hre from 'hardhat';

        async function main() {
          const [bobWallet, aliceWallet] = await hre.viem.getWalletClients();
          const publicClient = await hre.viem.getPublicClient();

          // 查Bob余额
          const bobBalance = await publicClient.getBalance({
            address: bobWallet.account.address,
          });
          console.log(`Bob余额：${formatEther(bobBalance)} ETH`);

          // Bob转1 ETH给Alice
          const hash = await bobWallet.sendTransaction({
            to: aliceWallet.account.address,
            value: parseEther('1'),
          });
          await publicClient.waitForTransactionReceipt({ hash });
        }
        main()
          .then(() => process.exit())
          .catch(console.error);
        ```

     2. 运行：`npx hardhat run scripts/clients.ts`

   - 合约操作：

     1. 新建`contracts/MyToken.sol`写合约（示例略，含`increaseSupply`等方法），编译：`npx hardhat compile`
     2. 新建`scripts/contracts.ts`部署并调用：

        ```typescript
        import hre from 'hardhat';

        async function main() {
          // 部署合约（初始供应量100万）
          const myToken = await hre.viem.deployContract('MyToken', [
            1_000_000n,
          ]);
          console.log('初始供应量：', await myToken.read.getCurrentSupply());

          // 增加供应量50万
          const hash = await myToken.write.increaseSupply([500_000n]);
          await hre.viem.getPublicClient().waitForTransactionReceipt({ hash });
          console.log('新供应量：', await myToken.read.getCurrentSupply());
        }
        main()
          .then(() => process.exit())
          .catch(console.error);
        ```

     3. 运行：`npx hardhat run scripts/contracts.ts`

   - 测试：
     1. 新建`test/my-token.ts`写用例（验证供应量增减、无效操作回滚），示例略
     2. 运行：`npx hardhat test`

3. 版本管理
   - 稳定优先：固定`viem`、`hardhat-viem`版本（去`package.json`的`^`）
   - 功能优先：不固定版本，接受偶尔类型错误

# 2025-08-15

### 命令行短命令

```BASH
npm install --global hardhat-shorthand
```

然后就可以简化命令了

```BASH
npx hardhat compile
// 简化为
hh compile
```

### Solidity 多版本

在配置文件里配置相应的版本及设置

```JS
module.exports = {
   solidity: {
      compilers: [
         {
            version: '0.7.0',
         },
         {
            version: '0.8.0',
            settings: {},
         }
      ],
      // 如果某个文件不想用中版本的最新版本，可以主动指定特定版本，设置如下
      overrides: {
         'contracts/ContractA.sol': {
            version: '0.7.4',
            settings: {},
         },
         'contracts/ContractB.sol': {
            version: '0.8.22',
            settings: {},
         },
      },
   }
}
```

### 创建任务

可以在配置文件中创建任务及子任务，以及任务组。

```JS
task("hello-world", "Prints a hello world message").setAction(
  async (taskArgs, hre) => {
    await hre.run("print", { message: "Hello, World!" });
  }
);

subtask("print", "Prints a message")
  .addParam("message", "The message to print")
  .setAction(async (taskArgs) => {
    console.log(taskArgs.message);
  });


const myScope = scope("my-scope", "Scope description");
myScope.task("my-task", "Do something")
  .setAction(async () => { ... });
myScope.task("my-other-task", "Do something else")
  .setAction(async () => { ... });

// 调用
await hre.run({
  scope: "my-scope",
  task: "my-task",
});
```

### 写脚本

#### 使用 hh 运行脚本

```BASH
npx hardhat run script.js.
```

#### 独立脚本：使用 Hardhat 作为库

```BASH
node script.js
```

```JS
const hre = require("hardhat");

async function main() {
  const accounts = await hre.ethers.getSigners();

  for (const account of accounts) {
    console.log(account.address);
  }
}

main().catch((error) => {
  console.error(error);
  process.exitCode = 1;
});
```

# 2025-08-14

### hardhat tasks，console

在 hardhat 的 config.ts 文件中增加调用 task 方法。

```ts
import { HardhatUserConfig, task } from 'hardhat/config';
import '@nomicfoundation/hardhat-toolbox';

const config: HardhatUserConfig = {
  solidity: '0.8.20',
};

task('accounts', 'Prints the list of accounts', async (taskArgs, hre) => {
  const accounts = await hre.ethers.getSigners();

  for (const account of accounts) {
    console.log(account.address, 'i');
  }
});

export default config;
```

借此可以自己定义一些方法，解决重复的功能。

#### 为了更好的调试

```BASH
npx hardhat console
```

然后可以获取一些信息，config、ethers 和 hrt 等。
根据这些信息和区块交互。

# 2025-08-13

看了 hardhat 的文档，根据文档动手操作了。为开发许愿树项目做准备。

# 2025-08-12

Uniswap V4 在 V3 基础上进行了多项重要升级，通过引入 Hooks 机制、采用单例模式等，进一步增强了平台的灵活性与可扩展性，降低了交易成本。以下是具体介绍：

- 核心功能升级：
  - Hooks 机制：这是 V4 最核心的创新。Hooks 本质是定制化合约，通过在交易池初始化时指定合约地址，可在 initialize、modifyPosition、swap 等关键操作前后添加自定义逻辑。例如实现随时间执行大额订单、链上限价单、动态费用等功能，还能为流动性提供者内化 MEV，大大提高了 Uniswap 的可扩展性。
  - 单例模式：V4 改用单例模式，所有池子由 PoolManager 合约统一管理。相比之前每个池子都是单独合约，新建池子无需部署新实例，降低了创建池子的成本。
  - 闪电记账：结合 EIP-1153 中提议的瞬态存储操作码，在交易操作前先锁定，仅更新内部净余额 delta，交易结束时再进行外部转账，减少了 Gas 消耗。
  - 支持原生 ETH：V4 重新支持原生 ETH 交易，且允许同时支持 ETH 和 WETH 的配对。由于 ETH 转账 Gas 成本约为 ERC20 转账的一半，此举降低了交易成本。此外，还引入 ERC1155 代币用于额外的代币记账，用户可将代币保留在单例合约中，避免 ERC20 频繁转入和转出。
- 对用户的影响：
  - 对交易者：交易成本进一步降低，由于 Gas 费用的优化以及动态费率机制，交易者有望获得更优的交易体验，尤其是在交易大额订单或高波动代币时。同时，可通过 Hooks 机制实现的限价单等功能，能更精准地执行交易策略。
  - 对流动性提供者（LP）：LP 对流动性池的管理更加灵活，可通过 Hooks 机制定制流动性池，实现个性化的收益策略。例如根据市场波动自动调整费率，或设计与社区治理相关的流动性池， potentially 获得更高的收益。
- 生态发展：目前 Uniswap V4 已经上线于 Ethereum、Polygon、Arbitrum 等多个区块链网络，流动性正在逐步向 V4 迁移。Uniswap 不仅专注于技术创新，还积极推动社群主导的开发模式，挂钩机制让开发者能够设计创新的 DeFi 应用，促进 Uniswap 生态系统的繁荣。

# 2025-08-10

Uniswap V3 是去中心化交易所（DEX）Uniswap 系列的第三个重要版本，于 2021 年 5 月推出，相比 V1 和 V2 进行了多项颠覆性创新，核心目标是提升资本效率、增强灵活性，同时优化交易者和流动性提供者（LP）的体验。以下从核心特点、关键改进、优势与局限等方面进行总结：


### **一、核心特点与创新**
1. **集中流动性（Concentrated Liquidity）**  
   - 这是 V3 最核心的突破。在 V1 和 V2 中，流动性被均匀分配在 **0 到无穷大** 的价格区间内，导致大量资金闲置（尤其是远离当前价格的区间）。  
   - V3 允许 LP 自主选择 **特定价格区间** 部署流动性（例如，将 USDC/ETH 的流动性集中在 1500-2000 USDC/ETH 区间）。当交易价格在该区间内时，LP 的资金才会被用于交易；若价格超出区间，流动性则暂时“失效”。  
   - 效果：相同资金量下，集中流动性可提供 **10-100 倍的资本效率**（即同等资金能支持更大交易量，滑点更低）。

2. **多费率层级（Multiple Fee Tiers）**  
   - V3 为每个交易对引入了 **三种费率选项**：0.05%、0.3%、1%（部分场景可能有特殊费率）。  
   - LP 可根据代币对的波动性选择费率：高波动代币对（如小市值山寨币/ETH）适合高费率（1%），低波动稳定币对（如 USDC/USDT）适合低费率（0.05%）。  
   - 交易者则根据需求选择费率池（低费率池滑点可能更高，高费率池滑点更低），实现风险与成本的平衡。

3. **流动性头寸以 NFT 表示**  
   - 在 V1/V2 中，LP 提供流动性后会收到 ERC-20 代币（如 UNI-V2 代币），代表对流动性池的份额（所有 LP 共享同一池的收益）。  
   - V3 中，每个 LP 的流动性头寸（因价格区间、费率不同而唯一）被封装为 **ERC-721 NFT**，每个 NFT 对应特定区间和费率的流动性份额。这意味着 LP 可精准管理单个头寸（如添加、移除、合并流动性），灵活性大幅提升。

4. **限价单功能（模拟）**  
   - V3 未直接引入限价单，但通过“集中流动性”间接实现：用户可在目标价格区间提供流动性（例如，希望以 1800 USDC 买入 ETH，可在 1790-1810 区间部署 USDC 流动性）。当市场价格进入该区间时，交易自动执行，相当于“限价单”。


### **二、对用户的影响**
- **对交易者**：  
  - 滑点显著降低（因资本效率提升），尤其在主流交易对中。  
  - 可根据成本偏好选择不同费率池（如高频交易者倾向低费率池）。  

- **对 LP**：  
  - 收益潜力提升：集中流动性可让 LP 在活跃价格区间内获得更高交易费分成。  
  - 管理复杂度增加：需主动选择价格区间（若价格移出区间则无收益），需频繁调整以适应市场波动（“主动做市”）。  


### **三、部署与生态扩展**
- **跨链与 Layer2 支持**：V3 最初部署在以太坊主网，后扩展至 Polygon、Arbitrum、Optimism 等 Layer2 网络，大幅降低 gas 费用（尤其以太坊主网 gas 高昂的问题），提升小额交易可行性。  
- **生态集成**：被大量 DeFi 协议集成（如借贷平台、聚合器），成为去中心化交易的核心基础设施之一。


### **四、优势与局限**
| **优势** | **局限** |
|----------|----------|
| 资本效率大幅提升，减少资金闲置 | LP 需主动管理价格区间，对新手不友好 |
| 费率灵活，适配不同波动性代币对 | 价格移出区间时，LP 无交易费收益 |
| 支持“模拟限价单”，功能更丰富 | NFT 流动性头寸难以拆分或转移（相比 ERC-20） |
| 扩展至 Layer2，降低使用成本 | 协议复杂度增加，潜在漏洞风险上升 |


### **五、总结**
Uniswap V3 通过“集中流动性”和“多费率层级”重新定义了 AMM（自动做市商）的效率，让 DEX 更接近中心化交易所的资本利用率。它更适合专业 LP 和高频交易者，但也因复杂度提高对新手不够友好。作为 DeFi 领域的标杆创新，V3 推动了整个行业对资本效率的探索，后续许多 DEX（如 Curve V2、Balancer V2）均借鉴了其设计思路。

# 2025-08-09

Uniswap V2 是去中心化交易所（DEX）Uniswap 的第二个主要版本，于 2020 年 5 月推出，在 V1 基础上进行了多项关键升级，进一步巩固了其在去中心化金融（DeFi）领域的核心地位。以下从核心机制、关键功能、手续费模式、局限性及意义等方面进行总结：


### **一、核心定位**  
Uniswap V2 是基于以太坊的去中心化交易所，采用 **自动做市商（AMM）机制**，无需传统订单簿，通过用户提供的流动性池实现代币交易，核心目标是降低去中心化交易的门槛，提升流动性效率。


### **二、核心机制：恒定乘积公式**  
V2 延续了 AMM 的核心逻辑，通过 **x*y=k** 公式维持流动性池的平衡（x 和 y 分别代表池中的两种代币数量，k 为常数）。  
- 当用户买入一种代币时，池中该代币数量减少，另一种代币数量增加，价格随供需自动调整（买盘推动价格上涨，卖盘推动价格下跌）。  
- 流动性提供者（LP）向池内注入两种代币（需按当前比例），获得代表其份额的 LP 代币，后续可通过赎回 LP 代币取回本金及交易手续费分成。  


### **三、关键功能与升级（相比 V1）**  
1. **支持 ERC20-ERC20 直接交易**  
   V1 仅支持 ETH 与 ERC20 代币的交易对，而 V2 允许任意两种 ERC20 代币直接交易（例如 USDC-USDT、DAI-WBTC 等），无需通过 ETH 作为中介，减少了交易步骤和 gas 费用。

2. **工厂模式（Factory Contract）**  
   引入 **Factory 合约**，用于自动创建和管理交易对（Pair 合约）。任何用户都可通过 Factory 合约创建新的 ERC20-ERC20 交易对，无需人工审核，极大提升了扩展性（截至目前，V2 已支持数万种交易对）。

3. **闪电贷（Flash Swaps）**  
   首次在 DEX 中原生支持闪电贷功能：用户可在同一笔交易中无抵押借入代币，只要在交易结束前归还同等数量（含少量手续费）即可。这为套利、清算等 DeFi 操作提供了便利。

4. **内置价格预言机**  
   每个交易对（Pair 合约）会记录过去 60 分钟的价格数据（时间加权平均价格，TWAP），开发者可直接调用这些数据作为价格参考（尽管精度低于专业预言机，但为低成本预言机需求提供了解决方案）。

5. **改进的流动性管理**  
   - 支持流动性池的动态调整（如添加/移除流动性）。  
   - LP 代币标准化：采用 ERC20 标准，便于在其他 DeFi 协议中作为抵押品使用（例如质押到借贷协议获取贷款）。


### **四、手续费模式**  
- 每笔交易收取 **0.3% 的手续费**，全部归流动性提供者（LP）所有（按其在池中的份额分配）。  
- 无平台额外收费，纯去中心化分配机制。  


### **五、局限性**  
1. **无常损失（Impermanent Loss）**  
   当池中两种代币的价格波动较大时，LP 赎回资产的价值可能低于初始投入（需通过交易手续费弥补），这是 AMM 机制的固有风险。

2. **滑点问题**  
   对于大额交易，由于恒定乘积公式的特性，价格滑点（实际成交价与预期价的偏差）可能较大，尤其在流动性较浅的交易对中更明显。

3. **仅支持以太坊**  
   V2 仅部署在以太坊主网，受限于以太坊的高 gas 费和拥堵问题（后续版本 V3 及其他链的部署部分解决了这一问题）。

4. **价格预言机精度有限**  
   内置 TWAP 数据易受短期大额交易操纵，不适合高精度场景（如大额借贷清算）。


### **六、意义与影响**  
- 推动了 AMM 模式的普及，成为 DeFi 基础设施的标杆，启发了大量同类 DEX（如 SushiSwap、PancakeSwap 等）。  
- 降低了去中心化交易的门槛，让任何人都能创建交易对和提供流动性，促进了代币的自由流通。  
- 为后续版本（如 V3 的集中流动性机制）奠定了技术和生态基础。


总之，Uniswap V2 是 DeFi 发展的关键里程碑，通过简化交易流程、提升扩展性和创新性功能，极大地推动了去中心化金融的规模化应用。

# 2025-08-08

### Uniswap V1 是基于以太坊区块链的去中心化交易协议，于 2018 年 11 月推出。它是去中心化交易所（DEX）领域的先驱，采用自动做市商（AMM）机制，以恒定乘积公式“x×y=k”确定兑换比例。以下是具体介绍：

- 核心特点：
  - 交易对限制：仅允许 ETH 和 ERC-20 代币之间的交换，不支持 ERC-20 代币直接互换。
  - 自动做市商机制：通过流动性池实现交易，流动性提供者将 ETH 和 ERC-20 代币存入智能合约形成资金池，合约按照恒定乘积公式与买卖双方完成交易。
  - 手续费模式：收取交易币种的 0.3%作为手续费，分配给流动性提供者作为激励。
- 体系组成：
  - Exchange：用于进行 ETH 和 ERC-20 代币之间的兑换。
  - Factory：用于创建和记录所有的 Exchange，也可用于查询代币对应的 Exchange。
- 优势：
  - 用户体验良好：恒定乘积做市算法能提供无限价格范围报价，小额交易滑点低。
  - 激励机制合理：向交易者收取 0.3%的手续费分给流动性提供者，激励方式简明。
  - 运营成本低：底层设计简单，依靠智能合约及算法自动化执行。
  - 无须许可：任何代币均可上市交易，满足了用户对长尾资产的需求。
  - 进入壁垒低：用户无需注册或进行 KYC，仅需 EVM 钱包即可交易。
- 局限性：
  - 交易对单一：只能进行 ETH 与 ERC-20 代币的交易，无法直接实现 ERC-20 代币之间的兑换，交易灵活性受限。
  - 运作成本高：链上 Gas 费用较高，若不是显著的链上牟利，交易数量会受限。
  - 价格稳定性差：对于大额交易，容易出现价格滑点，且存在被市场操纵的风险。
- 意义：Uniswap V1 以简洁的代码开启了 DEX 对抗 CEX 的历程，为后续 Uniswap 版本的发展奠定了基础，其创新机制也为其他 AMM 协议提供了启示，推动了去中心化交易市场的发展。

# 2025-08-07

#### 核心法律风险梳理

1. 代币发行与交易行为的法律风险
2. 赌博、传销、洗钱等刑事风险
3. 场外交易中的洗钱与非法经营风险
4. 民商争议

如果是内部人作恶，那用户就自求多福了。项目方内部的安全风控一定要特别重视人员安全，这永远是最值得花成本、花精力去建设的。人是最大的那只特洛伊木马，但却最容易被忽视。有的人安全意识实在太差，在安全上又不思进取。这种人，谁招谁倒霉。

# 2025-08-06

### 稳定币知识

#### 稳定币与区块链基础

目标： 搞懂”稳定币是什么、为什么存在、运行在什么环境上“

1. 稳定币核心概念
   _稳定币是锚定法币（如 USD）、加密资产或算法的加密货币，核心作用是解决加密货币价格波动问题，成为区块链生态的”支付媒介“”价值储备“。_

分类与代表：

- 法币抵押型：USDC（Circle 发行，1:1 锚定 USD, 有法币储备）、USDT（Tether，储备透明度较低）、TUSD（TrueUSD， 严格审计）。
- 加密资产抵押型：DAI（MakerDAO 发行，由 ETH 等加密资产超额抵押生成）、sUSD（Synthetix, 抵押 SNX 生成）。
- 算法型（非主流，风险高）：早期 UST（Terra，已崩盘）、FRAX（部分算法+部分抵押）。

前端开发中接触最多的是法币抵押型（USDC/USDT）和加密资产抵押型（DAI）。

2. 区块链基础（稳定币的”运行土壤“）
   核心概念：区块链是”去中心账本‘，稳定币的转账、余额记录都在链上。

- 公链（以太坊，BSC，Polygon、Solana 等）：不同链有自己的稳定币（如以太坊上的 USDC 和 BSC 上的 USDC 是不同资产）。
- 账户与地址：用户通过钱包地址（0x 开头的字符串）持有稳定币，前端需要处理地址格式验证（如以太坊地址是 42 位十六进制字符串）。
- 交易与 Gas：稳定币转账是“链上交易”，需要支付 Gas 费（用链上原生代币，如以太坊上用 ETH，BSC 上用 BNB），前端需要处理 Gas 估算、交易状态查询。

3. 前端视角的“稳定币与区块链”的关联
   前端开发无需深入区块链底层（如共识机制），但需要知道：稳定币的所有数据（余额、转账记录）都在链上，前端通过“区块链 API”或“钱包工具”读取/写入这些数据。

#### 稳定币的链上交互逻辑

目标：理解稳定币在链上的“数据格式”“交互规则”

1. 稳定币的标准化：ERC-20 协议

- 90%以上的稳定币（USDC、USDT、DAI 等）遵循以太坊的 ERC-20 标准（其他公链有类似标准，如 BSC 的 BEP-20、Solana 的 SPL）。
- ERC-20 定义了稳定币的核心接口（前端需要调用的函数）：
  - balanceOf(address user): 查询用户地址的稳定币余额（前端常用，如显示你的 USDC 余额）
  - transfer(address to, uint256 amount): 转账（前端实现给他人转 USDC 功能）
  - approve(address spender, uint256 amount): 授权其他合约/地址使用你的稳定币（如在 Uniswap 中用 USDC 换 ETH，需要先授权 Uniswap 合约使用你的 USDC）。
  - 这些接口是“智能合约函数”，前端通过“钱包+PRC 节点”调用（类似调用后端 API，但调用对象是链上合约）。

2. 稳定币的链上数据结构

- 余额：以最小单位存储（如 USDC 的最小单位是“分”， 1USDC = 1e6 最小单位，即 1USDC=1000000wei）。
- 从链上读取的余额是“最小单位数值”，需要转换为“用户可读单位”（如 1000000 -> 1 USDC）。
- 转账记录：链上每笔稳定币转账都会生成“交易哈希（txHash）”,可通过区块链浏览器（如 Etherscan）查询，前端可通过 txHas 跟踪交易状态（成功/失败/pending）。

3. 钱包与稳定币交互

- 钱包（如 MetaMask）是用户与链上资产交互的入口，前端需要通过钱包获取用户地址、发起交易。
- 核心流程（以转账 USDC 为例）
  1. 前端通过 window.ethereum(MeatMask 注入的对象)获取用户地址。
  2. 调用 USDC 合约的 balanceOf 方法，显示用户余额。
  3. 用户输入接收地址和金额后，前端调用 transfer 接口，钱包弹出签名框（用户确认交易）。
  4. 交易上链后，前端通过 txHash 查询结果，更新 UI。

#### 前端开发工具和稳定币功能实现

目标：掌握前端与稳定币交互的工具链，能独立开发稳定币相关功能（查余额、转账、授权等）。

1. 核心工具与库
   钱包交互：检测是否安装钱包，获取 Provider（链上交互的入口）。
   钱包时间监听：用户切换链/地址时，前端需要同步更新数据（accountsChanged、chainChanged 事件）。
   链上数据交互：（viem，ethers.js web3.js）封装了与区块链交互的方法，直接调用 ERC-20 合约接口。
   链 ID 与合约地址：不同链的稳定币合约地址不同，前端需要维护“链 ID-合约地址”映射表。
   开发环境：

   - Hardhat：本地开发和测试环境，模拟链上交互。
   - 区块链浏览器：Etherscan（以太坊）、BscScan（BSC），查询真实合约地址、交易记录、辅助调试。

2. 核心功能实现案例

- 稳定币余额查询工具
  功能：用户连接钱包后，显示其在当前链上的 USDC、USDT、DAI 余额。
  关键点：处理不同稳定币的 decimals（USDC/USDT 是 6，DAI 是 18），统一转换为可读格式。
- 稳定币转账 DApp
  功能：用户输入接收地址和金额，发起稳定币转账，显示交易状态。
  关键点：
  验证接收地址格式（用 isAddress）；
  处理用户拒绝交易（钱包签名框被取消）；
  监听交易上链状态（provider.waitForTransaction(txHash)）。
- 与 DeFi 协议交互（如 Aave 存稳定币）
  功能：用户将 USDC 存入 Aave 获取利息。
  关键点：
  先调用 USDC 的 approve 授权 Aave 合约使用用户的 USDC；
  再调用 Aave 的 deposit 函数存入 USDC（需理解 Aave 合约的接口）。

#### 稳定币的进阶场景与前端优化

目标：应对复杂场景，解决前端开发中的实际问题。

1. 跨链稳定币交互
   同一稳定币可能在多链发行（如 USDC 在以太坊、Polygon、Avalanche 均有版本），前端需支持：
   检测用户当前链，显示对应链的稳定币数据；
   引导用户切换链（如用户在 Polygon 链，想转以太坊 USDC，需提示切换链）；
   跨链转账功能（调用跨链桥合约，如 Polygon Bridge，前端需处理跨链交易的状态跟踪）。
2. 稳定币的 “合规与安全” 前端处理
   合规：部分场景需验证稳定币的 “合规性”（如只支持受监管的 USDC，不支持匿名稳定币），前端可通过合约地址白名单实现。
   安全：
   不存储用户私钥 / 助记词（前端只通过钱包交互，私钥由钱包管理）；
   验证合约地址（防止用户输入假 USDC 合约地址，可通过链上注册表查询官方地址）；
   处理重入攻击（前端无需深入，但需调用经过审计的合约，避免与恶意合约交互）。
3. 前端性能优化
   链上数据查询延迟：用 provider.getBalance 等接口查询余额时，可能因链上区块同步延迟返回旧数据，前端可：
   结合区块链浏览器 API（如 Etherscan API）做二次验证；
   监听区块事件（provider.on("block", ...)），区块更新后自动刷新数据。
   大量稳定币数据展示：如显示用户在 10 条链上的 5 种稳定币余额，可通过批量查询（multicall 合约）减少链上请求次数。

#### 实践与深入

目标：通过真实项目巩固知识，理解稳定币生态的深层逻辑。

1. 实战项目
   开发一个 “稳定币资产管理 DApp”：支持多链稳定币余额查询、跨链转账、DeFi 收益聚合（存到 Aave/Compound）。
   参与开源项目：如在 Uniswap 前端（开源）中添加 “稳定币兑换” 快捷功能，或在 MetaMask 插件中优化稳定币显示逻辑。
2. 深入稳定币生态
   理解稳定币的 “发行与赎回”：如 USDC 的发行流程（用户给 Circle 转 USD，Circle 在链上 mint USDC）、DAI 的生成（抵押 ETH 到 MakerDAO，生成 DAI），前端可开发 “DAI 生成计算器”（根据抵押 ETH 数量计算可生成 DAI 上限）。
   关注稳定币风险：如 UST 崩盘的原因（算法型稳定币的 “死亡螺旋”）、USDC 的储备风险（Circle 是否真有 1:1 法币储备），前端可在 DApp 中添加 “稳定币风险评级” 提示。

总结
前端开发学习稳定币的核心逻辑是：“用前端技术连接用户与链上稳定币数据”。从 “是什么” 到 “怎么交互”，再到 “怎么优化交互”，最终通过实践将稳定币功能融入 DApp。重点关注 ERC-20 接口、钱包交互、链上数据处理，利用 viem 等工具将前端技能与区块链场景结合，就能快速上手。

# 2025-08-05

### 安全

1. 重入攻击
   利用外部合约在 fallback 中重新调用原函数。历史上最著名的 The Dao 事件便因重入漏洞导致合约 6000 万美元 ETH 被盗，最终造成以太坊社区分裂（形成 ETH/ETC 链）。
   防护方法：先更新状态，在转账。

```solidity
function withdraw() public {
    require(balance[msg.sender] > 0);
    (bool sent,) = msg.sender.call{value: balance[msg.sender]}("");
    require(sent);
    balance[msg.sender] = 0;
}


// good
function withdraw() public {
    uint256 amount = balance[msg.sender];
    balance[msg.sender] = 0;
    (bool sent,) = msg.sender.call{value: amount}("");
    require(sent);
}
```

2. 预言机操纵
   依赖外部价格源的不可信更新。
   解决方法：

   - 使用 Chainlink 等权威价格源。
   - 增加时序约束和多源验证。
   - 使用 twap 等加权算法。

3. 整数溢出/下溢
   使用 unchecked {} 时需要确保逻辑安全。
   推荐使用 Solidity 0.8+ 的内建溢出检查或 SafeMath.

4. 权限控制缺失
   所有管理函数引用 onlyOwner 或 AccessControl 修饰符保护。
5. 未初始化代理

- 基于代理模式的合约未正确初始化函数，可能被任意人初始化并接管合约。
- 著名的例子包括 Harvest Finance 其在使用 Uniswap V3 做市策略的 Vault 合约中存在未初始化漏铜，如果被利用攻击者可销毁实现合约。该团队曾为此漏洞支付高额赏金修复。

6. 前置交易/三明治攻击

- 攻击者在交易执行前后分别发送交易，以不利滑点或套利为目的。
- 例如 2025 年 3 月，一名用户在 Uniswap V3 的稳定币兑换中遭遇三明治攻击，约 21.5 完美元的 USDC 兑换几乎被抢跑，损失 98%的资金。

# 2025-08-04

## Dapp 架构

### 前端

界面 React/Vue
数据管理 Zustand/Redux
工具库 Typescript、Tailwind
了解钱包的库：Viem、wagmi、Rainbow

### 智能合约

solidity： hardhat

### 数据检索器

智能合约通常以 Event 形式释放日志事件，比如释放代表 NFT 转移的 Transfer 事件，数据检索器会检索这些数据并将其写入到 PostgreSql 等传统数据库中。
Dapp 在前端进行数据展示时需要检索器内的数据。一个简单的示例是某个 NFT 项目需要展示用户持有的所有 NFT，但是 NFT 合约并不会提供通过输入地址参数返回该地址下的所有 NFT 的函数，此时我们可以运行检索器将 Transfer 事件读取后写入传统数据库内，前端可以在传统数据库内检索用户持有的 NFT 数据。

### 区块链和去中心化存储

区块链用于存储智能合约的状态数据以及交易记录。去中心化存储如 IPFS 或 Arweave，用于存储大规模的非结构化数据（如图片、文档等），确保数据不易丢失和篡改。
通过使用去中心化存储，Dapp 确保所有数据在多个节点上备份，保证数据的持久性和去中心化的特性。

### 常见优化技巧

#### 减少存储操作

- 读取存储第一次需 2100 gas（后续 100 gas），而内存读取仅 3 gas。推荐多次访问同一存储数据时，将其缓存到内存以减少 SLOAD 次数
- 每次写入 storage 的成本高达 20,000 gas；优先使用 memory。
- 示例：

```solidity
// ❌ 非优化写法
mapping(address => uint256) public balances;
function deposit() public payable {
    balances[msg.sender] += msg.value;
}

// ✅ 优化写法（一次读，一次写）
function deposit() public payable {
    uint256 current = balances[msg.sender];
    balances[msg.sender] = current + msg.value;
}
```

#### 使用位压缩（Bit Packing）

- 将多个变量压缩到一个 uint256 中以节省存储空间。
- 示例：

```solidity
struct Packed {
    uint128 a;
    uint128 b;
}
```

#### 循环优化

- 减少不必要的运算，如 array.length 缓存到变量中。
- 示例

```solidity
// ❌ 非优化
for (uint256 i = 0; i < arr.length; i++) {
    ...
}
// ✅ 优化
uint256 len = arr.length;
for (uint i = 0; i < len; ++i) {
    ...
}
```

#### 函数可见性选择

external 比 public 更节省 gas，适用于仅背外部调用的函数。


# 2025.07.29


<!-- Content_END -->
