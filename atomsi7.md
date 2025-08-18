---
timezone: UTC+8
---

# Link

**GitHub ID:** atomsi7

**Telegram:** @L1nk_ye

## Self-introduction

web3 beginner

## Notes

<!-- Content_START -->
# 2025-08-19

### Dapp开发
完整Dapp开发，前端后端合约测试。

# 2025-08-18

### 学习Dapp开发
开发ERC20代码合约

# 2025-08-15

###学习加密笔记
浏览椭圆曲线加密相关内容
- 陷门函数：
  一种在一个方向上很容易计算，但在没有特殊信息的情况下很难在相反方向上计算（寻找它的逆）的函数

# 2025-08-14

### Dapp
gas优化学习，

现代智能合约开发学习
   - 用组合而不是继承
   - 用soldeer管理包

优化Dapp逻辑。

# 2025-08-13

### Dapp开发
已完成大致功能的合约和前端开发。
合约逻辑和前端逻辑UI需要调整。

Vibe Coding也需要了解基础的技术栈和组件应用。

# 2025-08-12

Dapp 与 传统应用 不同

纯粹的去中性化app不多，web3 + web2 融合

存储类型和Gas消耗：

- 写存储十分昂贵

Gas优化

## 前端

```mermaid
flowchart TD
    A[前端界面] --> B[RainbowKit]
    A --> C[wagmi Hook]
    B --> D[用户钱包]
    C --> E[合约读取/写入]
    D --> F[以太坊网络]
    E --> F
    F --> G[智能合约]
```

**RPC节点配置**

重要概念：DApp 和用户钱包使用不同的 RPC 节点

- 合约读取：走 DApp 配置的 RP9（如下面的 http（））
- 交易签名/广播：走钱包的 RPC（用户在 MetaMask 中配置的节点）

读取合约用自己的RPC网络。

# 2025-08-11

### Dapp搭建
添加前端对于本地网络的测试。
美化前端，适配不同主题。

# 2025-08-10

### 编写front-end。
使用`react + vite + wagmi` 编写前端与合约交互。

# 2025-08-08

### 智能合约开发
- 学习[智能合约开发](https://web3intern.xyz/zh/smart-contract-development/)
- 部署智能合约到测试网上。
  - [合约地址](https://sepolia.etherscan.io/address/0x6b2a95b7fabab49c8ad109535e66a0e4449d31f9)

### 智能合约部署过程（Remix)
Deploy时Environment选择 **Injected Provider - MetaMask**，连接钱包，直接部署。

### 智能合约本地开发和部署
1. 使用`foundry`，生成好环境
2. 编写智能合约
3. `anvil` 启动以太节点
4. `forge script` 部署合约
5. `cast`与合约交互

# 2025-08-07

##  **技术岗**

- **前端工程师职责**：
    
    - 构建 DApp UI，集成钱包与智能合约交互；
        
    - 使用 React/Vue，处理链上/链下数据显示；
        
    - 保持界面高性能与兼容性。
        
    
- **后端工程师职责**：
    
    - 搭建链下服务，提供 API 支持；
        
    - 管理链上数据同步与交易流程；
        
    - 保障系统性能与安全。
        
    
- **智能合约工程师职责**：
    
    - 使用 Solidity 编写与部署智能合约；
        
    - 涉及 NFT、DeFi、DAO 等核心模块；
        
    - 编写测试，进行安全审计与 Gas 优化；
        
    - 与前后端团队协同开发与集成。
        
    
- **岗位通用要求**：
    
    - 扎实的计算机基础，熟练使用主流开发语言；
        
    - 熟悉生态；
        
    - 熟悉 Web3 工具链（Hardhat、Foundry、Viem、Ethers.js）；
        
    - 有实际项目经验、良好代码规范与团队协作能力。

## Web3安全与合规笔记

**法律合规风险**：中国禁止ICO、虚拟货币交易所运营，虚拟货币不具法偿性。Web3项目易涉及非法集资、传销、洗钱、赌博等刑事风险。技术人员参与代币设计、合约部署也可能被追究共同责任。

**入职风险防范**：Web3企业多采用境外注册，难以签订有效劳动合同和缴纳社保。薪酬常以"人民币+Token"或"全USDT"形式发放，存在法律与税务风险。虚拟货币出金易卷入刑事风险，可能导致银行卡冻结。

**网络安全威胁**：常见攻击包括钓鱼攻击（伪造邮件/网站）、恶意软件木马、社交工程攻击、剪贴板劫持等。防护措施包括：只用官方会议工具、启用双因素认证、使用密码管理器、转账前核对地址、助记词离线保存。

**典型案例警示**：即使是技术开发人员，也难以"只是写代码"为由免责。

# 2025-08-05

*Learn the Basic of the Blockchain.*

# Blockchain
## Bitcoin Revolution
Money -- Value -- Trust
- Fiat Money Problems: Centralized control, unlimited quantity (QE), government dependency
- Digital Money Challenge: Double-spending problem - digital files can be duplicated
- Bitcoin Solution (2008): Satoshi Nakamoto's peer-to-peer electronic cash system solving double-spending without central authority

## Blockchain Primer - Core Concepts

- **Hash Functions**: Convert variable-length input to fixed-size output (one-way, hard to invert)
- **Hash Pointers**: Point to data location + cryptographic hash for verification
- **Blockchain Structure**: Linked blocks using hash pointers, creating tamper-evident log
- **Digital Signatures**: Verify authenticity using public/private key cryptography

## How Blockchain Works

- **Proof of Work (PoW)**: Miners compete to solve mathematical puzzles to add blocks
- **Mining Process**: Find nonce value that produces hash meeting difficulty target
- **Consensus**: Network validates and accepts longest valid chain
- **Decentralized Network**: P2P system where all nodes maintain copy of ledger

# 2025-08-04

**Web3Intern**

- 学习[Web3 工作方式](https://web3intern.xyz/zh/remote-work-guide)

**Cryptography**

- 学习基础的加密算法知识

# 2025.07.31


<!-- Content_END -->
