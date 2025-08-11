---
timezone: UTC+8
---

# mX1@0

**GitHub ID:** xmhhmx

**Telegram:** @hhh xxx

## Self-introduction

一起加油

## Notes

<!-- Content_START -->
# 2025-08-11

| 今日学习内容                              |
| ----------------------------------------- |
| Chainlink预言机的solidity进阶课程部分内容 |
| 智能合约开发部分内容                      |



### Chainlink预言机的solidity进阶课程

#### 预言机网络（Chainlink）

![](https://cdn.jsdelivr.net/gh/xmhhmx/PicGoCDN//img/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-08-11%20123802.png)

**Chainlink** 是一个 **去中心化的预言机网络（Decentralized Oracle Network）**，主要解决区块链无法直接获取外部数据的问题。图中展示的是其如何为 **现代Web3应用** 提供多链、多服务的支持。

（1）Chainlink Web3 服务

| **服务类型**    | **功能**                       | **具体产品**                                                 |
| :-------------- | :----------------------------- | :----------------------------------------------------------- |
| **Data**        | 提供链外数据（如价格、天气等） | • Feeds（价格预言机） • Functions（链上请求API） • Data Stream（实时数据流） |
| **Compute**     | 提供可验证的随机数和自动化执行 | • VRF（可验证随机数） • Automation（智能合约自动化）         |
| **Cross-chain** | 跨链通信和数据传输             | CCIP（跨链互操作协议）                                       |

（2）连接对象

- **区块链（Blockchain）**：支持多链（图中标注 1.1/1.2 可能指不同链版本或主网/测试网）。
- **合约与资产（Contracts & Assets）**：如DeFi协议、NFT项目等。
- **Web2系统**：传统企业数据、API服务和遗留系统（如银行支付网关）。



#### 喂价（Data Feed）

##### Data Feed 架构

![](https://cdn.jsdelivr.net/gh/xmhhmx/PicGoCDN//img/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-08-11%20124451.png)

DON：去中心化的预言机网络

DON从数据提供商那里获取数据，将数据在DON网络中进行一次聚合，聚合之后将数据写入链上所部署的ChainLink Data Feed 合约中，用户合约可以通过Data Feed 合约的地址进行调用，从而获取到当前代币的价格



##### Data Feed 结构

![](https://cdn.jsdelivr.net/gh/xmhhmx/PicGoCDN//img/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-08-11%20124904.png)

Aggregator：聚合合约，用于收集链下的数据，将其存入到transmission中

通过查看chainlink文档中，Data Feed合约内容，用户可以通过调用合约中的getChainlinkDataFeedLatestAnswer函数来获取代币实时数据，合约会返回一个answer值，就是链下代币值。

<img src="https://cdn.jsdelivr.net/gh/xmhhmx/PicGoCDN//img/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-08-11%20125811.png" style="zoom:67%;" />



#### 部署一个FundMe的智能合约

需求：

1. 创建一个收款函数
1. 记录投资人并且查看
1. 在锁定期内，达到目标值，生产商可以提款
1. 在锁定期内，未达到目标值，投资人可以退款

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;
import {AggregatorV3Interface} from "@chainlink/contracts/src/v0.8/shared/interfaces/AggregatorV3Interface.sol";

//1.创建一个收款函数
//2.记录投资人并且查看 
//3.在锁定期内，达到目标值，生产商可以提款
//4.在锁定期内，未达到目标值，投资人可以退款

contract FundMe {
    mapping (address => uint256) public fundersToAmount;

    uint256 constant MINIMUM_VALUE = 100 * 10 ** 18;   //USD

    AggregatorV3Interface internal dataFeed;

    uint256 constant TARGET = 1000 * 10 ** 18;

    address public owner;

    constructor(){
        dataFeed = AggregatorV3Interface(0x694AA1769357215DE4FAC081bf1f309aDC325306);
        owner = msg.sender;
    }


    function fund() external  payable {
        require(convertEthToUsd(msg.value) >= MINIMUM_VALUE, "Send more ETH");
        fundersToAmount[msg.sender] = msg.value;
    }

    function getChainlinkDataFeedLatestAnswer() public view returns (int) {
        // prettier-ignore
        (
            //uint80 roundId,
            int answer,
            //uint256 startedAt,
            //uint256 updatedAt,
            //uint80 answeredInRound
        ) = dataFeed.latestRoundData();
        return answer;
    }

    function convertEthToUsd(uint256 ethAmount) internal view returns(uint256){
        uint256 ethPrice = uint256(getChainlinkDataFeedLatestAnswer());
        return ethAmount * ethPrice / (10**8);
    }

    function getFund() external {
        require(convertEthToUsd(address(this).balance) >= TARGET,"Target is not reached");
        require(msg.sender == owner, "this funciton can only be called by owner");
        //transfer
        // payable(msg.sender).transfer(address(this).balance);

        //send
        // bool success = payable(msg.sender).send(address(this).balance);
        // require(success, "tx failed");

        //call 
        bool success;
        (success, ) = payable(msg.sender).call{value: address(this).balance}("");
        require(success ,"transfer tx failed");
    }

    function transformOwnerShip(address newOwner) public {
        require(msg.sender == owner, "this funciton can only be called by owner");
    }

    function refund() external {
        require(convertEthToUsd(address(this).balance) < TARGET,"Target is reached");
        require(fundersToAmount[msg.sender] != 0, "there is no fund for you");
        bool success;
        (success, ) = payable(msg.sender).call{value: fundersToAmount[msg.sender]}("");
        require(success ,"transfer tx failed");
    }
}
```

payable：要有这个参数才能执行收款



创建一个构造函数，去chainlink中找到价格信息来源地址

```solidity
constructor(){
		//sepolia testnet
        dataFeed = AggregatorV3Interface(0x694AA1769357215DE4FAC081bf1f309aDC325306);
    }
```

![](https://cdn.jsdelivr.net/gh/xmhhmx/PicGoCDN//img/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-08-11%20154641.png)



写一个新的函数用于转换ETH和USD

```solidity
 function fund() external  payable {
        require(convertEthToUsd(msg.value) >= MINIMUM_VALUE, "Send more ETH");	//如果不满足执行回退，返回"Send more ETH"
        fundersToAmount[msg.sender] = msg.value;
    }

function convertEthToUsd(uint256 ethAmount) internal view returns(uint256){
        uint256 ethPrice = uint256(getChainlinkDataFeedLatestAnswer());
        return ethAmount * ethPrice / (10**8);
        // ETH / USD    precision= 10 ** 8
        // X / ETH 		precision= 10 ** 18
    }
```

注意：getChainlinkDataFeedLatestAnswer()获得的值是一个int类型的，需要强制转换为uint256类型



##### 转账的三种类型

1. transfer：失败则直接回退

```solidity
结构：addr.transfer(value)
payable(msg.sender).transfer(address(this).balance);
```

| 代码片段                | 作用                                                         |
| :---------------------- | :----------------------------------------------------------- |
| `address(this).balance` | 获取当前合约地址持有的 ETH 余额（单位：wei，1 ETH = 10¹⁸ wei） |
| `payable(msg.sender)`   | 将调用者地址转换为可接收 ETH 的 `payable` 类型               |
| `.transfer(...)`        | 向目标地址发送 ETH（自动处理 wei 单位，失败时回滚交易）      |

2. send：失败返回false布尔值

```solidity
结构：addr.send(value)
bool success = payable(msg.sender).send(address(this).balance);
require(success, "tx failed");
```

3. call：可以在转账的同时带上数据，返回函数返回值以及布尔值

```
结构：(bool ,result) = addr.call{value: value}("")
bool success;
(success, ) = payable(msg.sender).call{value: address(this).balance}("");
没有调用函数 所以""内为空，也没有函数返回值，result可以制空
```

 

##### 提款

```solidity
function refund() external {
        require(convertEthToUsd(address(this).balance) < TARGET,"Target is reached");
        require(fundersToAmount[msg.sender] != 0, "there is no fund for you");
        bool success;
        (success, ) = payable(msg.sender).call{value: fundersToAmount[msg.sender]}("");
        require(success ,"transfer tx failed");
    }
```

这个函数存在一个bug，就是一个人将自己的存款提出后，未对该账户进行清零，导致这个人可以继续refund，提取别人的存款

改进，加一句提款后清零账户，fundersToAmount[msg.sender] = 0

```solidity
function refund() external {
        require(convertEthToUsd(address(this).balance) < TARGET,"Target is reached");
        require(fundersToAmount[msg.sender] != 0, "there is no fund for you");
        bool success;
        (success, ) = payable(msg.sender).call{value: fundersToAmount[msg.sender]}("");
        require(success ,"transfer tx failed");
        fundersToAmount[msg.sender] = 0		//
    }
```



### 智能合约开发

#### Dapp 架构和开发流程

Dapp：去中心化应用，运行在区块链或分布式网络上，应用的逻辑和数据是由多个参与者共同维护。

##### Dapp 架构

三个核心部分：

1. **前端（User Interface）**：

- 前端是 Dapp 与用户交互的界面，通常由 HTML、CSS 和 JavaScript（如 React、Vue 等框架）构建。与传统 Web 应用不同，Dapp 前端会连接区块链来调用智能合约，呈现数据和执行交易。
- 前端还需要集成区块链钱包（如 MetaMask）来进行身份验证和签署交易，确保用户的隐私和安全。

2. **智能合约（Smart Contracts）**：

- 智能合约是 Dapp 的核心，它定义了应用的业务逻辑，并部署在区块链上。智能合约通过执行自动化的规则来确保交易和操作的透明性与不可篡改性。
- 在以太坊平台上，智能合约通常使用 **Solidity** 编程语言编写，并通过 **Ethereum Virtual Machine (EVM)** 执行。

3. **数据检索器（Indexer）**：

- 智能合约通常以 `Event` 形式释放日志事件，比如释放代表 NFT 转移的 `Transfer` 事件，数据检索器会检索这些数据并将其写入到 PostgreSQL 等传统数据库中
- Dapp 在前端进行数据展示时需要检索器内的数据。一个简单的示例是某 NFT 项目需要展示用户持有的所有 NFT，但是 NFT 合约并不会提供通过输入地址参数返回该地址下的所有 NFT 的函数，此时我们可以运行数据检索器将 `Transfer` 事件读取后写入传统数据库内，前端可以在传统数据库内检索用户持有的 NFT 数据

4. **区块链和去中心化存储（Blockchain & Decentralized Storage）**：

- 区块链用于存储智能合约的状态数据及交易记录。去中心化存储如 **IPFS**（InterPlanetary File System）或 **Arweave**，用于存储大规模的非结构化数据（如图片、文档等），确保数据不易丢失和篡改。
- 通过使用去中心化存储，Dapp 确保所有数据在多个节点上备份，保证数据的持久性和去中心化特性。



##### Dapp 开发流程

1. **需求分析与规划**

	在开发 Dapp 之前，首先需要进行需求分析和规划，明确应用的目标和功能。此阶段包括：

	- **确定功能需求**：需要定义用户可以进行的操作，比如转账、查询余额、创建投票等。
	- **选择区块链平台**：决定在哪个平台上构建 Dapp（如以太坊、Solana、Polygon 等），这通常取决于目标用户群、交易成本、可扩展性等因素。
	- **设计用户体验（UX）**：定义 Dapp 的界面设计和交互流程，确保用户能够轻松使用应用并与区块链交互。

1. **智能合约开发**

	智能合约是 Dapp 的核心，负责执行去中心化的业务逻辑和存储重要的数据。在这一阶段，开发者需要：

	- **编写智能合约**：使用 **Solidity** 或其他智能合约语言编写合约，确保合约的功能满足需求分析中定义的要求。
	- **编写测试用例**：为智能合约编写单元测试，确保合约逻辑正确、无漏洞。
	- **审计和优化**：对合约进行安全审计，确保合约的安全性，避免常见漏洞（如重入攻击、整数溢出等）。

1. **检索器开发**

	检索器是<u>**获取链上数据的核心**</u>，<u>**负责捕获智能合约释放的事件并以合理的方式将其存入数据库**</u>的不同的表内部。在这一阶段，开发者需要:

	- **确定功能需要的数据内容**: 前端使用的数据大部份都直接来自检索器，所以开发者需要确定前端工程师所需要的数据
	- **编写检索器程序**: 目前主流的检索器框架，如 ponder 和 subgraph 都是用了 TypeScript 语言作为检索器的程序编写语言，开发者主要编写事件数据清理以及事件数据写入数据库的代码
	- **部署和运维**: 编写程序完成后，一般使用 Docker 部署到云服务器中，当然目前很多检索器框架也提供 SaaS 服务，同时检索器作为一个常规的数据库应用需要运维

1. **前端开发**

	前端是用户与 Dapp 交互的主要界面，因此开发前端时需要：

	- **选择前端框架**：可以使用现代前端框架（如 **React**、**Vue**）来构建 UI。前端将通过 JavaScript 与智能合约进行交互。
	- **连接钱包**：通过集成 **MetaMask** 等 Web3 钱包，用户可以连接到 Dapp，并授权其与智能合约交互。
	- **显示区块链数据**：前端需要从区块链和检索器内获取数据（如账户余额、交易记录），并通过用户界面展示。
	- **处理交易签名与确认**：当用户发起交易时，前端需要与钱包进行交互，获取用户的签名并将交易发送到区块链。

1. **与区块链交互**

	前端和智能合约通过 **Viem**（推荐）、**Ethers.js** 或 **Wagmi** 等现代化库进行交互。这些库提供更好的 TypeScript 支持和性能优化：

	- **读取数据**：前端通过智能合约的公共函数读取区块链上的状态数据（如余额、合约信息）。
	- **发送交易**：当用户发起交易时，前端需要通过钱包签署交易并发送到区块链，执行合约中的某个功能（如转账）。

1. **部署与上线**

	一旦开发完成，Dapp 进入部署阶段。具体步骤包括：

	- **部署智能合约**：推荐使用 **Hardhat** 或 **Foundry**（现代化开发工具）将智能合约部署到测试网（如 **Sepolia**、**Holesky**）或主网。
	- **前端部署**：将前端应用部署到去中心化平台（如 **IPFS**）或传统的 Web 服务（Vercel）。
	- **发布和维护**：将 Dapp 上线，进行用户反馈收集，定期更新合约和前端，修复潜在问题。

# 2025-08-10

| 今日学习内容                                                 |
| ------------------------------------------------------------ |
| CryptoZombiesx的solidity学习/Solidity: Beginner to Intermediate Smart Contracts/lesson2 僵尸攻击人类 |



## CryptoZombiesx的solidity学习

### Solidity: Beginner to Intermediate Smart Contracts

### lesson2 僵尸攻击人类

### 第2章: 映射（Mapping）和地址（Address）

我们通过给数据库中的僵尸指定“主人”， 来支持“多玩家”模式。

如此一来，我们需要引入2个新的数据类型：`mapping`（映射） 和 `address`（地址）。

**Addresses （地址）**

以太坊区块链由 **_ account _** (账户)组成，你可以把它想象成银行账户。一个帐户的余额是 **_以太_** （在以太坊区块链上使用的币种），你可以和其他帐户之间支付和接受以太币，就像你的银行帐户可以电汇资金到其他银行帐户一样。

每个帐户都有一个“地址”，你可以把它想象成银行账号。这是账户唯一的标识符，它看起来长这样：

```solidity
0x0cE446255506E92DF41614C46F1d6df9Cc969183
```

（这是 CryptoZombies 团队的地址，如果你喜欢 CryptoZombies 的话，请打赏我们一些以太币！😉）

我们将在后面的课程中介绍地址的细节，现在你只需要了解**地址属于特定用户（或智能合约）的**。

所以我们可以指定“地址”作为僵尸主人的 ID。当用户通过与我们的应用程序交互来创建新的僵尸时，新僵尸的所有权被设置到调用者的以太坊地址下。

**Mapping（映射）**

在第1课中，我们看到了 **结构体** 和 **数组** 。 **映射** 是另一种在 Solidity 中存储有组织数据的方法。

映射是这样定义的：

```solidity
//对于金融应用程序，将用户的余额保存在一个 uint类型的变量中：
mapping (address => uint) public accountBalance;
//或者可以用来通过userId 存储/查找的用户名
mapping (uint => string) userIdToName;
```

映射本质上是存储和查找数据所用的**键-值对**。在第一个例子中，键是一个 `address`，值是一个 `uint`，在第二个例子中，键是一个`uint`，值是一个 `string`。

**实战演习**

为了存储僵尸的所有权，我们会使用到两个映射：一个记录僵尸拥有者的地址，另一个记录某地址所拥有僵尸的数量。

1.创建一个叫做 `zombieToOwner` 的映射。其键是一个`uint`（我们将根据它的 id 存储和查找僵尸），值为 `address`。映射属性为`public`。

2.创建一个名为 `ownerZombieCount` 的映射，其中键是 `address`，值是 `uint`。

```solidity
    // 在这里定义映射
    mapping (uint => address ) public zombieToOwner;
    mapping ( address => uint) ownerZombieCount;
```



### 第3章: Msg.sender

现在有了一套映射来记录僵尸的所有权了，我们可以修改 `_createZombie` 方法来运用它们。

为了做到这一点，我们要用到 `msg.sender`。

**msg.sender**

在 Solidity 中，有一些全局变量可以被所有函数调用。 其中一个就是 `msg.sender`，它指的是当前调用者（或智能合约）的 `address`。

> 注意：在 Solidity 中，功能执行始终需要从外部调用者开始。 一个合约只会在区块链上什么也不做，除非有人调用其中的函数。所以 `msg.sender`总是存在的。

以下是使用 `msg.sender` 来更新 `mapping` 的例子：

```solidity
mapping (address => uint) favoriteNumber;

function setMyNumber(uint _myNumber) public {
  // 更新我们的 `favoriteNumber` 映射来将 `_myNumber`存储在 `msg.sender`名下
  favoriteNumber[msg.sender] = _myNumber;
  // 存储数据至映射的方法和将数据存储在数组相似
}

function whatIsMyNumber() public view returns (uint) {
  // 拿到存储在调用者地址名下的值
  // 若调用者还没调用 setMyNumber， 则值为 `0`
  return favoriteNumber[msg.sender];
}
```

在这个小小的例子中，任何人都可以调用 `setMyNumber` 在我们的合约中存下一个 `uint` 并且与他们的地址相绑定。 然后，他们调用 `whatIsMyNumber` 就会返回他们存储的 `uint`。

使用 `msg.sender` 很安全，因为它具有以太坊区块链的安全保障 —— 除非窃取与以太坊地址相关联的私钥，否则是没有办法修改其他人的数据的。

**实战演习**

我们来修改第1课的 `_createZombie` 方法，将僵尸分配给函数调用者吧。

1. 首先，在得到新的僵尸 `id` 后，更新 `zombieToOwner` 映射，在 `id` 下面存入 `msg.sender`。
1. 然后，我们为这个 `msg.sender` 名下的 `ownerZombieCount` 加 1。

跟在 JavaScript 中一样， 在 Solidity 中你也可以用 `++` 使 `uint` 递增。

```solidity
uint number = 0;
number++;
// `number` 现在是 `1`了
```

修改两行代码即可。

```solidity
function _createZombie(string _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        // 从这里开始
        zombieToOwner[id] = msg.sender;
        ownerZombieCount[msg.sender]++;
        NewZombie(id, _name, _dna);
    }
```



### 第4章: Require

在第一课中，我们成功让用户通过调用 `createRandomZombie`函数 并输入一个名字来创建新的僵尸。 但是，如果用户能持续调用这个函数来创建出无限多个僵尸加入他们的军团，这游戏就太没意思了！

于是，我们作出限定：每个玩家只能调用一次这个函数。 这样一来，新玩家可以在刚开始玩游戏时通过调用它，为其军团创建初始僵尸。

我们怎样才能限定每个玩家只调用一次这个函数呢？

答案是使用`require`。 `require`使得函数在执行过程中，当不满足某些条件时抛出错误，并停止执行：

```solidity
function sayHiToVitalik(string _name) public returns (string) {
  // 比较 _name 是否等于 "Vitalik". 如果不成立，抛出异常并终止程序
  // (敲黑板: Solidity 并不支持原生的字符串比较, 我们只能通过比较
  // 两字符串的 keccak256 哈希值来进行判断)
  require(keccak256(_name) == keccak256("Vitalik"));
  // 如果返回 true, 运行如下语句
  return "Hi!";
}
```

如果你这样调用函数 `sayHiToVitalik（“Vitalik”）` ,它会返回“Hi！”。而如果调用的时候使用了其他参数，它则会抛出错误并停止执行。

因此，在调用一个函数之前，用 `require` 验证前置条件是非常有必要的。

**实战演习**

在我们的僵尸游戏中，我们不希望用户通过反复调用 `createRandomZombie` 来給他们的军队创建无限多个僵尸 —— 这将使得游戏非常无聊。

我们使用了 `require` 来确保这个函数只有在每个用户第一次调用它的时候执行，用以创建初始僵尸。

1. 在 `createRandomZombie` 的前面放置 `require` 语句。 使得函数先检查 `ownerZombieCount [msg.sender]` 的值为 `0` ，不然就抛出一个错误。

> 注意：在 Solidity 中，关键词放置的顺序并不重要
>
> - 虽然参数的两个位置是等效的。 但是，由于我们的答案检查器比较呆板，它只能认定其中一个为正确答案
> - 于是在这里，我们就约定把`ownerZombieCount [msg.sender]`放前面吧

```solidity
function createRandomZombie(string _name) public {
        // start here
        require(ownerZombieCount[msg.sender] == 0);
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }
```



### 第5章: 继承（Inheritance）

我们的游戏代码越来越长。 当代码过于冗长的时候，最好将代码和逻辑分拆到多个不同的合约中，以便于管理。

有个让 Solidity 的代码易于管理的功能，就是合约 ***inheritance\*** (继承)：

```solidity
contract Doge {
  function catchphrase() public returns (string) {
    return "So Wow CryptoDoge";
  }
}

contract BabyDoge is Doge {
  function anotherCatchphrase() public returns (string) {
    return "Such Moon BabyDoge";
  }
}
```

由于 `BabyDoge` 是从 `Doge` 那里 ***inherits\*** （继承)过来的。 这意味着当你编译和部署了 `BabyDoge`，它将可以访问 `catchphrase()` 和 `anotherCatchphrase()`和其他我们在 `Doge` 中定义的其他公共函数。

这可以用于逻辑继承（比如表达子类的时候，`Cat` 是一种 `Animal`）。 但也可以简单地将类似的逻辑组合到不同的合约中以组织代码。

**实战演习**

在接下来的章节中，我们将要为僵尸实现各种功能，让它可以“猎食”和“繁殖”。 通过将这些运算放到父类 `ZombieFactory` 中，使得所有 `ZombieFactory` 的继承者合约都可以使用这些方法。

1. 在 `ZombieFactory` 下创建一个叫 `ZombieFeeding` 的合约，它是继承自 `ZombieFactory 合约的。

```solidity
// Start here
contract ZombieFeeding is ZombieFactory {
    
}
```



### 第6章: 引入（Import）

哇！你有没有注意到，我们只是清理了下右边的代码，现在你的编辑器的顶部就多了个选项卡。 尝试点击它的标签，看看会发生什么吧！

代码已经够长了，我们把它分成多个文件以便于管理。 通常情况下，当 Solidity 项目中的代码太长的时候我们就是这么做的。

在 Solidity 中，当你有多个文件并且想把一个文件导入另一个文件时，可以使用 `import` 语句：

```solidity
import "./someothercontract.sol";

contract newContract is SomeOtherContract {

}
```

这样当我们在合约（contract）目录下有一个名为 `someothercontract.sol` 的文件（ `./` 就是同一目录的意思），它就会被编译器导入。

**实战演习**

现在我们已经建立了一个多文件架构，并用 `import` 来读取来自另一个文件中合约的内容：

1.将 `zombiefactory.sol` 导入到我们的新文件 `zombiefeeding.sol` 中。

```solidity
// put import statement here
import "./zombiefactory.sol";
```



### 第7章: Storage与Memory

在 Solidity 中，有两个地方可以存储变量 —— `storage` 或 `memory`。

**Storage** 变量是指**<u>永久存储</u>**在区块链中的变量。 **Memory** 变量则是**<u>临时的</u>**，当外部函数对某合约调用完成时，内存型变量即被移除。 你可以把它想象成存储在你电脑的硬盘或是RAM中数据的关系。

大多数时候你都用不到这些关键字，默认情况下 Solidity 会自动处理它们。 状态变量（在函数之外声明的变量）默认为“存储”形式，并永久写入区块链；而在函数内部声明的变量是“内存”型的，它们函数调用结束后消失。

然而也有一些情况下，你需要手动声明存储类型，主要用于处理函数内的 **结构体** 和 **数组** 时：

```solidity
contract SandwichFactory {
  struct Sandwich {
    string name;
    string status;
  }

  Sandwich[] sandwiches;

  function eatSandwich(uint _index) public {
    // Sandwich mySandwich = sandwiches[_index];

    // ^ 看上去很直接，不过 Solidity 将会给出警告
    // 告诉你应该明确在这里定义 `storage` 或者 `memory`。

    // 所以你应该明确定义 `storage`:
    Sandwich storage mySandwich = sandwiches[_index];
    // ...这样 `mySandwich` 是指向 `sandwiches[_index]`的指针
    // 在存储里，另外...
    mySandwich.status = "Eaten!";
    // ...这将永久把 `sandwiches[_index]` 变为区块链上的存储

    // 如果你只想要一个副本，可以使用`memory`:
    Sandwich memory anotherSandwich = sandwiches[_index + 1];
    // ...这样 `anotherSandwich` 就仅仅是一个内存里的副本了
    // 另外
    anotherSandwich.status = "Eaten!";
    // ...将仅仅修改临时变量，对 `sandwiches[_index + 1]` 没有任何影响
    // 不过你可以这样做:
    sandwiches[_index + 1] = anotherSandwich;
    // ...如果你想把副本的改动保存回区块链存储
  }
}
```

如果你还没有完全理解究竟应该使用哪一个，也不用担心 —— 在本教程中，我们将告诉你何时使用 `storage` 或是 `memory`，并且当你不得不使用到这些关键字的时候，Solidity 编译器也发警示提醒你的。

现在，只要知道在某些场合下也需要你显式地声明 `storage` 或 `memory`就够了！

**实战演习**

是时候给我们的僵尸增加“猎食”和“繁殖”功能了！

当一个僵尸猎食其他生物体时，它自身的DNA将与猎物生物的DNA结合在一起，形成一个新的僵尸DNA。

1. 创建一个名为 `feedAndMultiply` 的函数。 使用两个参数：`_zombieId`（ `uint`类型 ）和`_targetDna` （也是 `uint` 类型）。 设置属性为 `public` 的。
1. 我们不希望别人用我们的僵尸去捕猎。 首先，我们确保对自己僵尸的所有权。 通过添加一个`require` 语句来确保 `msg.sender` 只能是这个僵尸的主人（类似于我们在 `createRandomZombie` 函数中做过的那样）。

> 注意：同样，因为我们的答案检查器比较呆萌，只认识把 `msg.sender` 放在前面的答案，如果你切换了参数的顺序，它就不认得了。 但你正常编码时，如何安排参数顺序都是正确的。

1. 为了获取这个僵尸的DNA，我们的函数需要声明一个名为 `myZombie` 数据类型为`Zombie`的本地变量（这是一个 `storage` 型的指针）。 将其值设定为在 `zombies` 数组中索引为`_zombieId`所指向的值。

到目前为止，包括函数结束符 `}` 的那一行， 总共4行代码。

```solidity
// Start here
    function feedAndMultiply(uint _zombieId, uint _targetDna) public{
        require(msg.sender == zombieToOwner[_zombieId]);
        Zombie storage myZombie = zombies[_zombieId];
    }
```



### 第8章: 僵尸的DNA

我们来把 `feedAndMultiply` 函数写完吧。

获取新的僵尸DNA的公式很简单：计算猎食僵尸DNA和被猎僵尸DNA之间的平均值。

例如：

```solidity
function testDnaSplicing() public {
  uint zombieDna = 2222222222222222;
  uint targetDna = 4444444444444444;
  uint newZombieDna = (zombieDna + targetDna) / 2;
  // newZombieDna 将等于 3333333333333333
}
```

以后，我们也可以让函数变得更复杂些，比方给新的僵尸的 DNA 增加一些随机性之类的。但现在先从最简单的开始 —— 以后还可以回来完善它嘛。

**实战演习**

1. 首先我们确保 `_targetDna` 不长于16位。要做到这一点，我们可以设置 `_targetDna` 为 `_targetDna ％ dnaModulus` ，并且只取其最后16位数字。
1. 接下来为我们的函数声明一个名叫 `newDna` 的 `uint`类型的变量，并将其值设置为 `myZombie`的 DNA 和 `_targetDna` 的平均值（如上例所示）。

> 注意：您可以用 `myZombie.name` 或 `myZombie.dna` 访问 `myZombie` 的属性。

1. 一旦我们计算出新的DNA，再调用 `_createZombie` 就可以生成新的僵尸了。如果你忘了调用这个函数所需要的参数，可以查看 `zombiefactory.sol` 选项卡。请注意，需要先给它命名，所以现在我们把新的僵尸的名字设为`NoName` - 我们回头可以编写一个函数来更改僵尸的名字。

> 注意：对于 Solidity 高手，你可能会注意到我们的代码存在一个问题。别担心，下一章会解决这个问题的 ;）

```solidity
function feedAndMultiply(uint _zombieId, uint _targetDna) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    // start here
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    _createZombie("NoName", newDna);
  }
```



### 第9章: 更多关于函数可见性

**我们上一课的代码有问题！**

编译的时候编译器就会报错。

错误在于，我们尝试从 `ZombieFeeding` 中调用 `_createZombie` 函数，但 `_createZombie` 却是 `ZombieFactory` 的 `private` （私有）函数。这意味着任何继承自 `ZombieFactory` 的子合约都不能访问它。

**internal 和 external**

除 `public` 和 `private` 属性之外，Solidity 还使用了另外两个描述函数可见性的修饰词：`internal`（内部） 和 `external`（外部）。

`internal` 和 `private` 类似，不过， 如果某个合约继承自其父合约，这个合约即可以访问父合约中定义的“内部”函数。（嘿，这听起来正是我们想要的那样！）。

`external` 与`public` 类似，只不过这些函数只能在合约之外调用 - 它们不能被合约内的其他函数调用。稍后我们将讨论什么时候使用 `external` 和 `public`。

声明函数 `internal` 或 `external` 类型的语法，与声明 `private` 和 `public`类 型相同：

```solidity
contract Sandwich {
  uint private sandwichesEaten = 0;

  function eat() internal {
    sandwichesEaten++;
  }
}

contract BLT is Sandwich {
  uint private baconSandwichesEaten = 0;

  function eatWithBacon() public returns (string) {
    baconSandwichesEaten++;
    // 因为eat() 是internal 的，所以我们能在这里调用
    eat();
  }
}
```

**实战演习**

1. 将 `_createZombie()` 函数的属性从 `private` 改为 `internal` ， 使得其他的合约也能访问到它。

	我们已经成功把你的注意力集中在到`zombiefactory.sol`这个选项卡上啦。

```solidity
// 在这里修改函数的功能
    function _createZombie(string _name, uint _dna) internal {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        zombieToOwner[id] = msg.sender;
        ownerZombieCount[msg.sender]++;
        NewZombie(id, _name, _dna);
    }
```



### 第10章: 僵尸吃什么?

是时候让我们的僵尸去捕猎！ 那僵尸最喜欢的食物是什么呢？

Crypto 僵尸喜欢吃的是...

**CryptoKitties！** 😱😱😱

（正经点，我可不是开玩笑😆）

为了做到这一点，我们要读出 CryptoKitties 智能合约中的 kittyDna。这些数据是公开存储在区块链上的。区块链是不是很酷？

别担心 —— 我们的游戏并不会伤害到任何真正的CryptoKitty。 我们只 *读取* CryptoKitties 数据，但却无法在物理上删除它。

**与其他合约的交互**

如果我们的合约需要和区块链上的其他的合约会话，则需先定义一个 **interface** (接口)。

先举一个简单的栗子。 假设在区块链上有这么一个合约：

```solidity
contract LuckyNumber {
  mapping(address => uint) numbers;

  function setNum(uint _num) public {
    numbers[msg.sender] = _num;
  }

  function getNum(address _myAddress) public view returns (uint) {
    return numbers[_myAddress];
  }
}
```

这是个很简单的合约，您可以用它存储自己的幸运号码，并将其与您的以太坊地址关联。 这样其他人就可以通过您的地址查找您的幸运号码了。

现在假设我们有一个外部合约，使用 `getNum` 函数可读取其中的数据。

首先，我们定义 `LuckyNumber` 合约的 **interface** ：

```solidity
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

请注意，这个过程虽然看起来像在定义一个合约，但其实内里不同：

首先，我们只声明了要与之交互的函数 —— 在本例中为 `getNum` —— 在其中我们没有使用到任何其他的函数或状态变量。

其次，我们并没有使用大括号（`{` 和 `}`）定义函数体，我们单单用分号（`;`）结束了函数声明。这使它看起来像一个合约框架。

编译器就是靠这些特征认出它是一个接口的。

在我们的 app 代码中使用这个接口，合约就知道其他合约的函数是怎样的，应该如何调用，以及可期待什么类型的返回值。

在下一课中，我们将真正调用其他合约的函数。目前我们只要声明一个接口，用于调用 CryptoKitties 合约就行了。

**实战演习**

我们已经为你查看过了 CryptoKitties 的源代码，并且找到了一个名为 `getKitty`的函数，它返回所有的加密猫的数据，包括它的“基因”（我们的僵尸游戏要用它生成新的僵尸）。

该函数如下所示：

```solidity
function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
) {
    Kitty storage kit = kitties[_id];

    // if this variable is 0 then it's not gestating
    isGestating = (kit.siringWithId != 0);
    isReady = (kit.cooldownEndBlock <= block.number);
    cooldownIndex = uint256(kit.cooldownIndex);
    nextActionAt = uint256(kit.cooldownEndBlock);
    siringWithId = uint256(kit.siringWithId);
    birthTime = uint256(kit.birthTime);
    matronId = uint256(kit.matronId);
    sireId = uint256(kit.sireId);
    generation = uint256(kit.generation);
    genes = kit.genes;
}
```

这个函数看起来跟我们习惯的函数不太一样。 它竟然返回了...一堆不同的值！ 如果您用过 JavaScript 之类的编程语言，一定会感到奇怪 —— 在 Solidity中，您可以让一个函数返回多个值。

现在我们知道这个函数长什么样的了，就可以用它来创建一个接口：

1.定义一个名为 `KittyInterface` 的接口。 请注意，因为我们使用了 `contract` 关键字， 这过程看起来就像创建一个新的合约一样。

2.在interface里定义了 `getKitty` 函数（不过是复制/粘贴上面的函数，但在 `returns` 语句之后用分号，而不是大括号内的所有内容。

```solidity
// Create KittyInterface here
contract KittyInterface {
    function getKitty(uint256 _id) external view returns (
    bool isGestating,
    bool isReady,
    uint256 cooldownIndex,
    uint256 nextActionAt,
    uint256 siringWithId,
    uint256 birthTime,
    uint256 matronId,
    uint256 sireId,
    uint256 generation,
    uint256 genes
    );
}
```



### 第11章: 使用接口

继续前面 `NumberInterface` 的例子，我们既然将接口定义为：

```solidity
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}
```

我们可以在合约中这样使用：

```solidity
contract MyContract {
  address NumberInterfaceAddress = 0xab38...;
  // ^ 这是FavoriteNumber合约在以太坊上的地址
  NumberInterface numberContract = NumberInterface(NumberInterfaceAddress);
  // 现在变量 `numberContract` 指向另一个合约对象

  function someFunction() public {
    // 现在我们可以调用在那个合约中声明的 `getNum`函数:
    uint num = numberContract.getNum(msg.sender);
    // ...在这儿使用 `num`变量做些什么
  }
}
```

通过这种方式，只要将您合约的可见性设置为`public`(公共)或`external`(外部)，它们就可以与以太坊区块链上的任何其他合约进行交互。

**实战演习**

我们来建个自己的合约去读取另一个智能合约-- CryptoKitties 的内容吧！

1. 我已经将代码中 CryptoKitties 合约的地址保存在一个名为 `ckAddress` 的变量中。在下一行中，请创建一个名为 `kittyContract` 的 KittyInterface，并用 `ckAddress` 为它初始化 —— 就像我们为 `numberContract`所做的一样。

```solidity
contract ZombieFeeding is ZombieFactory {

  address ckAddress = 0x06012c8cf97BEaD5deAe237070F9587f8E7A266d;
  // Initialize kittyContract here using `ckAddress` from above
    KittyInterface kittyContract = KittyInterface(ckAddress);

  function feedAndMultiply(uint _zombieId, uint _targetDna) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    _createZombie("NoName", newDna);
  }

}
```



### 第12章: 处理多返回值

`getKitty` 是我们所看到的第一个返回多个值的函数。我们来看看是如何处理的：

```solidity
function multipleReturns() internal returns(uint a, uint b, uint c) {
  return (1, 2, 3);
}

function processMultipleReturns() external {
  uint a;
  uint b;
  uint c;
  // 这样来做批量赋值:
  (a, b, c) = multipleReturns();
}

// 或者如果我们只想返回其中一个变量:
function getLastReturnValue() external {
  uint c;
  // 可以对其他字段留空:
  (,,c) = multipleReturns();
}
```

**实战演习**

是时候与 CryptoKitties 合约交互起来了！

我们来定义一个函数，从 kitty 合约中获取它的基因：

1. 创建一个名为 `feedOnKitty` 的函数。它需要2个 `uint` 类型的参数，`_zombieId` 和`_kittyId` ，这是一个 `public` 类型的函数。

1. 函数首先要声明一个名为 `kittyDna` 的 `uint`。

	> 注意：在我们的 `KittyInterface` 中，`genes` 是一个 `uint256` 类型的变量，但是如果你记得，我们在第一课中提到过，`uint` 是 `uint256` 的别名，也就是说它们是一回事。

1. 这个函数接下来调用 `kittyContract.getKitty`函数, 传入 `_kittyId` ，将返回的 `genes` 存储在 `kittyDna` 中。记住 —— `getKitty` 会返回一大堆变量。 （确切地说10个 - 我已经为你数过了，不错吧！）。但是我们只关心最后一个-- `genes`。数逗号的时候小心点哦！

1. 最后，函数调用了 `feedAndMultiply` ，并传入了 `_zombieId` 和 `kittyDna` 两个参数。

```solidity
// define function here
    function feedOnKitty(uint _zombieId, uint _kittyId) public{
        uint kittyDna;
        (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
        feedAndMultiply(_zombieId,kittyDna);
    }
```



### 第13章: 奖励: Kitty 基因

我们的功能逻辑主体已经完成了...现在让我们来添一个奖励功能吧。

这样吧，给从小猫制造出的僵尸添加些特征，以显示他们是猫僵尸。

要做到这一点，咱们在新僵尸的DNA中添加一些特殊的小猫代码。

还记得吗，第一课中我们提到，我们目前只使用16位DNA的前12位数来指定僵尸的外观。所以现在我们可以使用最后2个数字来处理“特殊”的特征。

这样吧，把猫僵尸DNA的最后两个数字设定为`99`（因为猫有9条命）。所以在我们这么来写代码：`如果`这个僵尸是一只猫变来的，就将它DNA的最后两位数字设置为`99`。

**if 语句**

if语句的语法在 Solidity 中，与在 JavaScript 中差不多：

```solidity
function eatBLT(string sandwich) public {
  // 看清楚了，当我们比较字符串的时候，需要比较他们的 keccak256 哈希码
  if (keccak256(sandwich) == keccak256("BLT")) {
    eat();
  }
}
```

**实战演习**

让我们在我们的僵尸代码中实现小猫的基因。

1. 首先，我们修改下 `feedAndMultiply` 函数的定义，给它传入第三个参数：一条名为 `_species` 的字符串。

1. 接下来，在我们计算出新的僵尸的DNA之后，添加一个 `if` 语句来比较 `_species` 和字符串 `"kitty"` 的 `keccak256` 哈希值。

1. 在 `if` 语句中，我们用 `99` 替换了新僵尸DNA的最后两位数字。可以这么做：`newDna = newDna - newDna % 100 + 99;`。

	> 解释：假设 `newDna` 是 `334455`。那么 `newDna % 100` 是 `55`，所以 `newDna - newDna % 100` 得到 `334400`。最后加上 `99` 可得到 `334499`。

1. 最后，我们修改了 `feedOnKitty` 中的函数调用。当它调用 `feedAndMultiply` 时，增加 `“kitty”` 作为最后一个参数。

```solidity
// 这里修改函数定义
  function feedAndMultiply(uint _zombieId, uint _targetDna, string _species) public {
    require(msg.sender == zombieToOwner[_zombieId]);
    Zombie storage myZombie = zombies[_zombieId];
    _targetDna = _targetDna % dnaModulus;
    uint newDna = (myZombie.dna + _targetDna) / 2;
    // 这里增加一个 if 语句
    if(keccak256(_species) == keccak256("kitty")){
        newDna = newDna - newDna % 100 + 99;
    }
    _createZombie("NoName", newDna);
  }

  function feedOnKitty(uint _zombieId, uint _kittyId) public {
    uint kittyDna;
    (,,,,,,,,,kittyDna) = kittyContract.getKitty(_kittyId);
    // 并修改函数调用
    feedAndMultiply(_zombieId, kittyDna,"kitty");
  }
```



### 第14章: 放在一起

至此，你已经学完第二课了！

查看下→_→的演示，看看他们怎么运行起来得吧。继续，你肯定等不及看完这一页😉。点击小猫，攻击！看到你斩获一个新的小猫僵尸了吧！

**JavaScript 实现**

我们只用编译和部署 `ZombieFeeding`，就可以将这个合约部署到以太坊了。我们最终完成的这个合约继承自 `ZombieFactory`，因此它可以访问自己和父辈合约中的所有 public 方法。

我们来看一个与我们的刚部署的合约进行交互的例子， 这个例子使用了 JavaScript 和 web3.js：

```solidity
var abi = /* abi generated by the compiler */
var ZombieFeedingContract = web3.eth.contract(abi)
var contractAddress = /* our contract address on Ethereum after deploying */
var ZombieFeeding = ZombieFeedingContract.at(contractAddress)

// 假设我们有我们的僵尸ID和要攻击的猫咪ID
let zombieId = 1;
let kittyId = 1;

// 要拿到猫咪的DNA，我们需要调用它的API。这些数据保存在它们的服务器上而不是区块链上。
// 如果一切都在区块链上，我们就不用担心它们的服务器挂了，或者它们修改了API，
// 或者因为不喜欢我们的僵尸游戏而封杀了我们
let apiUrl = "https://api.cryptokitties.co/kitties/" + kittyId
$.get(apiUrl, function(data) {
  let imgUrl = data.image_url
  // 一些显示图片的代码
})

// 当用户点击一只猫咪的时候:
$(".kittyImage").click(function(e) {
  // 调用我们合约的 `feedOnKitty` 函数
  ZombieFeeding.feedOnKitty(zombieId, kittyId)
})

// 侦听来自我们合约的新僵尸事件好来处理
ZombieFactory.NewZombie(function(error, result) {
  if (error) return
  // 这个函数用来显示僵尸:
  generateZombie(result.zombieId, result.name, result.dna)
})
```

**实战演习**

选择一只你想猎食的小猫。你自家僵尸的 DNA 会和小猫的 DNA 结合，生成一个新的小猫僵尸，加入你的军团！

看到新僵尸上那可爱的猫咪腿了么？这是新僵尸最后DNA中最后两位数字 `99` 的功劳！

你想要的话随时可以重新开始。捕获了一只猫咪僵尸，你一定很高兴吧！（不过你只能持有一只），继续前进到下一章，完成第二课吧！

# 2025-08-09

| 今日学习内容                  |
| ----------------------------- |
| 安全与合规                    |
| Chainlink预言机的solidity课程 |



## Chainlink预言机的solidity课程

### solidity数据类型、函数、存储模式、数据结构

今天主要看了B站Chainlink预言机的solidity课程，然后跟着使用remix工具，进行简单的合约编写以及部署，学习到的包括数据结构、函数等内容，在昨天的CryptoZombiesx的课程中也都学过一遍，所以也就算是复习一遍了。

还有就是你觉得这个课程讲的真的很好，不仅讲解了密码学中包括公私钥加密原理，甚至还讲解了助记词产生过程以及助记词生成私钥的过程。强烈推荐！！

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.30;

contract HelloWorld{
    string strVar = "HelloWorld";
    struct Info {
        string phrase;
        uint256 id;
        address addr; 
    }

    Info[] infos;		//结构体数组

    mapping(uint256 id => Info info) infoMapping;

    function seyHello(uint256 _id) public view returns(string memory){
    	//通过判断是否存在地址，从而判断是否写入成功
    	//Mapping方式
        if(infoMapping[_id].addr == address(0x0)){
            return addinfo(strVar);
        }
        else {
            return addinfo(infoMapping[_id].phrase);
        }
		//for遍历方式，在infos数组很大时，遍历查询的消耗大，所以建议使用mapping键值对查询
        // for(uint256 i=0; i < infos.length; i++ ){
        //     if(infos[i].id == _id){
        //         return addinfo(infos[i].phrase);
        //     }
        // }
        // return addinfo(strVar);
    }
    function setHelloWorld(string memory newString, uint256 _id) public {
        Info memory info = Info(newString, _id, msg.sender);
        infoMapping[_id] = info; 
    } 
    function addinfo(string memory helloWorldStr) internal pure returns(string memory){
        return string.concat(helloWorldStr," from xxx's contract.");
    }
}
```



SPDX-License-Identifier：明确声明其使用的开源许可证。它通常以注释的形式出现在文件头部，帮助开发者、法律团队和自动化工具快速识别代码的许可条款



view：函数中只有读取操作，没有修改操作

pure：函数中种只需要进行运算，不需要读取任何变量



四个可见度标识符：

<img src="https://cdn.jsdelivr.net/gh/xmhhmx/PicGoCDN//img/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-08-09%20200259.png" style="zoom:67%;" />

**存储模式**：

永久性存储：storage

暂时性存储（交易结束后值就消失了）：memory、calldata

1、storage：永久存在合约内，合约中声明的默认是该类型，但不需要显示写出该关键词

2、memory：变量在运行时可以更改

3、calldata：变量在运行时不能更改，例如函数传参，用calldata传参数，函数中该值是不能改变的

4、stack

5、codes

6、logs



**数据结构**：

1、struct：结构体

2、array：数组

3、mapping：映射（键值对表示）



### solidity的工厂模式

#### 工厂模式介绍

在 Solidity 中，**工厂模式（Factory Pattern）** 是一种常用的智能合约设计模式，用于 **动态创建和管理其他合约的实例**。工厂合约（Factory Contract）负责部署子合约（Child Contracts），通常用于以下场景：

- 批量创建相同逻辑的合约（如代币、NFT、多签钱包等）。
- 降低重复部署的成本（通过复用逻辑合约）。
- 统一管理子合约地址。

```solidity
contract Factory {
      Child[] children;
      function createChild(uint data){
         Child child = new Child(data);
         children.push(child);
      }
}
contract Child{
     uint data;
     constructor(uint _data){
        data = _data;
     }
}
```



#### 引入合约方式

直接引入同一个文件系统下的合约

```solidity
import { HelloWorld } from "./test.sol";
```

也可以直接引入网上的URL以及可以精准写入sol文件中具体的合约名称

```
import { HelloWorld } from "https://11111/test.sol";
```

在实际应用中，涉及到公司的内容，会有更专业的引入方式，通过引入包的方式

```
import { HelloWorld } from "@companyName/product/contract";
```

# 2025-08-08

| 今日学习内容                                                 |
| ------------------------------------------------------------ |
| 安全与合规                                                   |
| cryptozombiesx的solidity学习/Solidity: Beginner to Intermediate Smart Contracts/lesson1搭建僵尸工厂 |



## 安全与合规







## cryptozombiesx的solidity学习

### Solidity: Beginner to Intermediate Smart Contracts

### lesson1搭建僵尸工厂

### 第2章: 合约

从最基本的开始入手:

Solidity 的代码都包裹在**合约**里面. 一份`合约`就是以太应币应用的基本模块， 所有的变量和函数都属于一份合约, 它是你所有应用的起点.

一份名为 `HelloWorld` 的空合约如下:

```solidity
contract HelloWorld {

}
```

#### 版本指令

所有的 Solidity 源码都必须冠以 "version pragma" — 标明 Solidity 编译器的版本. 以避免将来新的编译器可能破坏你的代码。

例如: `pragma solidity ^0.4.19;` (当前 Solidity 的最新版本是 0.4.19).

综上所述， 下面就是一个最基本的合约 — 每次建立一个新的项目时的第一段代码:

```solidity
pragma solidity ^0.4.19;

contract HelloWorld {

}
```

#### 实战演习

为了建立我们的僵尸部队， 让我们先建立一个基础合约，称为 `ZombieFactory`。

1. 在右边的输入框里输入 `0.4.19`，我们的合约基于这个版本的编译器。
1. 建立一个空合约 `ZombieFactory`。

一切完毕，点击下面 "答案" . 如果没效果，点击 "提示"。

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {

}
```



### 第3章: 状态变量和整数

真棒！我们已经为我们的合约做了一个外壳， 下面学习 Solidity 中如何使用变量。

**状态变量**是被永久地保存在合约中。也就是说它们被写入以太币区块链中. 想象成写入一个数据库。

例子:

```solidity
contract Example {
  // 这个无符号整数将会永久的被保存在区块链中
  uint myUnsignedInteger = 100;
}
```

在上面的例子中，定义 `myUnsignedInteger` 为 `uint` 类型，并赋值100。

#### 无符号整数: `uint`

`uint` 无符号数据类型， 指**其值不能是负数**，对于有符号的整数存在名为 `int` 的数据类型。

> 注: Solidity中， `uint` 实际上是 `uint256`代名词， 一个256位的无符号整数。你也可以定义位数少的uints — `uint8`， `uint16`， `uint32`， 等…… 但一般来讲你愿意使用简单的 `uint`， 除非在某些特殊情况下，这我们后面会讲。

#### 实战演习

我们的僵尸DNA将由一个十六位数字组成。

定义 `dnaDigits` 为 `uint` 数据类型, 并赋值 `16`。

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {
    uint dnaDigits = 16;
}
```



### 第4章: 数学运算

在 Solidity 中，数学运算很直观明了，与其它程序设计语言相同:

- 加法: `x + y`
- 减法: `x - y`,
- 乘法: `x * y`
- 除法: `x / y`
- 取模 / 求余: `x % y` *(例如, `13 % 5` 余 `3`, 因为13除以5，余3)*

Solidity 还支持 ***乘方操作\*** (如：x 的 y次方） // 例如： 5 ** 2 = 25

```solidity
uint x = 5 ** 2; // equal to 5^2 = 25
```

#### 实战演习

为了保证我们的僵尸的DNA只含有16个字符，我们先造一个`uint`数据，让它等于10^16。这样一来以后我们可以用模运算符 `%` 把一个整数变成16位。

1. 建立一个`uint`类型的变量，名字叫`dnaModulus`, 令其等于 **10 的 `dnaDigits` 次方**.

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {
    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;
}
```



### 第5章: 结构体

有时你需要更复杂的数据类型，Solidity 提供了 **结构体**:

```solidity
struct Person {
  uint age;
  string name;
}
```

结构体允许你生成一个更复杂的数据类型，它有多个属性。

> 注：我们刚刚引进了一个新类型, `string`。 字符串用于保存任意长度的 UTF-8 编码数据。 如： `string greeting = "Hello world!"`。

#### 实战演习

在我们的程序中，我们将创建一些僵尸！每个僵尸将拥有多个属性，所以这是一个展示结构体的完美例子。

1. 建立一个`struct` 命名为 `Zombie`.
1. 我们的 `Zombie` 结构体有两个属性： `name` (类型为 `string`), 和 `dna` (类型为 `uint`)。

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

}
```



### 第6章: 数组

如果你想建立一个集合，可以用 **数组**这样的数据类型. Solidity 支持两种数组: **静态**数组和**动态**数组:

```solidity
// 固定长度为2的静态数组:
uint[2] fixedArray;
// 固定长度为5的string类型的静态数组:
string[5] stringArray;
// 动态数组，长度不固定，可以动态添加元素:
uint[] dynamicArray;
```

你也可以建立一个 **结构体**类型的数组 例如，上一章提到的 `Person`:

```solidity
Person[] people; // 这是动态数组，我们可以不断添加元素
```

记住：状态变量被永久保存在区块链中。所以在你的合约中创建动态数组来保存成结构的数据是非常有意义的。

#### 公共数组

你可以定义 `public` 数组, Solidity 会自动创建 **getter** 方法. 语法如下:

```solidity
Person[] public people;
```

其它的合约可以从这个数组读取数据（但不能写入数据），所以这在合约中是一个有用的保存公共数据的模式。

#### 实战演习

为了把一个僵尸部队保存在我们的APP里，并且能够让其它APP看到这些僵尸，我们需要一个公共数组。

1. 创建一个数据类型为 `Zombie` 的结构体数组，用 `public` 修饰，命名为：`zombies`.

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

}
```



### 第7章: 定义函数

在 Solidity 中函数定义的句法如下:

```solidity
function eatHamburgers(string _name, uint _amount) {

}
```

这是一个名为 `eatHamburgers` 的函数，它接受两个参数：一个 `string`类型的 和 一个 `uint`类型的。现在函数内部还是空的。

> 注：: 习惯上函数里的变量都是以(`_`)开头 (但不是硬性规定) 以区别全局变量。我们整个教程都会沿用这个习惯。

我们的函数定义如下:

```solidity
eatHamburgers("vitalik", 100);
```

#### 实战演习

在我们的应用里，我们要能创建一些僵尸，让我们写一个函数做这件事吧！

1. 建立一个函数 `createZombie`。 它有两个参数: **_name** (类型为`string`), 和 **_dna** (类型为`uint`)。

暂时让函数空着——我们在后面会增加内容。

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function createZombie(string _name, uint _dna) {

    }

}
```



### 第8章: 使用结构体和数组

#### 创建新的结构体

还记得上个例子中的 `Person` 结构吗？

```solidity
struct Person {
  uint age;
  string name;
}

Person[] public people;
```

现在我们学习创建新的 `Person` 结构，然后把它加入到名为 `people` 的数组中.

```solidity
// 创建一个新的Person:
Person satoshi = Person(172, "Satoshi");

// 将新创建的satoshi添加进people数组:
people.push(satoshi);
```

你也可以两步并一步，用一行代码更简洁:

```solidity
people.push(Person(16, "Vitalik"));
```

> 注：`array.push()` 在数组的 **尾部** 加入新元素 ，所以元素在数组中的顺序就是我们添加的顺序， 如:

```solidity
uint[] numbers;
numbers.push(5);
numbers.push(10);
numbers.push(15);
// The `numbers` array is now equal to [5, 10, 15]
```

#### 实战演习

让我们创建名为createZombie的函数来做点儿什么吧。

1. 在函数体里新创建一个 `Zombie`， 然后把它加入 `zombies` 数组中。 新创建的僵尸的 `name` 和 `dna`，来自于函数的参数。
1. 让我们用一行代码简洁地完成它。

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function createZombie(string _name, uint _dna) {
        zombies.push(Zombie(_name, _dna));
    }

}
```



### 第9章: 私有 / 公共函数

Solidity 定义的函数的属性默认为`公共`。 这就意味着任何一方 (或其它合约) 都可以调用你合约里的函数。

显然，不是什么时候都需要这样，而且这样的合约易于受到攻击。 所以将自己的函数定义为`私有`是一个好的编程习惯，只有当你需要外部世界调用它时才将它设置为`公共`。

如何定义一个私有的函数呢？

```solidity
uint[] numbers;

function _addToArray(uint _number) private {
  numbers.push(_number);
}
```

这意味着只有我们合约中的其它函数才能够调用这个函数，给 `numbers` 数组添加新成员。

可以看到，在**<u>函数名字后面使用关键字 `private` 即可</u>**。**<u>和函数的参数类似，私有函数的名字用(`_`)起始。</u>**

#### 实战演习

我们合约的函数 `createZombie` 的默认属性是公共的，这意味着任何一方都可以调用它去创建一个僵尸。 咱们来把它变成私有吧！

1. 变 `createZombie` 为私有函数，不要忘记遵守命名的规矩哦！

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function _createZombie(string _name, uint _dna) private {
        zombies.push(Zombie(_name, _dna));
    }

}
```



### 第10章: 函数的更多属性

本章中我们将学习函数的返回值和修饰符。

#### 返回值

要想函数返回一个数值，按如下定义：

```solidity
string greeting = "What's up dog";

function sayHello() public returns (string) {
  return greeting;
}
```

Solidity 里，函数的定义里可包含返回值的数据类型(如本例中 `string`)。

#### 函数的修饰符

上面的函数实际上没有改变 Solidity 里的状态，即，它没有改变任何值或者写任何东西。

这种情况下我们可以把函数定义为 **view**, 意味着它只能读取数据不能更改数据:

```solidity
function sayHello() public view returns (string) {
```

Solidity 还支持 **pure** 函数, 表明这个函数甚至都不访问应用里的数据，例如：

```solidity
function _multiply(uint a, uint b) private pure returns (uint) {
  return a * b;
}
```

<u>**这个函数甚至都不读取应用里的状态 — 它的返回值完全取决于它的输入参数，在这种情况下我们把函数定义为 pure.**</u>

> 注：可能很难记住何时把函数标记为 pure/view。 幸运的是， Solidity 编辑器会给出提示，提醒你使用这些修饰符。

#### 实战演习

我们想建立一个帮助函数，它根据一个字符串随机生成一个DNA数据。

1. 创建一个 `private` 函数，命名为 `_generateRandomDna`。它只接收一个输入变量 `_str` (类型 `string`), 返回一个 `uint` 类型的数值。
1. 此函数只读取我们合约中的一些变量，所以标记为`view`。
1. 函数内部暂时留空，以后我们再添加代码。

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function _createZombie(string _name, uint _dna) private {
        zombies.push(Zombie(_name, _dna));
    }

    function _generateRandomDna(string _str) private view returns (uint) {
        
    }
}
```



### 第11章: Keccak256 和 类型转换

如何让 `_generateRandomDna` 函数返回一个全(半) 随机的 `uint`?

Ethereum 内部有一个散列函数`keccak256`，它用了SHA3版本。一个散列函数基本上就是把一个字符串转换为一个256位的16进制数字。字符串的一个微小变化会引起散列数据极大变化。

这在 Ethereum 中有很多应用，但是现在我们只是用它造一个伪随机数。

例子:

```solidity
//6e91ec6b618bb462a4a6ee5aa2cb0e9cf30f7a052bb467b0ba58b8748c00d2e5
keccak256("aaaab");
//b1f078126895a1424524de5321b339ab00408010b7cf0e6ed451514981e58aa9
keccak256("aaaac");
```

显而易见，输入字符串只改变了一个字母，输出就已经天壤之别了。

> 注: 在区块链中**安全地**产生一个随机数是一个很难的问题， 本例的方法不安全，但是在我们的Zombie DNA算法里不是那么重要，已经很好地满足我们的需要了。

#### 类型转换

有时你需要变换数据类型。例如:

```solidity
uint8 a = 5;
uint b = 6;
// 将会抛出错误，因为 a * b 返回 uint, 而不是 uint8:
uint8 c = a * b;
// 我们需要将 b 转换为 uint8:
uint8 c = a * uint8(b);
```

上面, `a * b` 返回类型是 `uint`, 但是当我们尝试用 `uint8` 类型接收时, 就会造成潜在的错误。如果把它的数据类型转换为 `uint8`, 就可以了，编译器也不会出错。

#### 实战演习

给 `_generateRandomDna` 函数添加代码! 它应该完成如下功能:

1. 第一行代码取 `_str` 的 `keccak256` 散列值生成一个伪随机十六进制数，类型转换为 `uint`, 最后保存在类型为 `uint` 名为 `rand` 的变量中。
1. 我们只想让我们的DNA的长度为16位 (还记得 `dnaModulus`?)。所以第二行代码应该 `return` 上面计算的数值对 `dnaModulus` 求余数(`%`)。

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function _createZombie(string _name, uint _dna) private {
        zombies.push(Zombie(_name, _dna));
    }

    function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    }

}
```



### 第12章: 放在一起

我们就快完成我们的随机僵尸制造器了，来写一个公共的函数把所有的部件连接起来。

写一个公共函数，它有一个参数，用来接收僵尸的名字，之后用它生成僵尸的DNA。

####  实战演习

1. 创建一个 `public` 函数，命名为 `createRandomZombie`. 它将被传入一个变量 `_name` (数据类型是 `string`)。 *(注: 定义公共函数 `public` 和定义一个私有 `private` 函数的做法一样)*。
1. 函数的第一行应该调用 `_generateRandomDna` 函数，传入 `_name` 参数, 结果保存在一个类型为 `uint` 的变量里，命名为 `randDna`。
1. 第二行调用 `_createZombie` 函数， 传入参数： `_name` 和 `randDna`。
1. 整个函数应该是4行代码 (包括函数的结束符号 `}` )。

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {

    // 这里建立事件

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function _createZombie(string _name, uint _dna) private {
        zombies.push(Zombie(_name, _dna));
    }

    function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    }

    function createRandomZombie(string _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```



### 第13章: 事件

我们的合约几乎就要完成了！让我们加上一个**事件**.

**事件** 是合约和区块链通讯的一种机制。你的前端应用“监听”某些事件，并做出反应。

例子:

```solidity
// 这里建立事件
event IntegersAdded(uint x, uint y, uint result);

function add(uint _x, uint _y) public {
  uint result = _x + _y;
  //触发事件，通知app
  IntegersAdded(_x, _y, result);
  return result;
}
```

你的 app 前端可以监听这个事件。JavaScript 实现如下:

```solidity
YourContract.IntegersAdded(function(error, result) {
  // 干些事
})
```

#### 实战演习

我们想每当一个僵尸创造出来时，我们的前端都能监听到这个事件，并将它显示出来。

1。 定义一个 `事件` 叫做 `NewZombie`。 它有3个参数: `zombieId` (`uint`)， `name` (`string`)， 和 `dna` (`uint`)。

2。 修改 `_createZombie` 函数使得当新僵尸造出来并加入 `zombies`数组后，生成事件`NewZombie`。

3。 需要定义僵尸`id`。 `array.push()` 返回数组的长度类型是`uint` - 因为数组的第一个元素的索引是 0， `array.push() - 1` 将是我们加入的僵尸的索引。 `zombies.push() - 1` 就是 `id`，数据类型是 `uint`。在下一行中你可以把它用到 `NewZombie` 事件中。

```solidity
pragma solidity ^0.4.19;

contract ZombieFactory {
	
	// 这里建立事件
    event NewZombie(uint zombieId, string name, uint dna);

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;

    struct Zombie {
        string name;
        uint dna;
    }

    Zombie[] public zombies;

    function _createZombie(string _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        //// 这里触发事件
        NewZombie(id, _name, _dna);
    }

    function _generateRandomDna(string _str) private view returns (uint) {
        uint rand = uint(keccak256(_str));
        return rand % dnaModulus;
    }

    function createRandomZombie(string _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

}
```



### 第14章: Web3.js

我们的 Solidity 合约完工了！ 现在我们要写一段 JavaScript 前端代码来调用这个合约。

以太坊有一个 JavaScript 库，名为**Web3.js**。

在后面的课程里，我们会进一步地教你如何安装一个合约，如何设置Web3.js。 但是现在我们通过一段代码来了解 Web3.js 是如何和我们发布的合约交互的吧。

如果下面的代码你不能全都理解，不用担心。

```js
// 下面是调用合约的方式:
var abi = /* abi是由编译器生成的 */
var ZombieFactoryContract = web3.eth.contract(abi)
var contractAddress = /* 发布之后在以太坊上生成的合约地址 */
var ZombieFactory = ZombieFactoryContract.at(contractAddress)
// `ZombieFactory` 能访问公共的函数以及事件

// 某个监听文本输入的监听器:
$("#ourButton").click(function(e) {
  var name = $("#nameInput").val()
  //调用合约的 `createRandomZombie` 函数:
  ZombieFactory.createRandomZombie(name)
})

// 监听 `NewZombie` 事件, 并且更新UI
var event = ZombieFactory.NewZombie(function(error, result) {
  if (error) return
  generateZombie(result.zombieId, result.name, result.dna)
})

// 获取 Zombie 的 dna, 更新图像
function generateZombie(id, name, dna) {
  let dnaStr = String(dna)
  // 如果dna少于16位,在它前面用0补上
  while (dnaStr.length < 16)
    dnaStr = "0" + dnaStr

  let zombieDetails = {
    // 前两位数构成头部.我们可能有7种头部, 所以 % 7
    // 得到的数在0-6,再加上1,数的范围变成1-7
    // 通过这样计算：
    headChoice: dnaStr.substring(0, 2) % 7 + 1，
    // 我们得到的图片名称从head1.png 到 head7.png

    // 接下来的两位数构成眼睛, 眼睛变化就对11取模:
    eyeChoice: dnaStr.substring(2, 4) % 11 + 1,
    // 再接下来的两位数构成衣服，衣服变化就对6取模:
    shirtChoice: dnaStr.substring(4, 6) % 6 + 1,
    //最后6位控制颜色. 用css选择器: hue-rotate来更新
    // 360度:
    skinColorChoice: parseInt(dnaStr.substring(6, 8) / 100 * 360),
    eyeColorChoice: parseInt(dnaStr.substring(8, 10) / 100 * 360),
    clothesColorChoice: parseInt(dnaStr.substring(10, 12) / 100 * 360),
    zombieName: name,
    zombieDescription: "A Level 1 CryptoZombie",
  }
  return zombieDetails
}
```

我们的 JavaScript 所做的就是获取由`zombieDetails` 产生的数据, 并且利用浏览器里的 JavaScript 神奇功能 (我们用 Vue.js)，置换出图像以及使用CSS过滤器。在后面的课程中，你可以看到全部的代码。

# 2025-08-07

| 今日学习内容            |
| ----------------------- |
| MyFirstLayer2  剩余学习 |
| 行业赛道全览            |

## MyFirstLayer2

### 1 公链瓶颈

#### 1.1改进尝试

为了保证参与门槛足够低，比特币将全网同步的出块时间控制在 10 分钟，平均 TPS （每秒可处理交易笔数）仅有个位数。以太坊出块时间约 12 秒，平均 TPS 也仅有十几笔。这对比于传统 Web2 的经济活动来说，远远不够用

提高区块链性能的两个尝试：

- <u>增大单个区块的大小，容纳更多的交易</u>

	这样做会引起区块账本的快速膨胀，参与验证的机器性能要求越来越高，提高了参与门槛，导致整个网络去中心化程度和安全性渐渐降低。从 BTC 分叉出来的 BCH（Bitcoin Cash） 将区块大小从 1MB 提升至 32MB，BSV（Bitcoin Satoshi's Vision） 则是更激进地取消了区块大小上限，允许无限多的信息融入一个区块。

- <u>降低出块的时间，追求一定时间内出更多的块来处理更多的交易</u>

	这样对节点的网络条件提出了更高要求，提高了参与门槛。并且影响了全网数据同步的稳定性，因为物理上相隔较远的节点集群容易对最新的区块产生分歧，导致分叉。分叉链总需要竞争出新的最长链，抛弃其中的一条分支，导致过去一段时间内的许多交易被重写，这就是“区块重组”现象，Polygon 在 2023 年发生过 157 个区块的重组事件。

此外，还有一些公链试图用更激进的方式改善性能：

- <u>使用数量更少的超级节点通讯：</u>

	超级节点的性能更强大，网络带宽更好更稳定，因此彼此之间能实现超高速的通讯，但这显然降低了去中心化程度。如 Fanton 有 51 个共识节点，BSC、EOS、TRON 则仅有 21 个超级节点。

- <u>用特殊的共识机制提升性能：</u>

	共识机制决定了全网节点对出块方式如何达成共识，一些特殊的机制也许可以提高出块速度，但共识机制越复杂，就对机器性能要求越高，也更容易出现单点故障导致整个系统出错。如 Solana，全网节点依赖随机选出的单个 Leader 节点来协调，因此获得了极高的理论 TPS 上限，但对节点性能要求变得非常高，并多次发生全网宕机的安全性事故。

目前高性能公链的探索成果，普遍获得了将 TPS 提升至 100 ~ 1000 的成绩，但共识节点数量也降低为几十个至一千多个不等。对比于以太坊分布全球的近万个共识节点 ，高性能公链在性能提高了一两个数量级的同时，去中心化程度也下降了一两个数量级。



#### 1.2 区块链三难困境

由于区块链的底层特性，存在一个不可能三角悖论。区块链不可能三角（Blockchain Trilemma）是指在设计区块链系统时，存在三个目标之间的矛盾，这三个目标分别是去中心化、安全性和可扩展性（性能）

1. **去中心化**（Decentralization）：指的是在区块链系统中，所有的节点都具有相同的权力，没有单一的中心化权威节点进行控制。这个目标是区块链的核心特性，也是保证系统安全性和抗攻击性的基础。
1. **安全性**（Security）：指的是在区块链系统中，保证交易的真实性、完整性、不可篡改性和抗攻击性等方面的安全。这个目标是区块链系统的重要保障，也是确保系统可靠性和信任度的基础。
1. **可扩展性**（Scalability）：可扩展性即性能，指的是在区块链系统中，支持足够大量的交易、节点和用户等系统扩展。这个目标是区块链系统的重要需求，也是确保系统能够满足现实需求的基础。

这三个目标之间存在矛盾关系。例如，在追求更高的去中心化的情况下，需要所有节点都保存完整的区块链数据，但是这会导致系统的可扩展性降低。相反，在追求更高的可扩展性的情况下，需要牺牲一定的去中心化；还比如采用分片技术 [1] 来扩容，但是可能导致节点同步数据困难，更容易发生故障，导致安全性降低。

**高安全性和高可扩展性**

追求安全性和可扩展性（即性能），往往采用少数超级节点进行通讯，超级节点拥有更强的性能和更好的网络环境，彼此之间能实现超高速的通讯。但参与门槛过高，牺牲了去中心化程度。

代表区块链：BSC、EOS、TRON 等区块链采用了少数高性能节点维护网络，仅有 21 个超级节点进行记账。

**高可扩展性和高去中心化**

追求可扩展性（即性能）和去中心化程度，为保证去中心化采用了较多验证节点，为了追求性能提高了出块速度，或采用了特殊的共识机制。但提高出块速度容易导致大规模区块重组，更复杂的共识机制容易导致全网宕机等安全事故，牺牲了安全性。

代表区块链：Polygon 在 2023 年发生了 157 个区块的大规模重组；Solana 多次出现全网宕机的事故。

**高去中心化和高安全性**

追求去中心化程度和安全性，采用更多的节点和更公平的出块方式，值得信赖。但为了允许低性能节点参与验证，协调全球网络延迟，导致每秒可处理的交易数较低，牺牲了性能。

代表区块链：BTC、ETH 追求了极致的安全可靠和去中心化，但处理交易的速度较低，BTC 约为 7 笔/秒，ETH 约为 10 - 20 笔/秒。



#### 1.3 Layer 2

事实上，有一种方法可以克服区块链三难困境。而工程实践上，我们可以通过增加层级的方式来优化三者的矛盾，增加层级实现了业务解耦，降低了原先层级的负担。

增加第二层网络（Layer 2 ），来分摊一层网络（Layer 1）的负担，达到扩容的目的。

> Layer 2 即二层网络，是在一层网络的基础上搭建的，用各种技术手段帮助底层公链扩容的解决方案



### 2 Layer 2发展史

#### 2.1 状态通道（State channel）

> 假设 Alice 经常在一家咖啡店消费，如果每次买一杯 5 美元的咖啡，都需要支付 0.5 美元的手续费，这也太痛苦了。那么假如 Alice 和咖啡店能达成共识，每次买咖啡时付给咖啡店一张签了名的欠条，一段时间后咖啡店攒了足够多的欠条，将欠条算好总金额一次性兑现，这样交易成本就可以大幅降低，对双方都有利。这种思路就是最早的二层网络，也就是状态通道的原理。

**状态通道** 是一种区块链扩容技术，允许用户在链下进行高频、低成本的交互，仅在开启和关闭通道时与区块链交互，从而实现近乎即时、零摩擦的交易。状态通道使用了**多签技术** [1]，允许两个个体之间提前存入一笔资金锁定在智能合约中，建立一个内部通道，然后双方可以在通道内进行多笔小额转账，速度极快，成本极低，再在一段时间后用转账证明一次性提款。状态通道也是比特币的 Lightning Network（闪电网络），以太坊的 Raiden Network（雷电网络）背后的底层技术。

> [1] 多签技术（Multi-sig）：即多重签名技术，是指需要多个授权方共同授权才能完成交易，提高了交易的安全性和可靠性。如公司财库的资金要求三人之中的至少两人签名（2/3）才能动用，状态通道中则要求同时拥有双方的签名（2/2）才能生效，多签也允许出现更多 n/m 的授权条件。



> **1. 核心原理**
>
> **(1) 基本流程**
>
> 1. **开启通道**：
> 	- 双方将资金锁定在链上的智能合约中（如存入 10 ETH）。
> 	- 生成初始状态（如 Alice: 6 ETH，Bob: 4 ETH），并签名确认。
> 1. **链下交易**：
> 	- 双方通过签名消息更新状态（如 Alice 转 1 ETH 给 Bob → Alice:5, Bob:5）。
> 	- **无需矿工/验证者**，交易即时生效。
> 1. **关闭通道**：
> 	- 将最终状态提交到链上，合约根据最新状态分配资金。
> 	- 若有争议（如一方提交过期状态），可挑战并惩罚作弊者。
>
> **(2) 技术关键点**
>
> - **多签名验证**：每次状态更新需双方签名。
> - **时间锁（Timelock）**：防止旧状态被恶意提交。
> - **欺诈证明（Fraud Proof）**：允许诚实方在争议期举证。
>
> **2. 为什么需要状态通道？**
>
> |     **问题**     |   **状态通道解决方案**   |
> | :--------------: | :----------------------: |
> | 链上交易慢且昂贵 | 链下交易零成本，即时确认 |
> | 区块链吞吐量有限 |    支持无限次链下交互    |
> |     隐私性差     | 交易细节仅在参与者间传递 |
>
> ------
>
> **3. 典型应用场景**
>
>  **(1) 高频微支付**
>
> - **案例**：
> 	- 按秒计费的视频流服务（如每小时支付 0.001 ETH）。
> 	- 游戏内道具实时交易。
>
>  **(2) 链下投票与治理**
>
> - **案例**：
> 	- DAO 成员在通道内频繁投票，最终结果上链。
>
> **(3) 隐私保护交易**
>
> - **案例**：
> 	- 双方链下协商价格，仅公开最终交易。
>
> **4.优缺点**
>
>  **✅ 优点**
>
> - **零延迟**：交易即时完成。
> - **零 Gas 费**：链下交互无需付费。
> - **隐私性**：交易细节不上链。
>
> **❌ 缺点**
>
> - **通道管理复杂**：需预先锁定资金。
> - **适用性有限**：仅适合固定参与者间的交互。
> - **流动性要求**：长期占用资金。



状态通道技术本质上是使用了**中心化的节点**，用户在链上质押一笔较大额的资金，然后在链下用签了名的“欠条”进行付款，积攒了一定量的“欠条”之后，任何一方都可以选择关闭通道进行结算。链上的智能合约只认可同时拥有双方签名的转账信息，Alice 和咖啡店都拥有自己的签名，再加上“欠条”上对方的签名，才能凑齐签名**通过多签验证**，因此无法提取不属于自己的资金。

但这样做产生了 2 个新的问题：

1. Alice 和咖啡店之间攒“欠条”的约定仅适合于最简单的转账交易，去中心化金融要实现的交易比转账复杂得多，那么**想要实现更复杂的交易要怎么办呢**？为了解决这个问题，引出了**侧链**方案。
1. Alice 可以给咖啡店发送资金，若超市与咖啡店有通道，也可以借助咖啡店向超市发送资金，但是**如何给未参与雷电网络的个体发送资金呢**？这个问题，则引出了 **Plasma** 方案。



#### 2.2 侧链（Sidechain）

为了解决**状态通道无法执行复杂交易**的问题，侧链方案进入人们视野。

**侧链**可以理解为一条相对独立的区块链，它们往往采用与主链（一般是以太坊）类似的架构，方便主链上的项目迁移至侧链。

我们可以在主链的智能合约内锁定一定量的资产，然后在侧链上铸造等量资产，实现“原子交换”。用这种方式将资产存入侧链，在侧链上进行各种交易，然后在必要时转移回主链。



**侧链核心特性**

|   **特性**   |                        **说明**                         |
| :----------: | :-----------------------------------------------------: |
|  **独立性**  | 拥有自己的共识机制（如 PoA、PoS）、区块参数和智能合约。 |
| **双向锚定** |      资产可从主链锁定后映射到侧链，也能反向返回。       |
| **互操作性** |           通过桥接协议（Bridge）与主链通信。            |
|  **定制化**  |   可针对特定需求优化（如高TPS、低Gas费、隐私增强）。    |

侧链会进行一定的定制化，获得更高的性能，帮助主链分担交易压力。

- 采用 **POS** 共识机制（Proof of Stake），大幅提高了出块速度以达到扩容的目的。如 Polygon 侧链，将出块时间缩短至 2 秒。
- 采用 **POA** 共识机制（Proof of Authority），用更少的经过授权的超级节点进行通信，以实现侧链上的超高性能。如负责游戏资产交易的 Ronin 侧链，因游戏内的 NFT 资产本就比较中心化，所以这种更中心化的方案也可以接受。



**侧链的工作原理**

**(1) 资产跨链流程（以以太坊 ↔ Polygon 为例）**

1. **存款（主链 → 侧链）**
	- 用户将 ETH 存入主链的桥合约，合约锁定资产并生成证明。
	- 侧链验证证明后，在侧链上 mint 等量的封装资产（如 WETH）。
1. **提款（侧链 → 主链）**
	- 用户销毁侧链的 WETH，提交销毁证明到主链桥合约。
	- 主链合约验证后解锁原始 ETH。

**(2) 共识机制差异**

- **主链**：通常采用高安全共识（如以太坊的 PoS）。
- **侧链**：可能使用更高效的共识（如 Polygon PoS 链的 Bor 共识）。



**为什么需要侧链？**

|      **主链瓶颈**       |               **侧链解决方案**               |
| :---------------------: | :------------------------------------------: |
| 低TPS（如以太坊15 TPS） |  侧链可达数千 TPS（如 Polygon 7,000 TPS）。  |
|         高Gas费         | 侧链交易成本极低（如 BSC 的 $0.01 交易费）。 |
|        功能限制         |  侧链可定制模块（如隐私交易、游戏专用链）。  |



 **典型侧链项目**

|      **侧链**      | **锚定主链** |         **特点**         |    **用例**    |
| :----------------: | :----------: | :----------------------: | :------------: |
|  **Polygon PoS**   |    以太坊    |      高TPS，EVM兼容      |   DeFi、NFT    |
|  **Gnosis Chain**  |    以太坊    | 注重去中心化，xDai 合并  |   支付、DAO    |
| **Liquid Network** |    比特币    |  联邦桥接，支持机密交易  | 比特币快速结算 |
|     **Ronin**      |    以太坊    | Axie Infinity 游戏专用链 |     GameFi     |



**侧链的安全风险**

**(1) 桥接攻击**

- **案例**：2022 年 Ronin 桥被盗 6.25 亿美元（私钥泄露）。
- **防御**：使用多重签名、零知识证明桥（如 zkBridge）。

**(2) 共识中心化**

- 部分侧链（如 BSC）由少数节点控制，可能被操纵。

**(3) 资产锚定失效**

- 若桥合约漏洞导致双向锚定断裂，侧链资产可能脱钩。



侧链通过 **牺牲部分主链安全性** 换取高性能和低成本，是区块链生态的重要扩容手段。用户在使用时需权衡**效率、成本与风险**，优先选择经过时间验证的侧链和桥接协议。



#### 2.3 Plasma

随着状态通道和侧链方案在实践中暴露出不足，Plasma 方案被提出并得到重视，它解决了前面两者暴露出的两个问题：

- **无法给未参与的个体发送资金**：未加入 Plasma 链的账户也可以收到资金，然后自行提取到 Layer 1。
- **无法继承 Layer 1 安全性**：Plasma 定期向主链提交信息，以继承 Layer 1 安全性。

**Plasma** 是以太坊创始人 **Vitalik Buterin** 和 **Joseph Poon** 在 2017 年提出的一种 **区块链扩容方案**，旨在通过构建多层链下结构（“子链”），将大部分交易从<u>主链（如以太坊）卸载到子链上处理</u>，从而显著提升吞吐量并降低交易成本。尽管后期因技术复杂性被 Rollup 取代，但 Plasma 的设计思想仍对 Layer2 发展影响深远。



**Plasma 的核心思想**

 **(1) 分层结构**

- **主链（Root Chain）**：以太坊等底层区块链，负责最终结算和争议仲裁。
- **子链（Plasma Chain）**：独立的链下区块链，处理高频交易，定期向主链提交状态承诺（Merkle Root）。

**(2) 关键机制**

- **状态承诺**：子链将交易批量打包后，生成 Merkle Root 并提交到主链。
- **欺诈证明（Fraud Proof）**：若子链作恶（如篡改交易），用户可提交证明至主链，触发惩罚和状态回滚。
- **资金退出（Exit）**：用户需通过主链的“退出游戏”机制安全取回资金。



**Plasma 的工作原理**

**(1) 存款（主链 → Plasma 链）**

1. 用户将资产（如 ETH）锁定在主链的 Plasma 智能合约中。
1. Plasma 链生成对应的资产凭证（如 1:1 映射的代币）。

**(2) 链下交易（Plasma 链内）**

- 用户在 Plasma 链上自由交易（如转账、支付），无需主链确认。
- 子链区块生产者（Operator）定期将交易打包，并提交 Merkle Root 到主链。

**(3) 提款（Plasma 链 → 主链）**

1. 用户发起提款请求，启动 **7天挑战期**（类似 Optimistic Rollup）。
1. 若无争议，主链释放锁定的资产；若有欺诈，提交证明取消提款。



**Plasma 的优缺点**

**✅ 优点**

- **高吞吐量**：交易在子链处理，主链仅存储压缩数据。
- **低成本**：Gas 费由子链承担，主链仅需极低开销。
- **安全性继承**：依赖主链的**欺诈证明机制**确保资金安全。

 **❌ 缺点**

- **退出延迟**：提款需等待挑战期（7天），流动性受限。
- **数据可用性问题**：**<u>若子链运营商隐藏数据，用户无法构造欺诈证明。</u>**
- **通用性差**：难以支持复杂智能合约（仅适合支付等简单逻辑）。



**为什么 Plasma 逐渐被淘汰？**

1. **数据可用性问题**：用户无法获取子链完整数据时，无法挑战无效交易。
1. **用户体验差**：提款延迟长，且需主动监控欺诈。
1. **Rollup 的崛起**：
	- **ZK Rollup** 通过零知识证明解决数据问题。
	- **Optimistic Rollup** 保留欺诈证明但强制数据上链，平衡安全与成本。



> 默克尔树利用了哈希算法（Hash Algorithm），每个子节点的内容，都影响着上一个节点的哈希值。因此保存在默克尔树内的数据，若被改变了哪怕一个字符，都将导致上方一连串节点的哈希值改变，最终传导至默克尔树的根部也发生改变。任何人都可以自行运算哈希算法来检查计算出的哈希值与公开的哈希值是否对应，以此确认内容是否被篡改。



#### 2.4 Rollup

虽然 Plasma 最终未能大规模使用，但它的一些理念在后续的 Layer 2 方案中被吸收和发展。

既然困扰 Plasma 最大的问题是**数据可用性**，也就是监督者不容易获取交易数据以进行检验的问题。那我们如果不仅仅上传一个根证明，而是把必要的关键数据通通上传至 Layer 1 ，是不是就解决了这个棘手问题呢？

这个思路带来了目前最具可行性的 Layer 2 扩容方案： Rollup。Rollup 意为“打包”，也就是将一段时间内发生的交易先进行压缩，再进行打包，然后周期性地上传至主网。目前主流的 Rollup 方案分为两大路线，分别为 **Optimistic Rollup**（乐观的 Rollup）与 **Zero-knowledge Rollup**（零知识证明的 Rollup）。

- Optimistic Rollup：OP-Rollup 是将一段时间内的所有交易细节全部压缩打包，定期发送至 Layer 1。这种机制乐观地相信大部分交易都是诚实的，继承了 Plasma 挑战期和欺诈证明机制。
- Zero-knowledge Rollup：ZK-Rollup 一般是将一段时间内的交易计算完成后，将状态变化的结果压缩打包，并附上交易已经在 Layer 2 被正确执行的零知识证明，定期发送至 Layer 1。用零知识证明代替监督者，依赖数学而非验证者（Rely on Math, not Validators）。



**Rollup 的核心思想**

**(1) 链下执行 + 链上验证**

- **链下**：交易在 Rollup 链（Layer2）上执行。
- **链上**：交易数据（或有效性证明）提交到主链（Layer1），确保安全性。

**(2) 数据压缩**

- Rollup 将多笔交易压缩为 **单个批次**，减少链上存储开销。
- 例如：1000 笔交易 → 1 个 Rollup 区块 → 1 笔主链交易。



**Rollup 的两种类型**

**(1) Optimistic Rollup（乐观汇总）**

- **假设交易合法**，默认不验证，依赖 **欺诈证明（Fraud Proof）** 争议机制。
- **特点**：
	- 兼容 EVM（如 Arbitrum、Optimism）。
	- 提款需等待 **7天挑战期**。
	- 成本低，适合通用智能合约。

**工作流程**：

1. 交易在 Layer2 执行。
1. 排序器（Sequencer）将交易数据（Calldata）提交到主链。
1. 若有欺诈，验证者在挑战期内提交证明，回滚错误状态。

**(2) ZK Rollup（零知识证明汇总）**

- 每批交易生成 **零知识证明（ZK-SNARK/STARK）**，主链即时验证。
- **特点**：
	- 无需挑战期，提款即时到账。
	- 证明生成计算量大，早期难兼容 EVM（现 zkEVM 已突破）。
	- 隐私性更强（可隐藏交易细节）。

**工作流程**：

1. 交易在 Layer2 执行。
1. 生成有效性证明（Proof）并提交到主链。
1. 主链验证证明后，更新状态。



**Rollup 的优缺点**

**✅ 优点**

- **高扩展性**：吞吐量提升百倍。
- **安全性**：依赖主链验证，抗审查。
- **低成本**：Gas 费仅为链上的 1/10~1/100。

**❌ 缺点**

- **Optimistic 的延迟**：提款需等待 7 天（可通过流动性提供商缓解）。
- **ZK 的硬件需求**：证明生成需要高性能服务器。
- **中心化风险**：排序器可能被垄断（逐步去中心化中）。



> ### **零知识证明（Zero-Knowledge Proof, ZKP）详解**
>
> **零知识证明** 是一种密码学技术，允许一方（证明者）向另一方（验证者）**证明某个陈述的真实性**，而无需透露任何额外信息。其核心思想是：**“我知道一个秘密，但我不会告诉你秘密是什么”**。
>
> 在区块链领域，ZKP 是隐私保护（如 Zcash）和扩容（如 ZK Rollup）的核心技术。
>
> #### **1. 零知识证明的三大核心特性**
>
> 1. **完备性（Completeness）**：
> 	- 如果陈述为真，诚实验证者一定会被说服。
> 1. **可靠性（Soundness）**：
> 	- 如果陈述为假，作弊的证明者无法欺骗验证者。
> 1. **零知识性（Zero-Knowledge）**：
> 	- 验证者除了“陈述为真”外，无法获取任何其他信息。
>
> #### **2. 经典类比：洞穴寓言**
>
> 假设有一个环形洞穴，中间有一道需要密码才能打开的门。
>
> - **证明者**知道密码，想向**验证者**证明这一点，但不想泄露密码。
> - **过程**：
> 	1. 验证者站在洞口，随机要求证明者从左侧（A）或右侧（B）进入。
> 	1. 证明者无论从哪边进入，都能用密码开门并从另一侧出来。
> 	1. 重复多次后，验证者确信证明者确实知道密码，但始终不知道密码是什么。
>
> #### 3. 零知识证明的两种主要类型
>
> **(1) 交互式零知识证明（Interactive ZKP）**
>
> - 需要多轮通信（如洞穴寓言中的多次进出）。
> - **缺点**：效率低，不适合区块链。
>
> **(2) 非交互式零知识证明（Non-Interactive ZKP, NIZK）**
>
> - 证明者生成单次证明，验证者可随时检查。
> - **区块链常用**：如 ZK-SNARKs、ZK-STARKs。
>
> #### **4. 区块链中的 ZKP 技术**
>
>  **(1) ZK-SNARKs（简洁非交互式知识论证）**
>
> - **特点**：
> 	- 证明体积小（约 200 字节），验证速度快。
> 	- 需要“可信设置”（Trusted Setup），存在潜在风险。
> - **应用**：
> 	- **Zcash**（隐私转账）。
> 	- **zkSync**（ZK Rollup）。
>
>  **(2) ZK-STARKs**
>
> - **特点**：
> 	- 无需可信设置，抗量子计算。
> 	- 证明体积较大（约 100 KB），但验证速度仍快。
> - **应用**：
> 	- **StarkNet**（以太坊 Layer2）。
>
> **(3) Bulletproofs**
>
> - **特点**：
> 	- 无需可信设置，适合范围证明（如 Confidential Transactions）。
> - **应用**：
> 	- **Monero**（隐私币）。
>
> #### **5. 零知识证明的区块链应用**
>
> **(1) 隐私保护**
>
> - **匿名交易**：Zcash 使用 ZK-SNARKs 隐藏发送方、接收方和金额。
> - **身份验证**：证明年龄 >18 岁，而无需透露出生日期。
>
>  **(2) 扩容（ZK Rollup）**
>
> - <u>将数千笔交易打包，生成 ZKP 证明提交到主链，验证者只需检查证明即可确认有效性。</u>
> - **代表项目**：zkSync、StarkNet、Scroll。
>
> **(3) 去中心化存储验证**
>
> - 证明文件已正确存储，而无需下载全部数据（如 Filecoin）。
>
> #### **6. 零知识证明的优缺点**
>
> **✅ 优点**
>
> - **隐私性**：隐藏敏感数据（如交易详情）。
> - **扩展性**：减少链上计算负担（如 ZK Rollup）。
> - **安全性**：数学上无法伪造证明。
>
> **❌ 挑战**
>
> - **计算密集型**：生成证明需要高性能硬件。
> - **可信设置（ZK-SNARKs）**：初始参数若泄露，可能伪造证明。
> - **开发门槛高**：需要密码学专业知识。



#### 2.5 Layer 2 对比

**核心特性对比**

|   **维度**   |     **State Channel**     |    **Sidechain**     |    **Plasma**     |        **Rollup**        |
| :----------: | :-----------------------: | :------------------: | :---------------: | :----------------------: |
|  **安全性**  |       依赖最终结算        |     依赖侧链共识     | 依赖主链欺诈证明  |      继承主链安全性      |
| **交易速度** |       即时（链下）        | 较快（依赖侧链性能） | 较快（链下执行）  |     较快（链下执行）     |
|   **成本**   | 链下免费，仅开/关通道付费 |   低（侧链Gas费）    |       极低        |     极低（数据压缩）     |
| **去中心化** |  需预存资金，参与者固定   |   可变（PoA/PoS）    |    依赖运营商     |       逐步去中心化       |
| **数据存储** |      仅最终状态上链       |     独立链上存储     | 仅提交Merkle Root | 交易数据全上链（Rollup） |
| **适用场景** |  高频小额支付（如游戏）   | 独立生态（如GameFi） | 简单支付/资产转移 |       通用智能合约       |



**工作原理对比**

**(1) State Channel（状态通道）**

- **流程**：
	1. 双方锁定资金在主链。
	1. 链下无限次交易（仅双方签名）。
	1. 最终结算状态上链。
- **例子**：比特币闪电网络、以太坊的Raiden Network。

**(2) Sidechain（侧链）**

- **流程**：
	1. 资产通过桥锁定在主链，映射到侧链。
	1. 侧链独立运行（自有共识机制）。
	1. 提款时反向桥接回主链。
- **例子**：Polygon PoS链、Ronin（Axie Infinity侧链）。

 **(3) Plasma**

- **流程**：
	1. 资产锁定在主链Plasma合约。
	1. 子链处理交易，定期提交Merkle Root到主链。
	1. 提款需挑战期（防欺诈）。
- **例子**：早期OMG Network（已转向Rollup）。

 **(4) Rollup**

- **流程**：
	1. 交易在链下执行并压缩。
	1. 数据批量提交到主链（Optimistic需欺诈证明，ZK需有效性证明）。
- **例子**：Optimism（OP）、Arbitrum、zkSync。



**安全性对比**

|     **方案**      |  **安全模型**  |            **主要风险**             |
| :---------------: | :------------: | :---------------------------------: |
| **State Channel** | 依赖参与者诚实 |      对手方离线时资金可能锁定       |
|   **Sidechain**   |  依赖侧链共识  |  桥接攻击（如Ronin被盗6.25亿美元）  |
|    **Plasma**     |  主链欺诈证明  | 数据不可用性问题（运营商隐藏数据）  |
|    **Rollup**     |  继承主链安全  | Optimistic的7天延迟，ZK的证明中心化 |



**优缺点总结**

|     **方案**      |          **优点**          |            **缺点**            |
| :---------------: | :------------------------: | :----------------------------: |
| **State Channel** | 零延迟，零Gas费，隐私性好  |   仅限固定参与者，需预存资金   |
|   **Sidechain**   |      高性能，灵活定制      |    安全性依赖侧链，桥接风险    |
|    **Plasma**     |      高吞吐量，低成本      |   退出延迟长，不支持复杂逻辑   |
|    **Rollup**     | 继承主链安全，支持智能合约 | Optimistic有延迟，ZK开发门槛高 |



**未来趋势**

- **Rollup 主导**：Optimistic 和 ZK Rollup 成为以太坊扩容主流。
- **State Channel 小众化**：仅用于特定场景（如微支付）。
- **Sidechain 专用化**：游戏、社交等垂直领域。
- **Plasma 淘汰**：被 Rollup 取代（因数据可用性问题）。



### 3 Rollup详情

#### 3.1 如何压缩

以 OP-Rollup 的为例，我们要向 Layer 1 上传一段时间内的所有交易详情，如果不对这部分数据进行高度压缩，那分担负载的效果就非常小了。我们以单笔交易为例，它身上其实有许多可改进的空间。

比如一笔常见的转账交易，它的原数交易数据可能是以下这样的：

**4232f461**000000000000000000000000**7ea2be2df7ba6e54b1a9c70676f668455e329d29**000000000000000000000000**d548a5e31de2b4c2681a58a3be5302abcae4bc57**00000000000000000000**000000000000000000000000000000000000000186a0**

（**Method ID** / 零填充[1] / **代币合约地址** / 零填充 / **收款的账户地址** / 零填充 / **提币数量**）

>  [1]零填充：之所以要填充零占用空间，是因为以太坊中的交易数据是固定长度编码的，比如 Method ID 占用 128 位（32 个十六进制字母），地址和金额占用 256 位（64 个十六进制字母），不够长的信息字段需要填充 0 以保持数据对齐和一致性。

<u>**原始交易数据可以通过以下手段压缩：**</u>

1. 用**科学计数法**把转账数量压缩成 64 位数据，并删除不必要的 0。（数量的精度会略微下降，但实践中影响不大）
1. 调用的方法如果很常见，可以删除所调用的 Method ID，因为如“转账一笔 ERC20 [3] 代币”的交易，可以通过交易内容的特征推测
1. 常用行为设置绿色通道（Helper ID）：大部分发送代币的行为都是如 USDC、WETH 等常用代币，可以用更短的一个 Helper ID，来表示调用方法是“发送”，发送的代币是“USDC”这两个信息。
1. 登记一个“电话簿”，记录收款人地址，将 40 位的地址压缩为第 XXX 页的第 X 个地址。
1. 如果发送的是 ETH，连 Helper ID 都可以省掉。



最终我们需要上传至 Layer 1 的数据从一段非常长的信息

**4232f461**000000000000000000000000**7ea2be2df7ba6e54b1a9c70676f668455e329d29**000000000000000000000000**d548a5e31de2b4c2681a58a3be5302abcae4bc57**00000000000000000000**000000000000000000000000000000000000000186a0**

变为了

**059c57**0186a0

（**收款账户“电话簿”编号** / 提币数量）



**数据压缩过程**

1. **原始状态**

<u>原始交易数据未经压缩</u>

> **4232f461**000000000000000000000000**7ea2be2df7ba6e54b1a9c70676f668455e329d29**000000000000000000000000**d548a5e31de2b4c2681a58a3be5302abcae4bc57**00000000000000000000**000000000000000000000000000000000000000186a0**

（Method ID / 零填充 / 代币合约地址 / 零填充  / 收款的账户地址 / 零填充 / 提币数量）

2. **压缩状态1**

<u>用**科学计数法**把转账数量压缩成 64 位数据，并删除不必要的 0。（数量的精度会略微下降，但实践中影响不大）</u>

> **4232f461**7ea2be2df7ba6e54b1a9c70676f668455e329d29**d548a5e31de2b4c2681a58a3be5302abcae4bc57**0186a0

（Method ID / 代币合约地址 / 收款的账户地址 / 提币数量）

3. **压缩状态2**

<u>调用的方法如果很常见，可以删除所调用的 Method ID，因为如“转账一笔 ERC20 [3] 代币”的交易，可以通过交易内容的特征推测</u>

> **7ea2be2df7ba6e54b1a9c70676f668455e329d29**d548a5e31de2b4c2681a58a3be5302abcae4bc57**0186a0**

（代币合约地址 / 收款的账户地址 / 提币数量）

4. **压缩状态3**

<u>常用行为设置绿色通道（Helper ID）：大部分发送代币的行为都是如 USDC、WETH 等常用代币，可以用更短的一个 Helper ID，来表示调用方法是“发送”，发送的代币是“USDC”这两个信息。</u>

> **0000ee**d548a5e31de2b4c2681a58a3be5302abcae4bc57**0186a0**

（Helper ID / 收款的账户地址 / 提币数量）

5. **压缩状态4**

<u>登记一个“电话簿”，记录收款人地址，将 40 位的地址压缩为第 XXX 页的第 X 个地址。</u>

> **0000ee**059c01**0186a0**

（Helper ID / 收款账户“电话簿”编号 / 提币数量）

6. **压缩状态5**

<u>如果发送的是 ETH，连 Helper ID 都可以省掉。</u>

> **059c01**0186a0

（收款账户“电话簿”编号 / 提币数量）



Layer 1 存储数据的成本是非常高昂的，Layer 2 的执行成本绝大部分的都消耗在了这一步，因此压缩需要上传的数据上可以显著降低 Layer 2 整体的交易成本。



#### 3.2 进一步压缩

数据压缩是个远早于区块链就存在的技术，除了前面对交易的定制化压缩之外，还有许多压缩算法能帮助进一步压缩数据空间。

比如我们生活中往往都接触过 zip、rar、7z 压缩包，它们就是使用了压缩算法将各种文件的体积减小。 Optimism 的 Zlib 压缩算法、 Arbitrum 的 Brotli 压缩算法都能起到类似的作用。



#### 3.3 Optimistic Rollup

Optimistic Rollup 是“乐观的打包”，它假设绝大部分的参与者都是诚实的，允许一批数量较少的验证者节点（Validator），对交易进行收集、排序、验证。同时还设置了挑战者的角色（Challenger），其职责是监督验证者提交的信息是否诚实。

OP-Rollup 会定期向主网上传两种数据：

- 状态根（State Root）: 状态根可以快速确认 Layer 2 小账本的内容是否被篡改。
- 压缩后的全部交易数据：包含各种交易细节，比如交易附带的“用户签名”。

虽然上传了近段时间的全部交易详情，但以太坊主网并不负责直接验证这些交易，只起到一个公示的作用。



与 Plasma 类似，OP-Rollup 也使用默克尔树的形式保存了一个“小账本”，记录了全体账户的所有状态（账户余额）。如果我们相信目前的交易验证者（Validator）都是诚实的，那么状态根能快速确认当前 Layer 2 的小账本记录的内容是否被篡改，确保安全性。

反之，假如我们对目前的交易合法性产生质疑，任何第三方都可以在主网获取最近一段时间内所有交易的副本，重新验证之后，将自己验证的结果与 Layer 2 的小账本的记录做对比，确认小账本上的记录均为合法。如果发现作恶，挑战者就可以在 Layer 1 提交欺诈证明来改写 Layer 2 的状态。挑战成功后，不诚实的验证者将受到惩罚，挑战者将获得奖励。同时，受影响的交易将被回滚 [1]，进行重新验证。

> [1] 回滚的影响范围取决于具体 Layer 2 的设计机制。有些选择仅回滚无效的交易；有些选择将受影响的区块变为孤块，从未受影响的区块后继续，这将导致一段时间内的交易都将被回滚，进行重新验证；有些选择用多轮挑战等其他手段，缩小需要回滚的范围。

这个过程中，负责监督的挑战者（Challenger）是直接与 Layer 1 的智能合约交互的，一层对二层的状态有着最终裁决权。



这种设计之下，即使只有一个诚实的挑战者，也足以确保整个 Layer 2 的交易安全。不过代价是 OP-Rollup 必须提供一个退出窗口期，让挑战者有时间去检验并提交欺诈证明，因此使用官方桥从 OP-Rollup 网络提款往往需要 7 - 14 天的等待期。

OP-Rollup 的逻辑简单易懂，而且上传了全部交易细节，因此对 EVM [2] （以太坊虚拟机）的兼容性也非常好，很有利于落地实施。但漫长的挑战期实在是太痛苦了，有办法解决**退出等待时间过长的问题**吗？

> [2] EVM 即 Ethereum Virtual Machine，是以太坊中智能合约的执行环境。虚拟机（Virtual Machine）是通过软件模拟的具有完整硬件系统功能的、运行在隔离环境中的完整计算机系统。开发者不必关心底层细节如何实现，只要在以太坊虚拟机中开发，就能确保代码执行环境的一致性。



#### 3.4 Zero-Knowledge Proof

造成漫长等待期的原因，是因为 OP-Rollup 需要人的参与（挑战者扮演的角色）。如果能让整个过程不需要人的参与，只需要算法的参与，那就完美了。

**零知识证明**是指向他人证明某个命题为真，但又不透露“该命题为真”之外的任何信息。例如，有一个环形的长廊，在走廊中间某处有一道安装了密码锁的门。如果 A 要向 B 证明自己拥有该门的密码，无需向 B 展示自己打开密码锁的过程。只需要让 B 看着 A 从入口进入走廊，然后又从另一侧的出口走出走廊，就可以完全证明 A 拥有密码，同时免除了暴露密码的风险。



**零知识证明具备以下性质**：

1. 完备性（Completeness）：若命题为真，任何证明者可以向验证者提出令人信服的证据，即“真的可被验证”。
1. 可靠性（Soundness）：若命题为假，则不存在不诚实的证明者能骗过验证者，即“假的会被发现”。
1. 零知识性（Zero-knowledge）：证明某个命题为真，但又不透露“该命题为真”之外的其他任何信息。

工程实践上我们还要求零知识证明的算法拥有以下性质：

1. 简洁性（Succinctness）：证明很小且验证速度快。
1. 零知识（Zero Knowledge）：可以隐藏计算的输入信息。



ZK-Rollup 会周期性向主网上传 3 种数据：

- 状态根：状态根可以快速确认 Layer 2 小账本的内容是否被篡改。
- 交易数据：经过压缩和聚合的交易数据，例如将多个交易合并为一批次的状态变化结果。通过使用零知识证明保证交易的安全性，可以舍弃一些不必要的信息，例如前面提到的“用户签名”。
- 有效性证明：即零知识证明，让 Layer 1 的智能合约在经过简单验证后，就能确认交易已经被正确执行。



ZK-Rollup 与 OP-Rollup 最大的不同，就在于 <u>Layer 1 是否对 Layer 2 上传的数据进行验证</u>。

ZK-Rollup 方案依靠 Layer 1 上的智能合约，用很小的代价检验 ZK Proof 的有效性，如果检验通过则表示这批交易已经被正确执行，那就更新状态。如果检验不通过，则拒绝这一批次的交易。

OP-Rollup 则完全将 Layer 1 当成了解决数据可用性的公告板，依赖挑战者的监督，两者的安全性和交易确认速度都会产生明显差异。

ZK-Rollup 的优点显而易见：

- 依靠数学而非验证者，安全性更有保障，并且确认时间更短。
- 更高的压缩率，让 Layer 2 的扩容上限更高。



#### 3.5 ZK 技术原理 STARK vs SNARK

<u>**本内容暂时无法理解**</u>

**ZK-Rollup 的基础组件**

ZK-Rollup 用了一些特定的数学工具，来实现在不透露原始输入数据的情况下确保交易已被正确执行，它通常包含以下主要技术：

- KZG 多项式承诺：因两个多项式最多拥有 n2 个交点，而定义域却存在极多的点，那么我们只需要检查有限的若干次，就能确信对方确实以正确的多项式进行了计算。若将信息编码在多项式中，则靠多次确认多项式在特定点上的输出结果，即可确认交易已被正确验证（确认过程原本是需要交互的，但可用其他方法变为非交互式）。
- 哈希算法：能将任意长度的数据映射为固定长度的哈希值，用于压缩证明。
- 椭圆曲线加密：可以将椭圆曲线上的两个点用难以预测的方式映射起来，用于构建证明系统。可用来进行一些复杂的证明，比如在不公开哈希值的情况下证明两个哈希相等。
- 随机数等其它组件：用于随机数来确认起始需要检查的点，并用类似“上一个哈希影响下一个哈希”的方式确认一连串需要检查的点，以确保检查点的随机性与非交互性。



##### **SNARK**

目前零知识证明主要有两种技术路线，SNARK 与 STARK。SNARK 出现更早，更加成熟，目前被更多的项目方采用。

SNARK：Succinct Non-Interactive Argument of Knowledge

1. 简洁（Succinct）：验证速度快于计算证明。
1. 非交互式（Non-Interactive）：无需证明者与验证者之间进行交互。如比特币的公私钥对也是一种零知识证明，但它要求私钥拥有者对一段文字进行签名才能证明自己拥有私钥，这需要发生一次交互。
1. 统计学上的可靠（Argument）：相对于数学上绝对的证明(即 100% 可靠)，实现了统计学上的可靠（如 99.99999999%）。
1. 包含信息（Knowledge）：可将某些信息编码进零知识证明中，如一笔交易已被正确执行。



##### **STARK**

STARK：Scalable Transparent Argument of Knowledge

1. 可扩展的（Scalable）：在进行大规模交易的证明时，验证时间仍然较短。
1. 透明的（Transparent）：随机数公开可验证，无需像 SNARK 一样设置初始可信环境。
1. 统计学上的可靠（Argument）：相对于数学上绝对的证明(即 100% 可靠)，实现了统计学上的可靠（如 99.99999999%）。
1. 包含信息（Knowledge）：可将某些信息编码进零知识证明中，如一笔交易已被正确执行。



对比两条技术路线，STARK 在 Layer 2 上处理大量交易时，其零知识证明生成速度和验证速度更快的特点将具有优势，并且具有抗量子性 [1]，无需初始可信环境 [2]，更加安全。而 SNARK 发展时间更久更成熟，将会更早取得应用。

SNARK 证明体积更小，并且随着技术发展，已经可以在升级后沿用最初的可信环境，在安全性上不会显著弱于 STARK。SNARK 与 STARK 的关系更像是 OP-Rollup 与 ZK-Rollup 之间的关系，前者可能更早落地，后者拥有更大潜力。

从底层技术路线去研究 ZK-Rollup 显得有点复杂了，**有更简单的方式去理解 ZK-Rollup 吗**？



> [1] 抗量子性指能够在量子计算机攻击下保证信息安全。传统的加密算法（如 RSA、DSA 和 ECC 等）可能在将来的大规模量子计算机上被有效地破解，因此需要使用新的抗量子密码学算法来替代，确保加密在未来的量子计算机时代仍然安全。
>
> [2] 初始可信环境是指在协议部署的最初，需要信任部署者诚实地部署了一个安全环境。比如参与部署的多个实体中，有一人销毁了自己的私钥，即可确认初始信息没有任何人能掌控。初始可信环境主要与随机数有关，零知识证明的计算过程非常依赖随机数作为输入，但如果初始的随机数被掌握，可能使得后续的计算变得可预测，产生被攻击的风险。



#### 3.6 ZK 发展路线 ZK-VM vs ZK-EVM

##### **ZK-VM：从零知识证明的角度出发**

第一种思路是从零知识证明技术出发，**专门开发适用于零知识证明的算法，从而构建一个 ZK-VM [1] （零知识虚拟机），而不是原生兼容 EVM 的 ZK-EVM [2] （零知识以太坊虚拟机）**，在此基础上尽可能实现 EVM 兼容。

StarkWare 和 zkSync 都采用了这种路线。StarkWare 的 Cairo 语言和 zkSync 的 Zinc 语言都是原生的零知识编程语言，甚至前者的账户地址系统与以太坊也存在很大的差异。

这种路线的优点是能够充分发挥零知识证明的潜力，实现最大化的扩展性。但缺点是开发者需要学习新的编程语言，并且现有项目需要将 Solidity 语言 [3] 的代码转换为 ZK-VM 的代码，这个过程中可能出现许多意想不到的问题，需要重新调试，重新审计。



##### **ZK-EVM：从 EVM 兼容性的角度出发**

反过来我们也可以从兼容 EVM 这个目的出发，**将 EVM 的交易在操作码 [4] 层面切割成更小的步骤，对每个步骤去找对应的零知识证明算法，力求实现完全的 EVM 兼容。** 这样做可以使开发者几乎无感地切换到二层网络，方便现有项目迁移，最大程度地保留目前的 EVM 生态成果。Scroll 和 Polygon Hermez 都采用了这种思路。

但是这种路线的缺点也显而易见，EVM 上的交易并非为零知识而设计，因此这种方法往往生成的证明体积较大，所需的时间也更长。



读到这里，我们已经了解了 Rollup 的当前进展。特别是零知识证明技术，令人感到十分神奇，使 Rollup 具备了非常大的潜力。但是仅仅通过 Rollup 实现扩容仍然存在上限，**未来我们还有其他提升的空间吗**？



> [1] VM（Virtual Machine）即虚拟机，一个由软件模拟硬件的可控代码执行环境。 ZK-VM（Zero Knowledge - Virtual Machine） 是适应于零知识证明的虚拟机。
>
> [2] ZK-EVM（Zero Knowledge - Ethereum Virtual Machine）是指利用了零知识证明技术的以太坊虚拟机，与原始的以太坊虚拟机有良好的兼容性。
>
> [3] Solidity 语言是以太坊原生的开发语言，大部分运行在以太坊网络上的智能合约均采用 Solidity 编写。
>
> [4] 计算机能直接读懂的语言是 0 和 1 ，操作码（Opcode）是非常底层的，与硬件（或是虚拟机中的虚拟硬件）直接交流的代码。操作码是指令集中每个指令的唯一标识符，在计算机执行指令时，用于识别具体指令的一个数字或符号。例如，在 x86 指令集中，加法指令（ADD）有一个特定的操作码，用于标识该指令。
>
> 此外还有一些相关概念：
>
> 1. 指令集（Instruction Set）： 指令集是一种用于编程的低级语言，它是计算机硬件能够理解和执行的一组指令。每种处理器都有自己的指令集，这些指令集定义了处理器可以执行的基本操作，如数据移动、算术和逻辑运算、条件分支等。指令集也被称为指令集架构（ISA，Instruction Set Architecture）。
> 1. 字节码（Bytecode）： 字节码是一种中间代码，介于源代码和机器代码之间。字节码通常由虚拟机（如 Java 虚拟机，JVM）执行，而不是直接由硬件执行。字节码的目的是提供一种平台无关的代码表示形式，这样不同平台上的虚拟机都可以执行相同的字节码。字节码通常比源代码更接近机器代码，但仍需要经过解释或编译为特定硬件的指令集。
>
> 指令集是计算机硬件能够执行的一组基本操作，每个操作都有一个唯一的操作码（Opcode）。字节码是一种平台无关的中间代码形式，它在执行时需要虚拟机将其转换为特定硬件的指令集。在这个过程中，字节码中的指令也会有相应的操作码。总之，操作码是指令的标识符，它在指令集和字节码中都有应用。



### 4 未来展望

#### 4.1 其它解决方案：Validium 和 Volition

纵览二层扩容方案的发展历程，其实都源自于**数据可用性**影响了安全性与实用性。

1. 侧链方案，数据对主链来说不可用，所以无法继承主链安全性。
1. Plasma 方案，状态根可以保障链下账本的不被篡改，但具体交易数据对一层网络来说也不可用，导致资金退出困难。
1. OP-Rollup 方案，让所有交易的详细数据都存在一层网络上，确保了全部交易数据的可用性，从而确保二层网络的安全。
1. ZK-Rollup 方案，提供关键交易数据以及能证明链下交易已经被正确执行的零知识证明，实现安全性的同时，还能一定程度保证隐私性。Rely on Math，Not Validator.

那么基于**零知识证明和数据可用性的新组合**，我们可以得到 Validium 和 Volition （由 StarkNet 提出）两种未来可行的方案。



##### Validium

ZK-Rollup 仍向主网上传了交易数据，那如果我们进一步牺牲一点点安全性，连交易数据也不上传到一层，仅上传状态根和证明交易已被正确执行的零知识证明，由二层网络自行解决交易数据的保存，就可以进一步降低交易成本，这就是 Validium。

它有点像零知识版的 Plasma，Plasma 需要用户参与监督交易是否被诚实执行，而采用了零知识证明之后，不诚实的交易无法生成证明，极大地提升了节点作恶的难度。并且因为交易数据对于一层网络来说完全不可用，Validium 将具有更高的隐私性。



##### Volition

Volition 是 Validium 方案的改进版本，允许用户可以自行选择其交易数据是否在一层网络可用，涉及大额资金的交易可以采用成本略高但安全性更好的交易上链模式。资金量较小的、更追求隐私的交易可以使用交易记录不上链的模式。



> **1. Validium：链下数据 + ZK 证明**
>
> **核心特点**
>
> - **数据存储**：交易数据存储在链外（由第三方委员会或去中心化网络维护，如 StarkEx 的 DAC）。
> - **安全性依赖**：依赖数据可用性委员会（DAC）的诚实性，若委员会作恶可能冻结资金（但无法盗取，因 ZK 证明保证状态正确）。
> - **优势**：
> 	- 极低的交易成本（无需支付以太坊主网的数据存储费用）。
> 	- 高吞吐量（适合高频交易场景，如交易所、游戏）。
> - **劣势**：
> 	- 牺牲部分去中心化（需信任 DAC）。
> 	- 用户无法独立验证数据（需委员会配合提供数据）。
>
> **典型应用**
>
> - **dYdX**（V3 版本）：采用 Validium 模式处理现货交易，降低手续费。
> - **Immutable X**：为 NFT 交易提供零 Gas 费体验。
>
> ------
>
> **2. Volition：用户自选数据存储模式**
>
>  **核心特点**
>
> - **混合架构**：允许用户为每笔交易选择数据存储方式：
> 	- **ZK-Rollup 模式**：数据上链（高安全性，高成本）。
> 	- **Validium 模式**：数据链下（低成本，依赖 DAC）。
> - **灵活性**：同一应用中，不同用户（甚至同一用户的不同交易）可自由选择模式。
> - **优势**：
> 	- 平衡安全性与成本（例如，大额交易选 Rollup，小额选 Validium）。
> 	- 无需迁移即可切换模式。
> - **劣势**：
> 	- 实现复杂度较高。
>
> **典型应用**
>
> - **StarkEx**（StarkWare 的引擎）：支持 Volition，供开发者灵活配置。
> - **Sorare**（梦幻足球 NFT 平台）：对普通用户用 Validium，对机构用户用 Rollup。



|   **维度**   |         **Validium**         |               **Volition**               |
| :----------: | :--------------------------: | :--------------------------------------: |
| **数据存储** |       强制链下（DAC）        | 用户可选链上（Rollup）或链下（Validium） |
|  **安全性**  |           依赖 DAC           |           用户自主控制安全等级           |
|   **成本**   |     极低（无链上数据费）     |       按需支付（Rollup 模式更贵）        |
| **适用场景** | 高频、低价值交易（如交易所） |   需灵活性的应用（如混合型 DeFi/NFT）    |
| **代表项目** |     dYdX V3, Immutable X     |      StarkEx 生态项目（如 Sorare）       |



#### 4.2 Deneb 更新与 Layer 2

以太坊 Deneb 更新（原 EIP-4844 ）是以太坊扩容路线 DankSharding 的前置步骤，旨在保证以太坊信标链安全性的前提下，为 Layer 2 扩容提供更大的储存资源。



Deneb 更新添加了一种新的交易：Blob 交易（大型二进制对象 Binary Large Object）。在主链上只留存指针，指针指向一个 Blob 块，上面可以储存约 128KB 的二进制数据。

以太坊矿工只负责在一段时间内（如一个月）保存 Blob 上的数据，通过随机抽样的方式确保 Blob 上二进制数据的真实性，但不验证存在 Blob 上的交易。

Blob 交易看起来对常规交易的扩容没有帮助，但这对于 Layer 2 打包的交易数据来说简直是完美契合。Layer 2 原先定期上传给 Layer 1 上的打包交易数据本来就不会执行，只是起到保障数据可用性的作用。

而 Blob 本身就相当于一个公示板，消息在公示板上的存续时间超过了 OP-Rollup 的挑战期，实践上完全够用，存储成本却降低了许多，这将促使 Layer 2 的交易成本进一步降低。

Blob 增加的唯一缺憾是超过保存期之后，Layer 2 如果想保存自己的全部历史交易数据，需要用另外的方案自行解决这些数据的可用性问题。

 

总体来说，Deneb 更新之后，Layer 2 将可以在保证安全性的同时进一步降低交易成本，变得更具实用价值。

探索到这里，**我们增加了这么多性能，在应用层能有什么提升吗？** 还是单纯只是量的堆积？



#### 4.3 账户抽象与大规模应用

钱包毫无疑问是 Web3 最重要的流量入口，是用户进入 Web3 世界最重要的工具，大部分用户主要接触到的钱包都是 EOA 钱包（外部拥有账户），即通过私钥和助记词掌握整个钱包的权限（如 MetaMask、Coinbase Wallet、Trust Wallet 等），这种 EOA 钱包有如下问题：

1. 高学习成本：用户需要了解复杂的非对称加密知识，理解任何人拥有私钥就能拥有账户，失去私钥就是失去资金的控制权。
1. 单点失效问题：私钥泄露丢失，或忘记密码时，既不能用手机或邮箱，也不能柜面处理用身份证件恢复，资产将永久丢失。
1. 风险控制难度较高：存在很多恶意攻击手段，试图盗取私钥；或在不盗取私钥的前提下，用恶意授权、恶意签名等方式盗取用户资产。新用户需要漫长的学习，具备谨慎的使用习惯才能保护资产安全。
1. 不支持智能合约：无法实现复杂多样的高级功能，如多签、批量发送代币等等。



**账户抽象**（Account Abstraction）本质上是个智能合约钱包。是指将某些功能和具体实现细节抽象出来，用一系列的智能合约模拟出一个“账户”的效果，脱离了 EOA 账户用私钥控制地址的底层原理，允许其它更人性化的确认账户所有权的方式，并让整个账户的功能更加灵活。它解决了现有 EOA 账户的问题：

1. 更符合现有的账户使用习惯：可以像传统 Web2 服务一样登陆，使用邮箱、手机短信、两步验证等验证方法。
1. 更多安全保障机制：出现安全风险时可以紧急暂停账户功能，并采用社交恢复等手段重新掌握账户控制权。
1. 可以接入第三方安全模块：接入专业的风控模块，过滤风险操作，从底层降低被攻击的可能。
1. 支持智能合约的更多功能：让用户在不具备代码能力的情况下使用多签、批量发送多种代币、批量授权额度等等操作。



甚至还增加了许多新的可能：

1. 代理支付 Gas 费：允许第三方支付 Gas 费，可以由钱包运营商补贴 Gas 费来吸引用户，钱包作为入口展现商业价值，用户享受更便宜的服务。
1. 条件交易：在满足一定条件时自动执行某些交易。
1. 跨链操作：可以和跨链桥智能合约实现原生的交互，更灵活地实现资产跨链和其他跨链交互。
1. 更多 DeFi 场景：账户抽象为 DeFi 提供了更多可能，如批量交易、自动借贷、流动性挖矿等。

账户抽象极大地拓展了以太坊的可能性，但账户抽象的问题在于一切交易都基于智能合约完成，Gas 成本较高，这让它在以太坊主网上显得过于昂贵。

Layer 2 作为一个执行层，天然适配账户抽象，用户不会感知到私钥甚至不会感知到 Gas 费的存在。智能合约钱包本身就是程序，能引入风控等第三方服务，让转账和交互变得安全，有更多的丰富业务场景可以实现。可以说将来区块链技术的大规模应用技术离不开账户抽象，而账户抽象需要成本更低的 Layer 2 才能成立。

# 2025-08-06

| 今日学习内容           |
| ---------------------- |
| MyFirstLayer2 部分学习 |
| 区块链岗位全景图       |



## MyFirstLayer2

### 1 公链瓶颈

#### 1.1改进尝试

为了确保参与的低门槛，比特币将整个网络的区块生成时间控制在 10 分钟左右，从而使平均每秒交易数 （TPS） 为个位数。以太坊的区块生成时间约为 12 秒，平均 TPS 仅为几十。与传统 Web2 的经济活动相比，这还不够

提高区块链性能的两个尝试：

- <u>增加单个区块的大小以容纳更多交易</u>

	这种方法导致区块链账本迅速膨胀，对参与验证机器的性能要求越来越高，提高了进入门槛，并逐渐降低了整个网络的去中心化和安全性。从比特币分叉出来的比特币现金（BCH）将区块大小从 1MB 增加到 32MB，而比特币中本聪的愿景（BSV）则更积极地取消了区块大小限制，允许将无限量的信息合并到区块中。

- <u>减少区块生成时间，目标是在一定时间内生成更多区块，以处理更多交易</u>

	这种方式对节点的网络条件提出了更高的要求，增加了进入门槛。它还影响了整个网络数据同步的稳定性，因为物理上相距较远的节点集群容易对最新区块产生分歧意见，从而导致分叉。分叉链需要竞争产生一条新的最长链，丢弃一个分支并导致在一定时期内重写许多交易，称为“区块重组”。Polygon 在 2023 年经历了 157 次区块重组事件

此外，一些公链试图以更激进的方式提高性能：

- *<u>使用更少的超级节点进行通信</u>*

	超级节点具有更高的性能和更好、更稳定的网络带宽，使它们之间的通信速度超快。然而，这显然降低了去中心化。例如，Fantom 有 51 个共识节点，而币安智能链（BSC）、EOS 和 TRON 只有 21 个超级节点。

- <u>使用特殊的共识机制来增强性能</u>

	共识机制决定了网络中的节点如何就区块生成达成共识。一些特殊的机制可能会提高区块生成速度，但共识机制越复杂，对机器性能的要求就越高，单点故障就越容易造成系统错误。例如，Solana 依靠从整个节点网络中随机选择的单个领导者节点进行协调，实现了理论上较高的 TPS 上限。但对节点性能要求极高，曾多次发生全网宕机安全事件。

高性能公链的探索成果普遍实现了 TPS 提升至 100-1000，但共识节点数量减少到 10s 至 1000 以上的区间。与以太坊在全球拥有近 10000 个分布式共识节点 [3] 相比，高性能公链的性能提高了一两个数量级，而去中心化程度却下降了一两个数量级。那么，区块链在尝试改进 Layer 1 时面临哪些挑战 [4] 呢？



#### 1.2 区块链三难困境

区块链三难困境代表了设计区块链系统时三个主要目标之间的冲突：**去中心化**、**安全性**和**可扩展性**

1. 去中心化：这需要确保区块链系统内的所有节点拥有平等的权力，而没有单一的中心化机构控制网络。去中心化是区块链的一个基本方面，是系统安全和抵御攻击的基础。
1. 安全性：安全性涉及确保区块链系统内交易的真实性、完整性、不变性和抵抗攻击性。它是区块链系统可靠性和可信度的重要保证。
1. 可扩展性：可扩展性或性能是指区块链系统支持大量交易、节点和用户的能力。这是满足现实世界需求的基本要求。

这三个目标往往需要权衡。例如，追求更高的去中心化程度需要所有节点存储完整的区块链数据，但这可能会阻碍可扩展性。相反，优先考虑可扩展性可能涉及牺牲一定程度的去中心化。例如，采用分片技术来增加容量可能会导致节点之间的数据同步面临挑战，并增加安全泄露的风险。

**高安全性和高可扩展性**

追求高安全性和可扩展性（性能）通常涉及依赖有限数量的超级节点进行通信。这些超级节点拥有卓越的性能和最佳的网络条件，使它们之间能够实现超快的通信。然而，由于进入门槛高，这种方法损害了去中心化水平。

代表性区块链：BSC、EOS 和 TRON 使用仅由 21 个高性能节点维护的网络运行。

**高可扩展性和高去中心化**

追求高可扩展性（性能）和去中心化往往涉及实施众多验证节点来加强去中心化。为了提高性能，采用了加快区块生成速度或采用独特的共识机制等策略。然而，加速区块生成可能会无意中触发广泛的区块重组。同样，更复杂的共识机制可能会导致网络范围的中断和其他安全漏洞，从而危及安全性。

代表区块链：2023 年，Polygon 经历了 157 个区块的重大重组。在运营的头两年内，Solana 遇到了 8 次全网中断。

**高去中心化和高安全性**

追求高度去中心化和安全性通常涉及使用更多的节点和更公平的区块生成方法，这些方法被认为是值得信赖的。然而，在验证过程中容纳性能较低的节点并管理全球网络延迟可以减少每秒可处理的交易数量，从而影响性能。

代表性区块链：BTC 和 ETH 优先考虑卓越的安全性和去中心化。然而，这是以交易处理速度较慢为代价的，BTC 每秒处理大约 7 笔交易，而 ETH 每秒处理 10 到 20 笔交易。



#### 1.3 Layer 2

事实上，有一种方法可以克服区块链三难困境。在工程实践中，我们可以通过引入额外的层来解决这三个目标之间的权衡。添加层可以实现业务解耦，减轻原层的负担

我们引入第二层网络（Layer 2）来分担第一层网络（Layer 1）的负载，实现可扩展性。

> Layer 2是指构建在主层之上的辅助网络。它是一种采用各种技术手段来促进底层公链可扩展性的解决方案。



### 2 Layer 2历史

#### 2.1 状态通道（State channel）

> 以爱丽丝为例，她是一家咖啡店的常客。如果她每购买一杯 5 美元的咖啡就需要支付 0.5 美元的交易费，那么很快就会变得昂贵。相反，如果爱丽丝和咖啡店同意使用已签署的欠条怎么办？爱丽丝每次购买咖啡时，都会递给商店一张签名的欠条。一段时间后，咖啡店可以一次性将累积的欠条兑换付款，从而大幅降低交易成本。这个想法标志着Layer 2的起源，称为状态通道。

**状态通道** 是一种区块链扩容技术，允许用户在链下进行高频、低成本的交互，仅在开启和关闭通道时与区块链交互，从而实现近乎即时、零摩擦的交易。状态通道利用多重签名技术 [1]，允许两方将一笔资金锁定在智能合约中，从而创建一个内部通道。在这个渠道内，他们可以快速、廉价地进行大量小额交易

- [1] 多重签名技术涉及要求多个授权者共同批准交易，从而增强安全性和可靠性。

例如，公司的资金可能需要至少三名授权人员中的两名 （2/3） 的签名才能访问。在状态通道中，交易同时需要双方的签名（2/2）才能有效。多重签名还允许存在额外的 n/m 授权条件。

> **1. 核心原理**
>
> **(1) 基本流程**
>
> 1. **开启通道**：
> 	- 双方将资金锁定在链上的智能合约中（如存入 10 ETH）。
> 	- 生成初始状态（如 Alice: 6 ETH，Bob: 4 ETH），并签名确认。
> 1. **链下交易**：
> 	- 双方通过签名消息更新状态（如 Alice 转 1 ETH 给 Bob → Alice:5, Bob:5）。
> 	- **无需矿工/验证者**，交易即时生效。
> 1. **关闭通道**：
> 	- 将最终状态提交到链上，合约根据最新状态分配资金。
> 	- 若有争议（如一方提交过期状态），可挑战并惩罚作弊者。
>
> **(2) 技术关键点**
>
> - **多签名验证**：每次状态更新需双方签名。
> - **时间锁（Timelock）**：防止旧状态被恶意提交。
> - **欺诈证明（Fraud Proof）**：允许诚实方在争议期举证。
>
> **2. 为什么需要状态通道？**
>
> |     **问题**     |   **状态通道解决方案**   |
> | :--------------: | :----------------------: |
> | 链上交易慢且昂贵 | 链下交易零成本，即时确认 |
> | 区块链吞吐量有限 |    支持无限次链下交互    |
> |     隐私性差     | 交易细节仅在参与者间传递 |
>
> ------
>
> **3. 典型应用场景**
>
>  **(1) 高频微支付**
>
> - **案例**：
> 	- 按秒计费的视频流服务（如每小时支付 0.001 ETH）。
> 	- 游戏内道具实时交易。
>
>  **(2) 链下投票与治理**
>
> - **案例**：
> 	- DAO 成员在通道内频繁投票，最终结果上链。
>
> **(3) 隐私保护交易**
>
> - **案例**：
> 	- 双方链下协商价格，仅公开最终交易。
>
> **4.优缺点**
>
>  **✅ 优点**
>
> - **零延迟**：交易即时完成。
> - **零 Gas 费**：链下交互无需付费。
> - **隐私性**：交易细节不上链。
>
> **❌ 缺点**
>
> - **通道管理复杂**：需预先锁定资金。
> - **适用性有限**：仅适合固定参与者间的交互。
> - **流动性要求**：长期占用资金。



然而，这种方法会产生两个新问题：

1. Alice 与咖啡店达成的积累借条的协议仅适用于简单的转账交易。去中心化金融旨在促进更复杂的交易。那么， **我们如何才能实现复杂的交易呢？** 这致使了**侧链**解决方案的出现。
1. 虽然 Alice 可以将资金转移到咖啡店，甚至通过渠道将资金发送到超市，但**她如何向不参与闪电网络的人发送资金呢？** 这一挑战催生了**等离子体**解决方案。



#### 2.2 侧链（Sidechain）

为了克服状态通道在**处理复杂交易**方面的局限性，侧链解决方案应运而生。

**侧链** 是一种与主区块链（如以太坊、比特币）**并行**运行的独立区块链，通过双向锚定（Two-way Peg）机制与主链互联，实现资产和数据跨链转移。其核心目标是 **扩展主链功能**，同时保持一定程度的去中心化和安全性。



**侧链核心特性**

|   **特性**   |                        **说明**                         |
| :----------: | :-----------------------------------------------------: |
|  **独立性**  | 拥有自己的共识机制（如 PoA、PoS）、区块参数和智能合约。 |
| **双向锚定** |      资产可从主链锁定后映射到侧链，也能反向返回。       |
| **互操作性** |           通过桥接协议（Bridge）与主链通信。            |
|  **定制化**  |   可针对特定需求优化（如高TPS、低Gas费、隐私增强）。    |

侧链经过定制以提高性能并减轻主链上的交易负载。

- 侧链采用权益证明（**PoS**）共识机制，显著提高了区块创建速度，从而实现了可扩展性。例如，Polygon 侧链已将区块时间减少到 2 秒。
- 采用权威证明（**PoA**）共识机制，侧链可以在更少的授权超级节点进行通信的情况下实现超高性能。一个例子是 Ronin 侧链，它支持游戏资产交易。鉴于游戏中 NFT 资产更加中心化的性质，这种方法是可以接受的。



**侧链的工作原理**

**(1) 资产跨链流程（以以太坊 ↔ Polygon 为例）**

1. **存款（主链 → 侧链）**
	- 用户将 ETH 存入主链的桥合约，合约锁定资产并生成证明。
	- 侧链验证证明后，在侧链上 mint 等量的封装资产（如 WETH）。
1. **提款（侧链 → 主链）**
	- 用户销毁侧链的 WETH，提交销毁证明到主链桥合约。
	- 主链合约验证后解锁原始 ETH。

**(2) 共识机制差异**

- **主链**：通常采用高安全共识（如以太坊的 PoS）。
- **侧链**：可能使用更高效的共识（如 Polygon PoS 链的 Bor 共识）。



**为什么需要侧链？**

|      **主链瓶颈**       |               **侧链解决方案**               |
| :---------------------: | :------------------------------------------: |
| 低TPS（如以太坊15 TPS） |  侧链可达数千 TPS（如 Polygon 7,000 TPS）。  |
|         高Gas费         | 侧链交易成本极低（如 BSC 的 $0.01 交易费）。 |
|        功能限制         |  侧链可定制模块（如隐私交易、游戏专用链）。  |



 **典型侧链项目**

|      **侧链**      | **锚定主链** |         **特点**         |    **用例**    |
| :----------------: | :----------: | :----------------------: | :------------: |
|  **Polygon PoS**   |    以太坊    |      高TPS，EVM兼容      |   DeFi、NFT    |
|  **Gnosis Chain**  |    以太坊    | 注重去中心化，xDai 合并  |   支付、DAO    |
| **Liquid Network** |    比特币    |  联邦桥接，支持机密交易  | 比特币快速结算 |
|     **Ronin**      |    以太坊    | Axie Infinity 游戏专用链 |     GameFi     |



**侧链的安全风险**

**(1) 桥接攻击**

- **案例**：2022 年 Ronin 桥被盗 6.25 亿美元（私钥泄露）。
- **防御**：使用多重签名、零知识证明桥（如 zkBridge）。

**(2) 共识中心化**

- 部分侧链（如 BSC）由少数节点控制，可能被操纵。

**(3) 资产锚定失效**

- 若桥合约漏洞导致双向锚定断裂，侧链资产可能脱钩。



侧链通过 **牺牲部分主链安全性** 换取高性能和低成本，是区块链生态的重要扩容手段。用户在使用时需权衡 **效率、成本与风险**，优先选择经过时间验证的侧链和桥接协议。



#### 2.3 Plasma

状态通道和侧链解决方案的实际实施暴露了某些缺陷，导致了 Plasma 解决方案的提出，并受到了广泛关注。它解决了以前方法中发现的两个问题：

- 无法**向非参与者发送资金** ：未连接到 Plasma 链的账户可以接收资金并独立提取到 Layer 1。
- **<u>无法继承 Layer 1 安全性</u>** ：Plasma 定期向主链提交信息以继承 Layer 1 安全性。

**Plasma** 是以太坊创始人 **Vitalik Buterin** 和 **Joseph Poon** 在 2017 年提出的一种 **区块链扩容方案**，旨在通过构建多层链下结构（“子链”），将大部分交易从<u>主链（如以太坊）卸载到子链上处理</u>，从而显著提升吞吐量并降低交易成本。尽管后期因技术复杂性被 Rollup 取代，但 Plasma 的设计思想仍对 Layer2 发展影响深远。



**Plasma 的核心思想**

 **(1) 分层结构**

- **主链（Root Chain）**：以太坊等底层区块链，负责最终结算和争议仲裁。
- **子链（Plasma Chain）**：独立的链下区块链，处理高频交易，定期向主链提交状态承诺（Merkle Root）。

**(2) 关键机制**

- **状态承诺**：子链将交易批量打包后，生成 Merkle Root 并提交到主链。
- **欺诈证明（Fraud Proof）**：若子链作恶（如篡改交易），用户可提交证明至主链，触发惩罚和状态回滚。
- **资金退出（Exit）**：用户需通过主链的“退出游戏”机制安全取回资金。



**Plasma 的工作原理**

**(1) 存款（主链 → Plasma 链）**

1. 用户将资产（如 ETH）锁定在主链的 Plasma 智能合约中。
1. Plasma 链生成对应的资产凭证（如 1:1 映射的代币）。

**(2) 链下交易（Plasma 链内）**

- 用户在 Plasma 链上自由交易（如转账、支付），无需主链确认。
- 子链区块生产者（Operator）定期将交易打包，并提交 Merkle Root 到主链。

**(3) 提款（Plasma 链 → 主链）**

1. 用户发起提款请求，启动 **7天挑战期**（类似 Optimistic Rollup）。
1. 若无争议，主链释放锁定的资产；若有欺诈，提交证明取消提款。



**Plasma 的优缺点**

**✅ 优点**

- **高吞吐量**：交易在子链处理，主链仅存储压缩数据。
- **低成本**：Gas 费由子链承担，主链仅需极低开销。
- **安全性继承**：依赖主链的**欺诈证明机制**确保资金安全。

 **❌ 缺点**

- **退出延迟**：提款需等待挑战期（7天），流动性受限。
- **数据可用性问题**：**<u>若子链运营商隐藏数据，用户无法构造欺诈证明。</u>**
- **通用性差**：难以支持复杂智能合约（仅适合支付等简单逻辑）。



**为什么 Plasma 逐渐被淘汰？**

1. **数据可用性问题**：用户无法获取子链完整数据时，无法挑战无效交易。
1. **用户体验差**：提款延迟长，且需主动监控欺诈。
1. **Rollup 的崛起**：
	- **ZK Rollup** 通过零知识证明解决数据问题。
	- **Optimistic Rollup** 保留欺诈证明但强制数据上链，平衡安全与成本。



#### 2.4 Rollup

虽然 Plasma 可能没有获得广泛接受，但其原理的某些元素在随后的第 2 层解决方案中得到了保留和进一步发展。

鉴于 Plasma 的主要挑战集中在**数据可用性**上，特别是审计师在访问交易数据进行验证时面临的障碍，如果我们不仅上传根证明，而且将所有重要的交易数据合并到Layer 1会怎样？这能否为问题提供可行的解决方案？

这种思路带来了目前最实用的 Layer 2 扩容解决方案：Rollup。术语“Rollup”是指将一定时期内的交易进行合并、打包并定期上传到主网的捆绑过程。目前主流的 Rollup 方案大致可以分为两大类：**Optimistic Rollup** 和**零知识 Rollup**。



**Rollup 的核心思想**

**(1) 链下执行 + 链上验证**

- **链下**：交易在 Rollup 链（Layer2）上执行。
- **链上**：交易数据（或有效性证明）提交到主链（Layer1），确保安全性。

**(2) 数据压缩**

- Rollup 将多笔交易压缩为 **单个批次**，减少链上存储开销。
- 例如：1000 笔交易 → 1 个 Rollup 区块 → 1 笔主链交易。



**Rollup 的两种类型**

**(1) Optimistic Rollup（乐观汇总）**

- **假设交易合法**，默认不验证，依赖 **欺诈证明（Fraud Proof）** 争议机制。
- **特点**：
	- 兼容 EVM（如 Arbitrum、Optimism）。
	- 提款需等待 **7天挑战期**。
	- 成本低，适合通用智能合约。

**工作流程**：

1. 交易在 Layer2 执行。
1. 排序器（Sequencer）将交易数据（Calldata）提交到主链。
1. 若有欺诈，验证者在挑战期内提交证明，回滚错误状态。

**(2) ZK Rollup（零知识证明汇总）**

- 每批交易生成 **零知识证明（ZK-SNARK/STARK）**，主链即时验证。
- **特点**：
	- 无需挑战期，提款即时到账。
	- 证明生成计算量大，早期难兼容 EVM（现 zkEVM 已突破）。
	- 隐私性更强（可隐藏交易细节）。

**工作流程**：

1. 交易在 Layer2 执行。
1. 生成有效性证明（Proof）并提交到主链。
1. 主链验证证明后，更新状态。



**Rollup 的优缺点**

**✅ 优点**

- **高扩展性**：吞吐量提升百倍。
- **安全性**：依赖主链验证，抗审查。
- **低成本**：Gas 费仅为链上的 1/10~1/100。

**❌ 缺点**

- **Optimistic 的延迟**：提款需等待 7 天（可通过流动性提供商缓解）。
- **ZK 的硬件需求**：证明生成需要高性能服务器。
- **中心化风险**：排序器可能被垄断（逐步去中心化中）。



> ### **零知识证明（Zero-Knowledge Proof, ZKP）详解**
>
> **零知识证明** 是一种密码学技术，允许一方（证明者）向另一方（验证者）**证明某个陈述的真实性**，而无需透露任何额外信息。其核心思想是：**“我知道一个秘密，但我不会告诉你秘密是什么”**。
>
> 在区块链领域，ZKP 是隐私保护（如 Zcash）和扩容（如 ZK Rollup）的核心技术。
>
> #### **1. 零知识证明的三大核心特性**
>
> 1. **完备性（Completeness）**：
> 	- 如果陈述为真，诚实验证者一定会被说服。
> 1. **可靠性（Soundness）**：
> 	- 如果陈述为假，作弊的证明者无法欺骗验证者。
> 1. **零知识性（Zero-Knowledge）**：
> 	- 验证者除了“陈述为真”外，无法获取任何其他信息。
>
> #### **2. 经典类比：洞穴寓言**
>
> 假设有一个环形洞穴，中间有一道需要密码才能打开的门。
>
> - **证明者**知道密码，想向**验证者**证明这一点，但不想泄露密码。
> - **过程**：
> 	1. 验证者站在洞口，随机要求证明者从左侧（A）或右侧（B）进入。
> 	1. 证明者无论从哪边进入，都能用密码开门并从另一侧出来。
> 	1. 重复多次后，验证者确信证明者确实知道密码，但始终不知道密码是什么。
>
> #### 3. 零知识证明的两种主要类型
>
> **(1) 交互式零知识证明（Interactive ZKP）**
>
> - 需要多轮通信（如洞穴寓言中的多次进出）。
> - **缺点**：效率低，不适合区块链。
>
> **(2) 非交互式零知识证明（Non-Interactive ZKP, NIZK）**
>
> - 证明者生成单次证明，验证者可随时检查。
> - **区块链常用**：如 ZK-SNARKs、ZK-STARKs。
>
> #### **4. 区块链中的 ZKP 技术**
>
>  **(1) ZK-SNARKs（简洁非交互式知识论证）**
>
> - **特点**：
> 	- 证明体积小（约 200 字节），验证速度快。
> 	- 需要“可信设置”（Trusted Setup），存在潜在风险。
> - **应用**：
> 	- **Zcash**（隐私转账）。
> 	- **zkSync**（ZK Rollup）。
>
>  **(2) ZK-STARKs**
>
> - **特点**：
> 	- 无需可信设置，抗量子计算。
> 	- 证明体积较大（约 100 KB），但验证速度仍快。
> - **应用**：
> 	- **StarkNet**（以太坊 Layer2）。
>
> **(3) Bulletproofs**
>
> - **特点**：
> 	- 无需可信设置，适合范围证明（如 Confidential Transactions）。
> - **应用**：
> 	- **Monero**（隐私币）。
>
> #### **5. 零知识证明的区块链应用**
>
> **(1) 隐私保护**
>
> - **匿名交易**：Zcash 使用 ZK-SNARKs 隐藏发送方、接收方和金额。
> - **身份验证**：证明年龄 >18 岁，而无需透露出生日期。
>
>  **(2) 扩容（ZK Rollup）**
>
> - <u>将数千笔交易打包，生成 ZKP 证明提交到主链，验证者只需检查证明即可确认有效性。</u>
> - **代表项目**：zkSync、StarkNet、Scroll。
>
> **(3) 去中心化存储验证**
>
> - 证明文件已正确存储，而无需下载全部数据（如 Filecoin）。
>
> #### **6. 零知识证明的优缺点**
>
> **✅ 优点**
>
> - **隐私性**：隐藏敏感数据（如交易详情）。
> - **扩展性**：减少链上计算负担（如 ZK Rollup）。
> - **安全性**：数学上无法伪造证明。
>
> **❌ 挑战**
>
> - **计算密集型**：生成证明需要高性能硬件。
> - **可信设置（ZK-SNARKs）**：初始参数若泄露，可能伪造证明。
> - **开发门槛高**：需要密码学专业知识。



#### 2.5 Layer 2 对比

**核心特性对比**

|   **维度**   |     **State Channel**     |    **Sidechain**     |    **Plasma**     |        **Rollup**        |
| :----------: | :-----------------------: | :------------------: | :---------------: | :----------------------: |
|  **安全性**  |       依赖最终结算        |     依赖侧链共识     | 依赖主链欺诈证明  |      继承主链安全性      |
| **交易速度** |       即时（链下）        | 较快（依赖侧链性能） | 较快（链下执行）  |     较快（链下执行）     |
|   **成本**   | 链下免费，仅开/关通道付费 |   低（侧链Gas费）    |       极低        |     极低（数据压缩）     |
| **去中心化** |  需预存资金，参与者固定   |   可变（PoA/PoS）    |    依赖运营商     |       逐步去中心化       |
| **数据存储** |      仅最终状态上链       |     独立链上存储     | 仅提交Merkle Root | 交易数据全上链（Rollup） |
| **适用场景** |  高频小额支付（如游戏）   | 独立生态（如GameFi） | 简单支付/资产转移 |       通用智能合约       |



**工作原理对比**

**(1) State Channel（状态通道）**

- **流程**：
	1. 双方锁定资金在主链。
	1. 链下无限次交易（仅双方签名）。
	1. 最终结算状态上链。
- **例子**：比特币闪电网络、以太坊的Raiden Network。

**(2) Sidechain（侧链）**

- **流程**：
	1. 资产通过桥锁定在主链，映射到侧链。
	1. 侧链独立运行（自有共识机制）。
	1. 提款时反向桥接回主链。
- **例子**：Polygon PoS链、Ronin（Axie Infinity侧链）。

 **(3) Plasma**

- **流程**：
	1. 资产锁定在主链Plasma合约。
	1. 子链处理交易，定期提交Merkle Root到主链。
	1. 提款需挑战期（防欺诈）。
- **例子**：早期OMG Network（已转向Rollup）。

 **(4) Rollup**

- **流程**：
	1. 交易在链下执行并压缩。
	1. 数据批量提交到主链（Optimistic需欺诈证明，ZK需有效性证明）。
- **例子**：Optimism（OP）、Arbitrum、zkSync。



**安全性对比**

|     **方案**      |  **安全模型**  |            **主要风险**             |
| :---------------: | :------------: | :---------------------------------: |
| **State Channel** | 依赖参与者诚实 |      对手方离线时资金可能锁定       |
|   **Sidechain**   |  依赖侧链共识  |  桥接攻击（如Ronin被盗6.25亿美元）  |
|    **Plasma**     |  主链欺诈证明  | 数据不可用性问题（运营商隐藏数据）  |
|    **Rollup**     |  继承主链安全  | Optimistic的7天延迟，ZK的证明中心化 |



**优缺点总结**

|     **方案**      |          **优点**          |            **缺点**            |
| :---------------: | :------------------------: | :----------------------------: |
| **State Channel** | 零延迟，零Gas费，隐私性好  |   仅限固定参与者，需预存资金   |
|   **Sidechain**   |      高性能，灵活定制      |    安全性依赖侧链，桥接风险    |
|    **Plasma**     |      高吞吐量，低成本      |   退出延迟长，不支持复杂逻辑   |
|    **Rollup**     | 继承主链安全，支持智能合约 | Optimistic有延迟，ZK开发门槛高 |



**未来趋势**

- **Rollup 主导**：Optimistic 和 ZK Rollup 成为以太坊扩容主流。
- **State Channel 小众化**：仅用于特定场景（如微支付）。
- **Sidechain 专用化**：游戏、社交等垂直领域。
- **Plasma 淘汰**：被 Rollup 取代（因数据可用性问题）。



### 3 Rollup详情







## 区块链岗位全景图

### 一、技术岗

#### 1. 前端工程师

1. **主要职责：**

	- 设计和开发基于区块链技术的前端应用，支持去中心化平台的交互。
	- 通过 React、Vue 等框架实现高效的用户界面，支持钱包连接、交易签名、信息验证等功能。
	- 集成并优化智能合约的前端交互，确保链上数据与用户界面的无缝连接。
	- 与后端团队协作，基于 GraphQL 或 RESTful API 获取链上和链下数据。
	- 持续优化前端性能，提升用户体验，确保在不同设备和浏览器上的兼容性。
	- 关注 Web3 技术趋势，参与技术评审与分享，不断优化产品架构与代码质量。

1. **职位要求：**

	- 本科及以上学历，计算机科学或相关专业，具备扎实的计算机基础。
	- 精通 HTML、CSS、JavaScript，熟悉 React、Vue 等前端框架，能够独立构建复杂的 UI 界面。
	- 熟悉 Web3.js、Viem、Metamask 等 Web3 技术栈，能够与智能合约进行交互。（现在 Ethers.js / Web3.js 已经不怎么使用了，大家现在基本上都是用的 Viem）
	- 了解常用的区块链网络，如以太坊、Solana 等，具备 Dapp 开发经验者优先。
	- 熟悉 Git 版本管理工具，具备良好的代码编写规范，注重代码可维护性。
	- 良好的沟通能力和团队协作精神，能够在快速发展的环境中高效工作。
	- 具有良好的问题解决能力，能在面对复杂的技术难题时，提出创新的解决方案。
	- 有开源项目或 Web3 相关项目的经验优先。

	```bash
	# 常用技术栈
	- HTML5
	- CSS3
	- JavaScript (ES6+)
	- React / Vue
	- TypeScript
	- Next.js
	- Ethers.js / Web3.js / Viem
	```

#### 2. 后端工程师

1. **主要职责：**

	- 设计、开发和维护去中心化应用（Dapp）的后端服务，包括链上数据交互、智能合约集成和事务处理。
	- 与前端团队合作，确保前后端数据交互的高效性和稳定性，支持多种 Web3 钱包（如 Metamask）的集成。
	- 基于 Node.js、Python 等技术栈构建高性能的 RESTful 或 GraphQL API，以支持 Web3 平台的需求。
	- 与智能合约团队合作，确保智能合约与后端服务的无缝连接，优化链上数据的读取和写入效率。
	- 优化后端性能，确保系统的高可用性、高吞吐量和低延迟，满足高并发访问需求。
	- 定期进行系统架构和代码的评审，不断提升代码质量与技术标准。
	- 参与 Web3 技术的前沿研究，保持对新兴区块链技术的学习和应用，推动公司技术迭代。

1. **职位要求：**

	- 本科及以上学历，计算机科学或相关专业，具备扎实的计算机基础。
	- 精通 Node.js、Go、Python 等后端开发语言，具有构建高并发、分布式系统的经验。
	- 熟悉 Viem、Web3.js、Ethers.js 等 Web3 工具，能够与区块链进行交互并处理链上数据。
	- 具备 RESTful API 或 GraphQL 开发经验，能够设计高效的 API 服务，支持前端与区块链的交互。
	- 熟悉数据库技术，具备 MySQL、PostgreSQL 或 NoSQL 数据库的开发与优化经验。
	- 对智能合约有一定的了解，具备与智能合约交互、读取链上数据等相关经验。
	- 熟悉消息队列（如 RabbitMQ、Kafka）及事件驱动架构，能够处理异步事务。
	- 熟悉容器化技术（如 Docker、Kubernetes），具备 CI/CD 经验者优先。
	- 良好的代码编写规范与文档写作能力，具备良好的团队协作精神和沟通能力。
	- 有 Web3 项目开发经验或开源贡献者优先。

	```bash
	# 常用技术栈
	- Node.js / Go / Python
	- Viem / Web3.js / Ethers.js
	- RESTful API / GraphQL
	- MySQL / PostgreSQL
	- Docker / Kubernetes
	```

#### 3. 智能合约工程师

1. **主要职责：**

	- 设计、开发和部署智能合约，确保合约在区块链网络上的安全性、可靠性和高效性。
	- 使用 Solidity 编写智能合约，涵盖各类去中心化应用的需求，如 NFT、DeFi、DAO 等。
	- 与团队合作，定义智能合约的功能需求，并根据需求设计合适的智能合约架构。
	- 部署智能合约至区块链网络（如以太坊、Polygon、Arbitrum、Base 等），并确保合约的高效执行。
	- 编写智能合约的单元测试，进行代码审计与安全性测试，确保合约代码无漏洞，避免潜在的安全风险。
	- 优化智能合约性能，减少 Gas 费用。
	- 研究和应用最新的区块链技术和智能合约最佳实践，推动技术的持续进步。
	- 与前端和后端开发团队紧密协作，确保智能合约与其他系统组件的顺畅集成。
	- 为团队成员提供智能合约相关的技术支持和指导，推动团队的技术提升。

1. **职位要求：**

	- 本科及以上学历，计算机科学或相关专业，具备扎实的计算机基础。
	- 3 年以上智能合约开发经验，熟练使用 Solidity 或类似的智能合约开发语言。
	- 熟悉 Ethereum、Polygon、Arbitrum、Base 等主流区块链平台，能够在这些平台上部署和维护智能合约。
	- 了解智能合约开发的安全性问题，具备智能合约审计和漏洞修复经验，熟悉常见的攻击模式（如重入攻击、溢出、权限管理等）。
	- 熟悉区块链的基本原理，理解 Gas 费用、交易确认、区块链共识机制等概念。
	- 熟练使用 Foundry、Hardhat、Remix 等智能合约开发框架，具备项目开发、测试与部署经验。
	- 具备一定的 Viem、Web3.js、Ethers.js 等 Web3 工具使用经验，能够与前端或其他系统进行无缝集成。
	- 熟悉 IPFS、NFT、Token 标准（ERC-20、ERC-721、ERC-1155 等）及去中心化身份（DID）等 Web3 相关技术。
	- 具有良好的代码编写规范，注重代码的可读性和可维护性。
	- 良好的沟通能力和团队协作精神，能够在快速发展的环境中有效工作。

	```bash
	# 常用技术栈
	- Solidity
	- Remix
	- Foundry / Hardhat
	- Phalcon / Tenderly
	- Yul
	```

### 二、非技术岗

#### 1. 产品与运营

1. 职位描述：
	- 在 Web3 产品生命周期中负责协调产品发布、用户反馈收集和持续改进流程，以提升用户体验和产品迭代效率。
	- 执行以用户获取、留存和参与度提升为目标的增长战略，并监控实施效果。
	- 与产品、技术、市场及合规团队紧密合作，确保产品上市（go-to-market）策略与各部门需求保持一致。
	- 持续分析运营数据，跟踪关键绩效指标（KPIs），并根据数据提出优化建议。
1. 职位要求：
	- 熟悉产品上线（Go-to-market）全流程，擅长跨部门资源协调与项目推进。
	- 具备扎实的数据分析能力，能熟练使用 SQL、Excel 或其他数据工具进行数据统计和洞察提炼。

#### 2. 社区管理

1. 职位描述

	- 构建并管理 Telegram、Twitter（X）、Discord 等社交平台的社区，实现持续的用户互动与增长；
	- 组织线上 AMA（问答）、活动、竞赛等形式多样的社区互动，以提升用户活跃度和品牌粘性；
	- 跟踪社区健康度指标和情感分析，定期向管理层汇报洞察与优化建议；
	- 与营销团队协作，制定并执行内容日历，发布社区公告和运营手册。

1. 职位要求

	- Web3、DAO 或 NFT 社区管理经验，深刻理解去中心化应用生态；
	- 出色的文案撰写与沟通能力，能够有效引导社区讨论并快速响应用户反馈；
	- 熟练使用社区数据分析工具，能够监测并解读用户行为与舆情动态；
	- 具备活动策划与执行能力，能够独立组织线上及线下社区活动。

	```bash
	# 常用平台
	- Telegram
	- Twitter (X)
	- Discord
	```

#### 3. 研究分析

1. 职位描述

	- 收集、整理并分析 Web3 行业市场与用户数据，编写可行性研究报告，为产品与运营提供决策支持；
	- 跟踪区块链协议技术演进及生态动态，撰写深度研究报告或白皮书；
	- 进行竞争对手分析，评估市场趋势与用户行为模式，为战略规划提供数据驱动的建议；
	- 支持项目的加密经济模型设计与博弈论分析，以保证项目的经济激励合理性。

1. 职位要求

	- 熟练使用 Excel、SPSS 或 Python 等数据分析工具，具备定量和定性研究方法经验；
	- 深入了解区块链生态、DeFi 协议及加密经济学原理；
	- 优秀的报告撰写与演示能力，能够清晰传达研究结论与建议；
	- 精通链上数据分析工具（Glassnode、Token Terminal）。

	```bash
	# 常用工具
	- Excel
	- SPSS
	- Python
	- Glassnode
	- Token Terminal
	```

# 2025-08-05

# 

| 今日学习内容                 |
| ---------------------------- |
| 以太坊概览                   |
| unphishable钓鱼攻防挑战-初级 |



## 以太坊概览

### 以太坊介绍

以太坊（Ethereum）被称为“区块链 2.0”，它不仅是一种加密货币（以太币 ETH），更是一台支持智能合约的“全球共享计算机”。通过代码自动执行规则，开发者可以在区块链上构建去中心化金融（DeFi）、数字艺术品（NFT）和去中心化自治组织（DAO）等创新应用，无需依赖银行或中心化平台。

以太坊的核心创新在于 **智能合约**（Smart Contracts） 。<u>智能合约是存储在区块链上的可执行代码，能够在满足预设条件时自动执行操作，无需人工干预</u>。这一特性使得以太坊不仅是数字货币的载体，更是构建去中心化应用（Dapps）、去中心化金融（DeFi）、非同质化代币（NFT）等生态系统的基础设施。



### Ethereum 与 Bitcoin 的差异

|      维度      | 比特币（Bitcoin）                                            | 以太坊（Ethereum）                                          |
| :------------: | :----------------------------------------------------------- | :---------------------------------------------------------- |
| **目标与定位** | 去中心化的**数字货币**，强调安全、稳定和稀缺性（总量 2100 万枚） | 去中心化平台，支持**智能合约**和 Dapps，定位为“区块链 2.0”  |
|  **编程能力**  | 脚本语言有限，仅支持简单的交易验证逻辑                       | 图灵完备的编程语言（如 Solidity），可开发复杂智能合约       |
|  **共识机制**  | 工作量证明（PoW），矿工通过算力竞争记账权                    | 从 PoW 转向权益证明（PoS），通过 The Merge 实现能源效率优化 |
|  **交易速度**  | 每 10 分钟生成一个区块，交易确认较慢                         | 区块时间约 12 秒，交易确认更快，适合高频应用                |
|  **经济模型**  | 总量固定，强调抗通胀属性                                     | 供应灵活，通过 EIP-1559 等机制可能呈现通缩趋势              |



### 以太坊的定位与演进

#### 以太坊1.0（PoW阶段）

这个阶段就是挖矿，和比特币的机制相同，同样的消耗电力和处理交易速度慢、费用高



#### 以太坊2.0与The Merge：从双链并行到完美合并

> **The Merge 完整故事**：
>
> 1. **2020 年 12 月：信标链启动**
>
> 	以太坊团队首先创建了一条全新的**信标链（Beacon Chain）**，专门运行 PoS 共识机制。此时：
>
> 	- 以太坊主网继续使用 PoW 挖矿
> 	- 信标链独立运行 PoS 验证
> 	- 两条链并行存在，互不干扰
>
> 1. **2022 年 9 月：历史性合并**
>
> 	2022 年 9 月 15 日，**The Merge** 发生：
>
> 	- 以太坊主网“关闭”了 PoW 挖矿引擎
> 	- 将共识机制“插接”到信标链的 PoS 系统
> 	- 从此，以太坊主网由信标链保护安全
>
> 1. **合并后的新架构**
>
> 	现在的以太坊实际上是两层结构：
>
> 	- **执行层**：处理交易、智能合约（原主网）
> 	- **共识层**：管理验证者、确定区块顺序（信标链）

**PoS机制详解：**

**验证者如何工作**：

- **准入门槛**：质押 32 ETH 成为验证者
- **工作方式**：系统随机选择验证者来提议和验证区块
- **奖励机制**：验证者获得新发行的 ETH + 交易费用（gas）
- **惩罚机制**：作恶者质押的 ETH 被销毁（Slashing）

**相比 PoW 的优势**：

- **能耗降低 99.95%**：无需大量电力和硬件
- **经济安全性**：攻击成本约需控制全网 67% 的质押 ETH（价值数百亿美元）
- **最终确定性**：区块确认更快、更可靠



#### 未来升级路线

暂时学不懂



### 以太坊生态概览：L1、L2、Sidechains 等

以太坊的生态系统由多层架构组成，包括 **L1（主网）、L2（二层扩展解决方案）、侧链（Sidechains）** 等，共同支持高吞吐量和低费用的交易处理。

1. Layer 1（L1）
	- **以太坊主网**：核心区块链，负责最终安全性与共识。
	- **EVM**：以太坊虚拟机，执行智能合约代码。
	- **账户系统**：外部账户（EOA）与合约账户（CA）共同构成网络基础。
1. Layer 2（L2）
	- Rollup：通过将交易批量处理后提交至 L1，降低 Gas 费。
		- **Optimistic Rollup**：假设交易合法，仅在争议时验证。
		- **ZK Rollup**：通过零知识证明验证交易，无需链上争议。
1. **侧链（Sidechains）**：独立运行的链，通过**桥接**与主网交互。
1. **以太坊生态分层架构**



> **1. Optimistic Rollup（乐观汇总）**
>
> **核心思想**
>
> “默认信任，争议时验证” —— 假设所有提交到链上的交易是合法的，仅在有人提出质疑时进行验证。
>
> **工作流程**
>
> 1. **交易打包**：
> 	- 用户将交易发送给排序器（Sequencer），排序器批量打包交易并生成状态根（State Root），提交到主链（如以太坊）。
> 	- 主链仅存储交易数据（Calldata），不立即验证。
> 1. **挑战期（Fraud Proof）**：
> 	- 提交后有一段挑战期（通常 **7天**），任何人可质疑交易有效性。
> 	- 如果发现无效交易，验证者提交欺诈证明（Fraud Proof），触发链上计算并回滚错误状态。
> 1. **最终确认**：
> 	- 若挑战期内无争议，交易最终确认。
>
>  **优点**
>
> - **兼容性强**：支持任意智能合约（EVM 兼容）。
> - **Gas 成本低**：仅需提交数据，无需复杂计算。
> - **开发门槛低**：无需零知识证明（ZKP）专业知识。
>
>  **缺点**
>
> - **提款延迟**：用户需等待挑战期结束（7天）才能提取资金。
> - **中心化风险**：依赖排序器（Sequencer）快速打包交易。
> - **安全依赖诚实多数**：若无人监控并提交欺诈证明，恶意交易可能通过。
>
>  **代表项目**
>
> - **Optimism**、**Arbitrum**（EVM 兼容，主打 DeFi 生态）。
>
> ------
>
>  **2. ZK Rollup（零知识证明汇总）**
>
> **核心思想**
>
> “数学证明，即时验证” —— 每批交易通过零知识证明（ZKP）验证有效性，无需争议期。
>
> **工作流程**
>
> 1. **交易打包**：
> 	- 排序器收集交易，生成状态变更和有效性证明（ZK-SNARK/STARK）。
> 	- 将证明和状态根提交到主链。
> 1. **链上验证**：
> 	- 主链验证 ZKP，确保交易合法后立即更新状态。
> 	- 无需挑战期，交易即时确认。
>
> **优点**
>
> - **即时最终性**：无提款延迟，资金可立即提取。
> - **更高安全性**：依赖数学证明，无需诚实多数假设。
> - **隐私性**：可选择性隐藏交易细节（如 Zcash 风格隐私交易）。
>
> **缺点**
>
> - **计算资源消耗大**：生成 ZKP 需要高性能硬件。
> - **EVM 兼容性有限**：早期仅支持简单逻辑，现逐步改进（如 zkEVM）。
> - **开发复杂度高**：需密码学专业知识。
>
> **代表项目**
>
> - **zkSync**、**StarkNet**、**Scroll**（专注 zkEVM 兼容）。



以太坊生态可以分为以下几个层次：

1. **应用层（Application Layer）**

用户直接交互的应用和界面：

- **DeFi 应用**：Uniswap（去中心化交易所）、Aave（借贷协议）、Compound（借贷协议）
- **NFT 平台**：OpenSea、Foundation、SuperRare
- **钱包应用**：MetaMask、Coinbase Wallet、Rainbow
- **DAO 工具**：Snapshot、Aragon、Colony

2. **协议层（Protocol Layer）**

以太坊的核心基础设施：

- **共识层客户端**：Prysm、Lighthouse、Nimbus、Teku
- **执行层客户端**：Geth、Nethermind、Erigon、Besu
- **核心协议**：EVM、状态管理、Gas 机制

3. **扩展层（Scaling Layer）**

提升性能和降低成本的解决方案：

- **Layer 2 Rollups**：Arbitrum、Optimism、Polygon zkEVM、zkSync Era
- **侧链**：Polygon PoS、xDAI（Gnosis Chain）
- **状态通道**：Lightning Network for Ethereum



### 以太坊文化与价值观

核心价值观

1. **去中心化治理（Decentralization）**
	- 没有单一的控制者或权威机构
	- 社区通过公开讨论和 EIP（以太坊改进提案）机制共同决策
	- 验证者遍布全球，防止权力集中
1. **无需许可与开放性（Permissionless & Open）**
	- 任何人都可以使用、开发、部署智能合约
	- 开源代码，透明可审计
	- 无身份、地域、财富限制的参与门槛
1. **抗审查性（Censorship Resistance）**
	- 交易和智能合约不受政府或机构干预
	- 通过分布式验证确保网络弹性
	- 支持言论自由和经济自由
1. **密码朋克精神（Cypherpunk Ethos）**
	- 代码即法律：用算法和数学构建信任
	- 密码学保护隐私和自主权
	- 技术驱动的社会变革，而非政治手段
1. **公共物品导向（Public Goods Orientation）**
	- 优先考虑生态系统整体利益
	- 支持开源项目和基础设施建设
	- 通过各种资助计划推动创新
1. **可持续发展理念**
	- The Merge 体现了对环境责任的承诺
	- 长期主义思维，注重技术的可持续演进
	- 平衡创新速度与网络稳定性



### 以太坊核心机制：从账户到执行的完整链路

以太坊三个关键机制：**账户系统**、**Gas 模型** 和 **以太坊虚拟机（EVM）**。

（1）账户系统：你的数字身份

**账户系统** 包含由私钥控制的 **外部账户（EOA）** 和由智能合约代码控制的 **合约账户（CA）** 。

想象你第一次接触以太坊——你需要一个“数字钱包”来参与网络。这个钱包的核心是 **外部账户（EOA）** ，它由一对密钥（私钥和公钥）生成，就像银行账户的密码和账号。私钥是你控制账户的“钥匙”，必须严格保密；公钥通过加密算法生成一个唯一的地址（如 `0xAbc...123`），你可以把它分享给朋友接收转账。

除了用户控制的 EOA，还有 **合约账户（CA）** 。它们不像 EOA 那样受私钥控制，而是由代码驱动。比如，你部署一个智能合约（如一个 NFT 市场），区块链会自动生成一个 CA 地址（如 `0xDef...456`）。这个账户不能主动发起交易，只能通过 EOA 触发——比如你点击“购买 NFT”按钮时，EOA 向 CA 发送交易，CA 的代码自动执行出货逻辑。

每个账户都包含四个关键字段：

- **Nonce**：防止重复交易的计数器（EOA 记录发送次数，CA 记录创建合约次数）。
- **余额**：账户持有的 ETH 数量（单位为 Wei）。
- **CodeHash**：EOA 为空哈希，CA 存储合约字节码的哈希值。
- **StorageRoot**：记录账户数据的 Merkle 树根哈希（如 NFT 归属关系）。

（2）Gas 模型：交易的燃料费

当你用钱包使用自己的 EOA 发起一笔交易（比如转账或操作合约），这件事当然 **不会是免费的**，你需要支付“燃料费”——也就是 **Gas**。

Gas 费用 = **用多少 × 每单位多少钱**，就像你打车一样：

- **Gas Limit（限额）**：你最多愿意“烧”多少燃料。 比如你觉得最多可能需要 15 万单位，就设置 150,000。
- **Gas Price（单价）**：每单位燃料多少钱，用 Gwei 表示（1 Gwei = 0.000000001 ETH）。 网络越拥堵，价格越贵，就像打车高峰期加价。

所以，**总费用 = Gas Limit × Gas Price**。

```
假设你设置 Gas Limit 为 150 000，Gas Price 为 100 Gwei，总费用就是 0.015 ETH。如果实际消耗 120 000 Gas，剩余 30 000 Gas × 100 Gwei = 0.003 ETH 会退还给你。
```

Gas 的存在有两个目的：

- **激励矿工/验证者**：你给得越多（Gas Price 越高），他们越愿意优先处理你的交易。
- **防止资源滥用**：如果有人想让合约死循环，Gas 会用光，交易失败，系统不会被拖垮。

<u>在 EIP-1559 升级后，Gas Price 被拆分了：</u>

以前，Gas Price 全部给矿工。 现在，分成两部分：

- **基础费用（Base Fee）**：每个区块都会有，自动计算，直接销毁（直接消失）来帮助 ETH 通缩，毕竟物以稀为贵。
- **小费（Tip）**：你额外加的钱，用来鼓励矿工优先处理你。

（3）以太坊虚拟机（EVM）：代码的执行引擎

EVM（Ethereum Virtual Machine）是 **以太坊的“大脑”**，是专门用来**运行智能合约的虚拟计算机**。它运行在每个节点上，确保整个网络在处理代码时，**结果都一致、可信任**。

EVM 的核心特点：

- **图灵完备**：就像真正的电脑一样，EVM 可以执行各种逻辑，比如 if 判断、循环等。
- **全球同步**：每个矿工/节点都会自己执行一遍合约代码，保证结果一样。
- **隔离安全**：EVM 把合约“关”在一个小房间里运行，不允许它乱访问用户的数据和网络，保护隐私和安全。





## unphishable钓鱼攻防挑战

挑战地址：[unphishable.io](http://unphishable.io/) 

### 初级

### No.0x0001 Web3 钱包设置指南

第一关就是创建一个MetaMask的小狐狸钱包，在这之前我就有一个钱包里。这关也重点强调了助记词的重要性。非常重要！！千万不能泄露，接下来一关我就中招了！

安全存储助记词方法附上：

- 将其写在纸上（多份副本）
- 存放在安全、防水、防火的地方
- 考虑使用金属备份解决方案以提高耐久性
- 切勿以数字方式存储（不要拍照、不要云存储、不要电子邮件）
- 切勿与任何人分享，包括"支持人员"

还有一个点就是要领取Holesky的测试币，找了几个水龙头都不是很好弄，看群里的小伙伴都是用Google水龙头领测试币，但是我的就用不了，Google的水龙头搞抽象，无法领取，问了群里大佬说他是这样的。不过最后找到一个好用的水龙头，不仅之前领了Sepolia的测试币，现在也领了Holesky的测试币，附上地址

https://www.alchemy.com/faucets/ethereum-sepolia

但是应该是需要eth主网上有一定量的以太币（0.001 ETH）才能在这里领测试币

领完测试币，连接钱包，这关就结束了



### No.0x0002 钱包恢复助手

这是一个ai对话界面，功能是钱包恢复，然后让你输入你的助记词，我看到了这样的字样 “请放心，这是一个安全的环境，您的信息将被加密处理。”然后还真的去找了我的助记词给他添上了，我还特意用ai查了这个平台是否靠谱，结果输入进入，告诉我我被钓鱼了。身为一个学安全的人，我自认为安全意识还是有点的，但是也可能是我还没意识到这个挑战是在做个什么，所以被骗+1。

所以！！永远不要向任何人透露您的助记词，无论他们声称是谁



### No.0x0003 USDC Permit 钓鱼模拟

这一个内容就到了我要学习的地方，之前没听过。开始学！

因为一开始还不知道所以还是点了授权，被骗+1

> 知识点：
>
> **通过使用 EIP-2612 的 `permit` 签名功能**，用户可以在不预先进行链上授权交易（即无需支付 Gas 费）的情况下，授权第三方合约使用自己的代币（如 USDC、DAI 等）。这是以太坊上一种更高效、更省 Gas 的授权方式，尤其适合优化用户体验（UX）和批量操作。
>
> **1. EIP-2612 `permit` 的核心机制**
>
> **（1）传统授权（`approve`）的问题**
>
> - 需要发送一笔链上交易（支付 Gas）。
> - 用户必须提前授权，导致交互流程变长。
>
>  **（2）`permit` 的改进**
>
> ✅ **免 Gas 授权**：用户签署一条链下消息（签名），第三方合约可凭此签名直接获得代币使用权，无需用户预先发送 `approve` 交易。
> ✅ **单次有效**：签名可设置过期时间（`deadline`），避免长期风险。
> ✅ **兼容 ERC-20**：无需修改代币标准，只需代币合约实现 `permit` 函数。
>
>  **2. `permit` 的工作原理**
>
>  **（1）用户签署离线消息**
>
> 用户对以下数据进行签名（使用 EIP-712 结构化签名）：
>
> ```solidity
> {
>   owner: "0x用户地址",       // 代币持有者
>   spender: "0x合约地址",    // 被授权方（如 Uniswap）
>   value: 1000000,          // 授权数量（如 1 USDC = 1e6）
>   nonce: 123,              // 防止重放攻击
>   deadline: 1698765432     // 过期时间（UNIX 时间戳）
> }
> ```
>
>  **（2）第三方合约提交签名**
>
> 合约调用代币的 `permit` 方法，传入签名数据：
>
> ```solidity
> token.permit(owner, spender, value, deadline, v, r, s);
> ```
>
> - `v, r, s` 是签名的 ECDSA 参数。
> - 代币合约验证签名，并更新授权状态（相当于执行了 `approve`）。
>
>  **3. 安全注意事项**
>
>  **✅ 优点**
>
> - **节省 Gas**：用户只需签一次名，无需单独发 `approve` 交易。
> - **更短交互流程**：适合钱包内直接签名授权。
>
>  **⚠️ 风险**
>
> - **签名钓鱼**：恶意 DApp 可能诱导用户签署高额 `permit`（检查 `value` 和 `spender`）。
> - **过期时间失效**：若 `deadline` 过长，签名可能被重复使用（建议设置较短有效期）。

我的理解就是 授权第三方合约使用自己的代币，然后可以省去gas费用

这个签名将允许攻击者控制您的 USDC 代币！通过使用 EIP-2612 permit 签名，攻击者可以：

- 获得对您所有 USDC 的完全访问权限
- 在未来任何时间转移您的代币
- 无需您进一步批准即可花费您的资金

安全检查要点

- 检查 Permit 类型，了解授权范围

- 验证 Spender 地址是否为可信来源

- 注意授权金额，警惕无限制授权

- 确认网站来源的可信度

所以此处不能进行授权



### No.0x0004 专属代币空投

此交易实际上是向合约 0xbe535a82f2c3895bdaceb3ffe6b9b80ac2f832a0 发送 0.5 ETH，而不是领取任何代币。

函数选择器 0x5fba79f5 调用了一个名为 SecurityUpdate() 的函数，该函数可能会将您的资金转移给攻击者。

**在真实情况下，永远不要在不了解交易内容的情况下签署交易！**



### No.0x0005 USDT 授权钓鱼模拟

挑战：请小心真实的授权请求

 **授权风险提示：**

- 除了 approve 外，也要当心 increaseAllowance 函数
- increaseAllowance 同样可以增加代币授权额度
- 一些钓鱼网站会通过这个方式来掩饰其真实意图

💡 安全建议：永远不要给不明来源的网站无限授权！

> `increaseAllowance` 是 **ERC-20 代币标准** 中的一个扩展函数，用于**安全地增加**某个地址（`spender`）的代币授权额度。它是对传统 `approve` 方法的改进，旨在避免潜在的安全风险（如前端竞态条件攻击）。
>
> **传统 `approve` 的问题**
>
> - **竞态条件（Race Condition）**
> 	如果用户连续发起两笔 `approve` 交易（例如先授权 100，再改为 200），矿工可能以相反顺序打包交易，导致最终授权额度被意外覆盖（变成 100 而非 200）。
>
> 	solidity
>
> 	```
> 	// 危险操作：可能被覆盖
> 	approve(spender, 100);  // 交易1
> 	approve(spender, 200);  // 交易2
> 	```
>
>  **`increaseAllowance` 的解决方案**
>
> ✅ **增量调整**：基于当前授权额度增加数值，而非直接覆盖。
> ✅ **安全操作**：避免竞态条件，适合前端交互。



### No.0x0006 假冒代币空投钓鱼攻击

挑战：识别真假域名

- 场景描述

某天，你收到一封电子邮件，声称你有资格获得UNI代币的空投！邮件中包含一个链接，引导你到一个看似合法的网站。你点击了该链接，并看到以下交易记录：

<img src="https://cdn.jsdelivr.net/gh/xmhhmx/PicGoCDN//img/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-08-05%20144514.png" style="zoom:67%;" />

仔细观察上面的交易记录。这种钓鱼攻击通常利用视觉上相似的字符（如用数字"1"替代字母"i"）来欺骗用户。

**安全建议**

- 始终逐字符检查域名
- 警惕使用数字代替字母的域名（例如，用'1'代替'i'）
- 收藏官方网站而不是点击电子邮件中的链接
- 使用密码管理器，它只会在合法域名上自动填充
- 安装警告钓鱼网站的浏览器扩展



### No.0x0007 超高收益质押平台

挑战：了解如何识别可疑的质押合约

某些可疑的质押合约可能会引导您进行 **approve** 授权，表面上看似只为了质押特定金额，实际上却请求对整个代币余额的完全访问权限。一旦授权成功，攻击者即可任意转走您的所有资产。

<img src="https://cdn.jsdelivr.net/gh/xmhhmx/PicGoCDN//img/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-08-05%20145026.png" style="zoom:67%;" />

如上图，存在approve方法需要谨慎！！

> `approve` 是 **ERC-20 代币标准** 中的一个核心方法，用于授权另一个地址（通常是智能合约）代表你支配一定数量的代币。它是 DeFi（去中心化金融）交互的基础，但错误使用可能导致资金风险。



### No.0x0008 Telegram 代币钓鱼挑战

挑战：学习识别和避免 Telegram 上的助记词钓鱼攻击

风险点依旧是提供助记词，所以千万不要给别人提供自己钱包的助记词！！！

<img src="https://cdn.jsdelivr.net/gh/xmhhmx/PicGoCDN//img/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-08-05%20145933.png" style="zoom:67%;" />



### No.0x0009 Punycode 钓鱼攻击

挑战：识别 Punycode 钓鱼域名

- 场景描述

您收到一封电子邮件，声称是来自 Trezor（一个知名的硬件钱包品牌）的重要安全更新通知。电子邮件中的链接看起来像是指向官方 Trezor 网站，但实际上是一个精心伪装的钓鱼网站。

```
trẹzor.com
看起来像正常的 Trezor 域名
```

> #### 什么是 Punycode？
>
> Punycode 是一种编码系统，允许将非 ASCII 字符（如西里尔字母、中文等）转换为 ASCII 字符，以便在域名系统中使用。攻击者经常利用视觉上相似的字符创建看似合法的域名。 例如，某些特殊字符看起来与拉丁字母几乎相同，但它们是不同的字符： 你可以使用 [Punycoder](https://www.punycoder.com/) 来转换 Unicode 和 Punycode 域名。
>
> | 显示域名   | 实际 Punycode 域名 | 说明                       |
> | :--------- | :----------------- | :------------------------- |
> | trẹzor.com | xn--trzor-o51b.com | 使用特殊字符替换了某些字母 |

所以 trezor的官方域名应该为trezor.io

防御方式：

- 直接在浏览器中输入已知的官方网址，而不是点击电子邮件中的链接
- 使用书签保存常用的重要网站
- 安装可以检测 Punycode 域名的浏览器扩展
- 注意域名中不寻常的字符或拼写



### No.0x0010 剪贴板钓鱼挑战

挑战：识别剪贴板型钓鱼攻击

- 场景描述

您需要转账 1 ETH 到朋友的钱包。他们已经分享了他们的钱包地址，您正在使用加密货币转账界面进行转账。

<img src="https://cdn.jsdelivr.net/gh/xmhhmx/PicGoCDN//img/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-08-05%20150754.png" style="zoom:50%;" />



复制粘贴之后，发现钱包地址与之前的不同了

- 攻击原理：

1. 攻击者创建一个看似合法的网站
1. 当你点击"复制"按钮时，恶意JavaScript代码会秘密替换复制的地址
1. 如果粘贴后没有验证地址，你可能会将资金发送给攻击者

- 防御方式：

	- 粘贴后务必再次检查地址

	- 考虑使用带有地址验证的硬件钱包

	- 在电脑上复制敏感信息（如钱包地址）时要特别小心 - 即使使用Ctrl+C，恶意软件也可能篡改你的剪贴板内容



### No.0x0011 Google 搜索广告钓鱼攻击

挑战：识别 Google 搜索广告钓鱼

- 场景描述

您想要使用 Lido Finance 质押 ETH。在 Google 上搜索"Lido Finance"时看到这些结果。您能识别出哪个是合法网站，哪个是钓鱼网站吗？

<img src="https://cdn.jsdelivr.net/gh/xmhhmx/PicGoCDN//img/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-08-05%20151348.png" style="zoom:67%;" />

看第一条的左上角有一个Ad，是广告的意思，怀疑是钓鱼网站

辨别网站真伪的方法：

- 攻击者经常购买与热门加密货币项目相关的 Google 广告，这些广告会出现在搜索结果的顶部，标记为"赞助"或"广告"。
- 这些广告通常使用与官方网站非常相似的域名，但有细微的差别，例如：
	-  使用不同的顶级域名（如用 .is 代替 .fi）
	-  在域名中添加或删除字母
	-  使用连字符或用数字替换字母
	-  当用户点击这些广告时，他们会被引导到看起来与官方网站完全相同的钓鱼网站，这些网站旨在窃取资金或私钥。



### No.0x0012 Microsoft Teams 钓鱼攻击

本页面模拟攻击者如何创建虚假的 Microsoft Teams 网站来分发恶意软件和窃取敏感信息。

<img src="https://cdn.jsdelivr.net/gh/xmhhmx/PicGoCDN//img/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-08-05%20151843.png" style="zoom:67%;" />

本示例中的钓鱼指标

- **可疑 URL**：注意域名是 "microsoft-meet.com" 而不是 "teams.microsoft.com"
- **简化界面**：与真实的 Teams 登录相比，虚假页面具有简化的界面
- **缺乏安全功能**：缺少 Microsoft 通常包含的安全元素
- **加入按钮**：突出的 "在 Teams 应用程序中加入" 按钮可能会导致恶意软件下载

安全提示

- 在输入凭据或下载软件之前，始终验证 URL。
- 仅从官方 Microsoft 网站或应用商店下载 Microsoft Teams。
- 对任何异常的安装过程或请求保持警惕。
- 如果您下载了可疑的软件包，可以在打开之前使用 [VirusTotal.com](https://www.virustotal.com/) 进行扫描。但请注意，即使没有检测到威胁（0 检测），也不能保证绝对安全。



> ## Microsoft Teams 钓鱼攻击的工作原理
>
> 攻击者创建令人信服的 Microsoft Teams 登录页面或更新通知的复制品，诱骗用户下载恶意软件或泄露其凭据。这些攻击变得越来越复杂，针对个人和组织。
>
> 风险1：数据泄露
>
> 通过虚假 Teams 更新安装的恶意软件可以访问您设备上的敏感文件，可能导致未经授权访问个人和公司数据。这可能导致知识产权盗窃、机密信息泄露和合规违规。
>
> 风险2：凭据盗窃
>
> 当用户在虚假 Teams 网站上输入其 Microsoft 凭据时，攻击者会捕获这些信息以获取对电子邮件、OneDrive、SharePoint 和其他 Microsoft 365 服务的访问权限。这可能导致账户被接管并进一步危及组织资源。
>
> 风险3：钱包资金耗尽
>
> 对于从同一设备访问加密货币钱包或金融服务的用户，通过虚假 Teams 更新安装的恶意软件可能包含扫描钱包凭据的功能，导致数字资产被盗。



### No.0x0037 虚假扩展程序钓鱼

挑战：识别出虚假扩展程序

<img src="https://cdn.jsdelivr.net/gh/xmhhmx/PicGoCDN//img/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-08-05%20155308.png" style="zoom: 50%;" />

扩展程序安全最佳实践：

- 始终从官方网站下载：直接访问官方网站（例如 metamask.io），而不是在扩展商店中搜索。虚假或恶意扩展程序可能出现在搜索结果中，甚至出现在官方扩展商店中，有时会模仿真实扩展程序的名称、图标或品牌。通过访问官方网站，您可以确保获得正宗、安全的版本，避免落入可能危及您安全和资产的仿冒或诈骗列表的陷阱。
- 检查用户数量和评论：官方扩展程序拥有大量用户和普遍积极的评论
- 仔细阅读权限：只授予必要的权限
- 保持扩展程序更新：定期更新通常包含安全补丁
- 删除未使用的扩展程序：通过删除不再使用的扩展程序来减少攻击面

# 2025-08-04

# Web3共学

## 区块链基础概念

### 区块链介绍

区块链是一种**去中心化**的**分布式账本**技术，用于在网络节点之间安全、透明且不可篡改地记录事务数据

- 去中心化：区块链网络通常分布在全球，每个节点都将会存储一份相同的区块链数据。没有人能够控制全部的节点，因此这份区块链数据将会一直存在。



### 区块链特性

1. 不可篡改

你无法改变历史信息，因为每个区块包含了上一个区块的摘要并串联起来，如果你修改了历史的区块，你将必须修改后面的全部区块。所以交易一旦上链，就无法再更改

2. 公开透明、匿名

在区块链上的信息全部公开透明。每个人都可以顺着区块和链找到历史上所有的记录来查看你的钱包余额。可以在区块链浏览器上进行查看

3. 快速交易

无论金额多少以及你在什么地方，只要你的交易记录被打包在区块链中，交易就自动完成。相比传统的跨国汇款非常快速便捷



### 区块链的核心组成部分

#### **去中心化的网络和区块链**

区块链将会有一条链来记录全部的信息，这条链将存在对应的去中心化网络中。 去中心化的网络，将由无数节点提供服务来维持网络运行。节点通过计算验证交易获得代币奖励



#### **维持网络运行的代币激励**

去中心化的网络由无数节点提供服务来维持网络运行，整合区块并合并到链上的操作称为**挖矿**。维持这些服务的人一般称之为**矿工**。矿工们维持网络运行会得到代币奖励以及燃料费（Gas Fee）。 你使用这个网络进行交易、转账、铸造 NFT 等等，均需要支付代币



### 公链 私链 联盟链

区块链根据访问权限与治理模式，大致可分为三类。按照去中心化程度从高到低排列

![](https://cdn.jsdelivr.net/gh/xmhhmx/PicGoCDN//img/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-08-04%20205327.png)

####  公链（Public Blockchain）

- **特点**：
	- **完全开放**：任何人可参与读写、验证交易或成为节点。
	- **去中心化**：无单一控制方，数据公开透明，不可篡改。
	- **激励机制**：通常通过代币（如比特币、以太坊）奖励矿工/验证者。
	- **性能较低**：因共识机制（如PoW、PoS）需全局节点验证，交易速度较慢。
- **典型应用**：
	- 加密货币（比特币、以太坊）。
	- 去中心化应用（DApp）、DeFi、NFT等开放生态。
- **代表案例**：
	Bitcoin、Ethereum、Solana。



#### 私链（Private Blockchain）

- **特点**：
	- **权限封闭**：由单一组织或实体控制，参与者需授权。
	- **中心化**：节点由管理者指定，交易验证效率高。
	- **隐私性强**：数据仅对授权方可见，适合企业内部使用。
	- **无代币激励**：通常无需挖矿，节点由组织自行维护。
- **典型应用**：
	- 企业数据管理、内部审计、供应链追踪等。
	- 对隐私和效率要求高的封闭场景。
- **代表案例**：
	Hyperledger Fabric（可配置为私链）、R3 Corda。



#### 联盟链（Consortium Blockchain）

- **特点**：
	- **部分去中心化**：由多个组织联合管理（如银行、企业联盟）。
	- **准入机制**：节点需许可加入，但参与者间平等协作。
	- **平衡效率与信任**：共识机制（如PBFT）比公链更快，兼顾一定透明度。
	- **部分公开**：数据可对成员共享，对外保密。
- **典型应用**：
	- 跨机构业务（跨境支付、贸易金融）。
	- 行业协作（物流、医疗数据共享）。
- **代表案例**：
	Hyperledger Fabric、FISCO BCOS、Quorum。

#### 对比总结

| **维度**     | **公链**            | **联盟链**         | **私链**         |
| :----------- | :------------------ | :----------------- | :--------------- |
| **控制权**   | 无中心主体          | 多组织共同治理     | 单一组织控制     |
| **参与权限** | 完全开放            | 需许可加入         | 严格授权         |
| **透明度**   | 全网公开            | 成员间透明         | 仅内部可见       |
| **性能**     | 低（TPS低，延迟高） | 中（优化共识机制） | 高（中心化处理） |
| **用例**     | 加密货币、开放生态  | 跨机构协作         | 企业内部管理     |

#### 选择依据

- **公链**：适合需要完全去中心化和信任透明的场景。
- **联盟链**：适合多组织协作且需平衡效率与隐私的行业。
- **私链**：适合单一组织追求高效可控的私有化应用。


# 2025.07.30


<!-- Content_END -->
