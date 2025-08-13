---
timezone: UTC+8
---

# wanlu

**GitHub ID:** wangwanlu09

**Telegram:** @wangwanlu

## Self-introduction

我是一名初级开发，正在学习以太坊智能合约调用相关知识，比如wagmi等，相关demo已经在我的repo中，希望未来学习更多web3相关开发知识

## Notes

<!-- Content_START -->
# 2025-08-13

## 今日学习内容总结（Foundry + Solidity + WSL 环境）

### 一、环境搭建

1. **安装 WSL（Windows Subsystem for Linux）**

   * 在 Windows 11 上安装 WSL2 并选择 Ubuntu 作为 Linux 发行版。
   * 设置 Linux 用户名和密码完成初始化。

2. **安装 Foundry**

   * 在 WSL 的 Ubuntu 终端执行：

     ```bash
     curl -L https://foundry.paradigm.xyz | bash
     source ~/.bashrc
     foundryup
     ```
   * 验证安装：

     ```bash
     forge --version
     anvil --version
     ```
   * 成功安装 `forge`、`anvil`、`cast`、`chisel`。

3. **VS Code 配合 WSL**

   * 安装 VS Code 的 WSL 扩展，实现直接在 Windows 编辑 Linux 文件，并在 WSL 终端运行命令。
   * 项目可以放在 Windows 文件系统中，通过 WSL 访问。

### 二、Foundry 项目初始化

1. 创建新项目：

   ```bash
   cd /mnt/c/Projects/Defi
   forge init MyFirstFoundryProject
   ```
2. 安装依赖：

   ```bash
   cd MyFirstFoundryProject
   git init
   forge install OpenZeppelin/openzeppelin-contracts
   ```

   * 成功安装 OpenZeppelin 合约库，为后续合约编写做准备。

### 三、Solidity 合约准备

* 学习了 Solidity 的基本语法：

  * 文件头：

    ```solidity
    // SPDX-License-Identifier: UNLICENSED
    pragma solidity ^0.8.20;
    ```
  * 可以创建简单合约，如 `Counter`：

    ```solidity
    contract Counter {
        uint256 public number;

        function setNumber(uint256 newNumber) public {
            number = newNumber;
        }

        function increment() public {
            number++;
        }
    }
    ```
* 了解了 ERC20 合约引用方式（OpenZeppelin）：

  ```solidity
  import {ERC20} from "openzeppelin-contracts/contracts/token/ERC20/ERC20.sol";
  ```
* 可以创建一个最基础的代币合约骨架：

  ```solidity
  contract MyToken is ERC20 {
      constructor(string memory name, string memory symbol, uint256 initialSupply) ERC20(name, symbol) {
          _mint(msg.sender, initialSupply);
      }
  }
  ```

### 四、总结

* 今天主要掌握了 **开发环境搭建**、**Foundry 项目初始化** 以及 **Solidity 合约基础编写**。
* 尚未进入测试和前端交互阶段，为后续开发打下基础。
* 下一步计划：完成合约逻辑后再进行本地测试和前端连接。

# 2025-08-12

### 今天学习内容总结

1. **DeFi 合约基础**

   * 了解了 ERC4626 Vault 合约是什么，及其作为标准化金库的基本作用
   * 理解了 Solidity 合约中的 `import` 语法和接口定义（interface）
   * 学习了常用的合约工具库：`Ownable`（权限控制）、`ReentrancyGuard`（防重入攻击）

2. **Solidity 智能合约基础**

   * 写了简单的存取款合约示例，理解了变量、函数、事件、权限控制等基本概念
   * 学习了如何在本地使用 Hardhat 或 Ganache 部署和测试智能合约

3. **前端与智能合约的关系**

   * 明白了前端调用智能合约的基本流程和关键点，如合约接口、交易签名、钱包连接
   * 了解了使用 Scaffold-ETH 框架搭建前端项目的基本步骤

4. **Git 远程仓库操作**

   * 解决了远程仓库地址冲突的问题，掌握了如何删除旧远程地址并添加新远程地址

# 2025-08-11

#提前预习什么是DApp
---

### 1. DApp 基础介绍

* **DApp**：基于区块链的去中心化应用，无中心服务器，数据和逻辑由链和智能合约保障。
* **区别**：传统应用依赖中心服务器，DApp依赖区块链网络。
* **核心组成**：前端界面、智能合约、区块链网络。
* **去中心化优势**：安全透明、抗审查、用户掌控数据。

### 2. DApp 架构设计

* **智能合约层**：链上业务逻辑代码。
* **前端应用**：用户交互界面（网页/移动端）。
* **区块链节点连接**：RPC节点，钱包集成。
* **数据存储**：链上存储有限，链下数据用IPFS等去中心化存储。
* **安全设计**：合约安全审计、交易签名验证。

### 3. 开发环境搭建

* **工具**：Solidity、Hardhat、Truffle、Remix。
* **本地链**：Ganache、Hardhat Network。
* **前端框架**：React + web3.js/ethers.js。

### 4. 智能合约开发

* **合约编写**：Solidity语言。
* **编译与部署**：自动化脚本（Hardhat、Truffle）。
* **测试调试**：单元测试，安全审计。
* **优化**：Gas优化、漏洞修复。

### 5. 前端开发

* **链交互**：调用合约方法，监听事件。
* **钱包连接**：MetaMask、WalletConnect。
* **交易处理**：签名、发送、确认。
* **UI设计**：用户友好、安全提示。

### 6. 集成与测试

* **测试网部署**：Ropsten、Rinkeby、Polygon Mumbai。
* **集成测试**：合约与前端联调。
* **性能与异常**：优化响应，处理失败交易。

### 7. 部署上线

* **主网部署**：智能合约发布到主链。
* **前端托管**：Netlify、Vercel、IPFS + ENS。
* **版本管理**：代理合约升级策略。

### 8. 运维与监控

* **节点维护**：自己跑节点或用第三方节点。
* **交易监控**：跟踪交易状态，数据分析。
* **用户反馈**：收集改进，版本迭代。

# 2025-08-10

今日学习内容包括两部分：
1.	NFT 图片资源的存储与访问
了解了如何将 NFT 图像文件上传至去中心化存储网络（如 IPFS），并通过生成的 CID（Content Identifier）获得唯一可访问的 imageURI。
2.	基于 ECDSA 签名的铸造授权机制
掌握了在智能合约中通过指定 _signer 公钥来验证签名的流程。签名由项目方使用其对应私钥离线生成，包含 imageURI 等数据。合约在 mint 时使用 _verify 方法校验签名有效性，从而确保只有获得项目方授权的资源可被铸造为 NFT，并且 NFT 最终归属于发起交易的 msg.sender 地址。

# 2025-08-09

前两天Scaffold-ETH版本装成了旧版。今日改为新版，有了最新的Next+wagmi+viem等，今日正式开始全面搭建NFT.

# 2025-08-08

# 使用 Scaffold-ETH + Next.js + wagmi + viem + RainbowKit 开发 DApp

---

## 一、背景和方案选择

* **目标**：用 Scaffold-ETH 提供的 Hardhat 环境写智能合约，部署到测试网（如 Sepolia）。
* **前端**：放弃 Scaffold-ETH 自带的 React-app，使用全新 Next.js 框架，搭配 wagmi、viem 和 RainbowKit 实现钱包连接和合约交互。
* **原因**：

  * Scaffold-ETH 提供了标准的合约开发和部署工具链，方便快速迭代智能合约。
  * Next.js 是更现代和流行的 React 框架，支持 SSR/ISR，有更好的性能和开发体验。
  * wagmi + viem 提供轻量且强大的以太坊交互 Hooks 和工具。
  * RainbowKit 提供美观且易用的钱包连接 UI。

---

## 二、开发环境准备

### 1. Scaffold-ETH 合约环境搭建

* 初始化项目（可用官方模板）。

* **安装依赖**：

  * 遇到 `gluegun`、`concat-stream` git 依赖无法拉取的问题，需在 `package.json` 里添加 `"resolutions"`，指定正确版本：

    ```json
    "resolutions": {
      "gluegun": "^5.2.0",
      "concat-stream": "^2.0.0"
    }
    ```
  * 解决 node 版本兼容问题，Hardhat 2.11.2 要求 Node 版本 14/16/18，当前用 Node 22 会报错，建议切换到符合版本。

* 部署合约需要：

  * `.env` 配置测试网 RPC 地址（如 Alchemy 或 Infura 提供的 Sepolia RPC）。
  * 私钥（钱包的私钥）用于部署。

* 运行 `yarn deploy` 或 `npx hardhat deploy --network sepolia` 部署合约。

### 2. OpenZeppelin 依赖安装

* 在合约包目录（如 `packages/hardhat`）用 yarn 添加：

  ```
  yarn workspace @scaffold-eth/hardhat add @openzeppelin/contracts
  ```
* 确保依赖成功安装且能导入。

---

## 三、前端开发环境搭建（Next.js + wagmi + viem + RainbowKit）

### 1. 新建 Next.js 项目

* 在 Scaffold-ETH 项目外（或任意目录）运行：

  ```
  npx create-next-app@latest my-nft-frontend
  ```
* 选择配置（ESLint、Tailwind、App Router等按需选择）。

### 2. 安装关键依赖

```
cd my-nft-frontend
yarn add wagmi viem @rainbow-me/rainbowkit ethers
```

### 3. 基本配置

* 在 `_app.tsx` 或 `app/layout.tsx` 里配置 RainbowKit 和 wagmi Provider：

```tsx
import { WagmiConfig, createClient, configureChains, mainnet, sepolia } from 'wagmi';
import { publicProvider } from 'wagmi/providers/public';
import { RainbowKitProvider, getDefaultWallets } from '@rainbow-me/rainbowkit';
import '@rainbow-me/rainbowkit/styles.css';

const { chains, provider } = configureChains(
  [sepolia, mainnet],
  [publicProvider()]
);

const { connectors } = getDefaultWallets({
  appName: 'My NFT DApp',
  chains,
});

const wagmiClient = createClient({
  autoConnect: true,
  connectors,
  provider,
});

function MyApp({ Component, pageProps }) {
  return (
    <WagmiConfig client={wagmiClient}>
      <RainbowKitProvider chains={chains}>
        <Component {...pageProps} />
      </RainbowKitProvider>
    </WagmiConfig>
  );
}

export default MyApp;
```

### 4. 调用合约示例

* 使用 `viem` 和 `wagmi` hooks 调用部署好的合约的 mint 函数或读取数据。

---

## 四、部署与验证

* 合约部署到 Sepolia 测试网，确保 `.env` 配置正确。
* 通过 Etherscan 测试网查看部署和交易状态。
* 用 Next.js 前端连接钱包，调用合约，完成 Mint 流程。
* 可以上传 NFT 元数据及图片至 IPFS，确保 tokenURI 有效。

---

## 五、常见问题及解决方案

| 问题                         | 解决方案                                            |
| -------------------------- | ----------------------------------------------- |
| `gluegun` 依赖 GitHub 仓库找不到  | 在 package.json 用 resolutions 强制指定版本，或清除缓存重新安装   |
| Hardhat node 版本不兼容         | 切换 Node 版本到 14/16/18                            |
| yarn add 报错 workspace root | 使用 `yarn workspace <name> add <package>` 指定包内安装 |
| PowerShell 删除文件夹命令错误       | 使用 `Remove-Item -Recurse -Force <folder>`       |
| 合约部署 RPC 配置                | 使用 Infura/Alchemy 提供的测试网 RPC URL 和私钥配置在 .env    |

# 2025-08-07

## **NFT Mint Demo 流程概念版**

---

### **1. 初始化开发环境**

* 选择 **Hardhat** 作为开发框架，用于编译、部署和测试智能合约。
* 安装 **OpenZeppelin** 合约库，避免从零写标准 ERC-721 逻辑。

---

### **2. 编写 NFT 智能合约**

* 使用 Solidity 创建合约，继承 **ERC721URIStorage**（用于支持 NFT 元数据 URI）。
* 添加一个 `mintNFT` 函数，允许合约所有者铸造新的 NFT。
* 每个 NFT 绑定一个 `tokenURI`（通常指向 IPFS 上的 JSON 文件，描述图片和属性）。

---

### **3. 上传 NFT 资源**

* NFT 图片和元数据不能直接放链上（成本高），通常上传到 **IPFS**。
* **IPFS 文件结构**：

  * 图片文件 → 生成 IPFS 哈希。
  * JSON 元数据文件（包含图片链接） → 生成 IPFS 哈希。

---

### **4. 部署合约到测试网**

* 使用 **Hardhat 脚本**调用部署逻辑，将合约部署到测试网（如 Sepolia）。
* 需要配置测试网 RPC（Alchemy/Infura）+ 部署者钱包私钥。

---

### **5. 前端交互（DApp）**

* 用 **React + Wagmi + RainbowKit** 搭建前端。
* 功能：

  * 连接钱包（MetaMask）。
  * 点击按钮 → 调用合约的 `mintNFT` 函数 → 发送交易。

---

### **6. 验证结果**

* 在 **Etherscan 测试网**查看交易成功。
* 在 **OpenSea Testnet**（或类似平台）查看 NFT 是否显示。

---

## **核心知识点**

* **Hardhat**：智能合约开发框架（编译、部署、测试）。
* **OpenZeppelin**：安全、标准的合约库（ERC-20、ERC-721、ERC-1155）。
* **ERC-721**：NFT 标准（每个 token 独一无二）。
* **IPFS**：分布式文件存储，NFT 元数据常用方案。
* **前端交互**：通过 wagmi（Web3 Hooks）+ RainbowKit（钱包 UI）连接以太坊。

# 2025-08-06

## 使用 Scaffold-ETH 2 部署 NFT 项目

### **今日目标**

* 掌握 Scaffold-ETH 2 快速搭建和部署 Web3 应用的流程
* 在以太坊测试网（Sepolia）部署 NFT 智能合约
* 将前端 DApp 部署到 Vercel，生成可访问的公开链接

---

### ** 核心学习内容**

#### **1. Scaffold-ETH 2 的作用**

Scaffold-ETH 2 是一个 Web3 全栈开发框架，集成了：

* **智能合约开发（Hardhat）**
* **前端框架（Next.js）**
* **自动合约接口生成**
* **钱包集成和测试工具**

它的核心优势是让开发者 **从编写合约到部署前端一站式完成**。

---

#### **2. 项目部署流程**

1. **初始化 Scaffold-ETH 项目**

   * 自动生成 `hardhat` 和 `nextjs` 两个包。
   * 前端和合约集成度高，减少手动配置。

2. **生成部署钱包**

   * 使用 `yarn generate` 创建新钱包，并设置密码加密。
   * 支持导入到 MetaMask，或使用 Scaffold-ETH 的 Burner Wallet。

3. **配置 Sepolia 测试网**

   * 在 `hardhat.config.ts` 配置 RPC URL 和私钥。
   * Sepolia 是以太坊官方测试网，适合部署练习。

4. **部署智能合约**

   ```bash
   yarn deploy --network sepolia
   ```

   部署完成后输出合约地址，并同步到前端。

   本次部署的合约地址：

   ```
   0x297fFCAf15eAAB709b875ad74a7ED36B4591F895
   ```

   可以在 [Sepolia Etherscan](https://sepolia.etherscan.io/) 查询。

5. **启动前端应用**

   * 前端自动集成合约信息，支持连接钱包。
   * 本地调试：

     ```bash
     yarn start
     ```

6. **部署前端到 Vercel**

   * 使用 `yarn vercel` 进行上线，生成可访问的 URL。

   本次项目前端地址：
   [https://nft-deployment.vercel.app/](https://nft-deployment.vercel.app/)

---

#### **3. 学到的知识**

* Scaffold-ETH 2 提供一体化开发体验，省去了 ABI 地址手动绑定的繁琐操作。
* 明确了 **Web3 项目标准流程**：钱包 → 测试网 → 合约部署 → 前端集成 → 上线。
* 学会使用 Vercel 快速上线 Next.js DApp。
* 知道了 API Key 的作用（Alchemy RPC、Etherscan 验证）。

# 2025-08-05

学习了安全相关知识，对面试出现的恶意软件保持警惕，因最近案列太多。以前有关注漫雾后面还会持续关注。合约部署研究中。

# 2025-08-04

完成了全部web3入门导读，加入LXDAO Discord, NFT mint仍然有问题，但整个流程已经读完。


# 2025.07.30


<!-- Content_END -->
