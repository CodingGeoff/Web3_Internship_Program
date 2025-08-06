---
timezone: UTC+8
---

# lynn

**GitHub ID:** Free-EntropyMXZ

**Telegram:** @lyndonma

## Self-introduction

web3初学者，做过一些学习项目，涉及defi，zkp，web3+ai，希望通过公学有更深入的钻研

## Notes

<!-- Content_START -->
# 2025-08-06

# 今日学习笔记：Web3智能合约测试与模糊测试

**日期**：2025年8月6日  
**主题**：基于transcript的Web3安全审计课程，聚焦智能合约测试，特别是集成测试和模糊测试（fuzz testing），以Pool Sharks协议为例。

---

## 1. 核心概念：智能合约测试的重要性
- **为什么重要**：Web3中，bug可能导致数亿资产损失（如误发交易到零地址）。测试比Web2更关键，是防止漏洞的第一道防线。
- **测试类型**：
  - **单元测试**：测试单一函数（如算术逻辑），早期捕获bug，易调试。
  - **集成测试**：验证用户交互流程和跨合约逻辑，捕获复杂bug（如状态不一致）。
  - **模糊测试**：用随机输入覆盖边缘场景，高效发现关键漏洞。
- **代码覆盖率局限性**：
  - 100%覆盖率 ≠ 无漏洞。示例：票务合约覆盖100%，但因缺少`onlyOwner`修饰符，任何人可提取资金。
  - 需测试路径依赖场景（path-dependent），如用户操作顺序、价格波动。

---

## 2. Pool Sharks协议简介
- **功能**：一个在Fuel Network上的自动化做市商（AMM），支持单向流动性（Directional Liquidity）和价格-时间优先级队列，优化交易效率，减少无常损失。
  - **单向流动性**：仅在价格上升时执行交易（如买入ETH），下降时订单保留（不取消）。
  - **场景**：代币交换、期权交易、网格交易。
- **测试重点**：
  - 验证“全局流动性不溢出”（`totalLiquidity >= 0`）。
  - 测试路径依赖场景（如价格上升执行、下降不取消）。

---

## 3. 集成测试与路径依赖
- **定义**：集成测试模拟用户交互和跨合约逻辑，验证协议整体行为。
- **路径依赖**：
  - **路径无关**：如ERC20转账，顺序不影响结果。
  - **路径依赖**：如AMM中，Alice先买推高价格，Bob后买得更少代币；反序结果不同。
  - Pool Sharks例子：价格上升时执行买入，下降时保留订单，需测试不同顺序（如Alice先提供流动性 vs Bob先交易）。
- **设计方法**：
  - 模拟多用户（用`vm.prank`切换用户）。
  - 测试不同顺序和边缘场景（如零输入、最大杠杆）。
  - 用`assert`验证状态（如`totalLiquidity`）。
- **Solidity示例**：
  ```solidity
  function testPathDependentLiquidity() public {
      vm.prank(alice);
      pool.provideLiquidity(alice, 100, 1900);
      assertEq(pool.totalLiquidity(), 100);

      vm.prank(bob);
      pool.executeTrade(bob, 50, 2500); // 价格上升
      assertEq(pool.totalLiquidity(), 50);
  }
  ```

---

## 4. 模糊测试与Forge Test
- **模糊测试**：用随机输入（如`uint256 amount`）测试边缘场景，高效发现漏洞（如溢出）。
- **Pool Sharks测试优秀之处**：
  - **路径依赖测试**：覆盖价格上升/下降场景，验证单向流动性逻辑。
  - **不变性验证**：用Echidna或Forge检查`totalLiquidity >= 0`，300次随机输入（`fuzz.runs = 300`）。
  - **自动化反例**：失败时自动保存调用序列，方便修复。
- **Forge Test行为**：
  - 命令：`forge test --match-contract CalcTest -Vv`
    - `--match-contract CalcTest`：只运行`CalcTest`合约。
    - `-Vv`：详细日志，显示模糊测试输入和反例。
    - `fuzz.runs = 300`（在`foundry.toml`）自动触发模糊测试，运行300次随机输入。
  - 函数需接受随机参数（如`test_Fuzz_Liquidity(uint256 amount)`）。
  - 用`vm.assume`限制输入范围（如`amount <= type(uint256).max`）。
- **Solidity示例**：
  ```solidity
  function test_Fuzz_Liquidity(uint256 amount, uint256 minPrice) public {
      vm.assume(amount > 0 && amount <= type(uint256).max - pool.totalLiquidity());
      vm.assume(minPrice <= pool.currentPrice());
      pool.provideLiquidity(address(this), amount, minPrice);
      assertGe(pool.totalLiquidity(), amount);
  }
  ```

---

## 5. 审计视角
- **检查点**：
  - 测试套件是否覆盖路径依赖场景（如多用户顺序、价格波动）。
  - 模糊测试是否用`vm.assume`限制输入，验证不变性（如流动性不溢出）。
  - 日志（`-Vv`）是否输出清晰反例，方便调试。
- **建议**：
  - 运行`forge coverage`检查覆盖率，关注未测分支。
  - 验证`foundry.toml`的`fuzz.runs`（如300次）是否足够。
  - 模拟攻击场景（如sandwich攻击）测试协议鲁棒性。

---

## 6. 总结
今日学习了Web3智能合约测试的核心：单元测试打基础，集成测试验证用户流程，模糊测试覆盖边缘场景。Pool Sharks的测试通过路径依赖（价格上升执行、下降不取消）和不变性（流动性不溢出）高效发现漏洞。`forge test`结合`fuzz.runs = 300`自动运行模糊测试，`-Vv`提供详细调试信息。理解这些对Web3安全审计至关重要。

**后续计划**：
- 练习用Foundry写模糊测试，模拟Pool Sharks场景。
- 学习Echidna配置，结合Forge运行不变性测试。
- 审计时重点检查路径依赖和反例生成。

# 2025-08-05

# Resupply Protocol 攻击事件深度分析

## 1. 协议简介

**Resupply Protocol (ResupplyFi)** 是一个去中心化稳定币协议，旨在为用户提供稳定币资产的再融资和收益最大化策略。其核心机制基于**抵押债仓（CDP）**，允许用户通过抵押稳定币资产，如 `crvUSD` 和 `frxUSD`，来铸造其原生稳定币 **reUSD**。

Resupply 协议的运作依赖于外部 DeFi 生态，特别是 **Curve LlamaLend**。其借贷市场（`Market`）的抵押品不是直接的稳定币，而是 **ERC-4626 Vault** 发行的份额代币，例如 `cvcrvUSD`。

## 2. 攻击核心漏洞

本次攻击的核心在于 Resupply 协议的**价格预言机（Price Oracle）**和**健康检查机制**存在严重缺陷。攻击者利用了一个多环节的逻辑漏洞链，包括：

1.  **ERC-4626 Vault 价格操纵**：利用新创建的、流动性极低的 Vault，通过“捐赠攻击”人为地夸大其每股价格。
2.  **整数除法向下舍入**：Resupply Market 用来计算借贷健康系数的 `_exchangeRate` 公式存在缺陷，即 `_exchangeRate = 10^36 / price`。当被操纵的 `price` 大于 $10^{36}$ 时，Solidity 的整数除法会使 `_exchangeRate` 结果为 `0`。
3.  **健康检查失效**：`_exchangeRate = 0` 使得协议的健康检查 (`_isSolvent` ) 逻辑被绕过，允许借款人借出无抵押的资金。

## 3. 攻击复盘：五个核心步骤

攻击者通过一笔闪电贷（Flash Loan）完成了所有关键步骤，确保了攻击的原子性和成功：

1.  **准备资金**：通过闪电贷借入 4,000 USDC 并兑换为 3,999 `crvUSD`。

2.  **执行捐赠攻击**：将 **2,000 `crvUSD`** 捐赠给 `Controller 0x8970`（`Vault 0x0114` 的底层资产存储合约）。这使得 `Vault` 的 **`total_assets`** 激增，但 **`totalSupply`** 仍然极小，造成了资产与份额的严重不平衡。

3.  **铸造被操纵的抵押品**：将剩余的 **2 `crvUSD`** 存入 `Vault 0x0114`。由于其每股价格被操纵得极高，攻击者只获得了极少量的 `cvcrvUSD` 份额代币（即 1 unit）。

4.  **提供抵押品**：将这 `1 unit` 的 `cvcrvUSD` 抵押到 `Market 0x6e90`。此时，`Market` 错误地将这极少量的抵押品估值为天价。

5.  **借出巨额资金**：从 `Market 0x6e90` 借出 **10,000,000 reUSD**。在借款前的健康检查中，被操纵的 `cvcrvUSD` 价格导致 `_exchangeRate` 被计算为 `0`，从而绕过了安全检查。

## 4. 攻击结果与影响

* **巨额损失**：Resupply Protocol 遭受了约 **960 万美元**的资金损失。
* **资金洗白**：被盗资金通过去中心化交易所兑换后，利用 **Tornado Cash** 等隐私协议进行了混淆。
* **行业警示**：该事件再次凸显了 DeFi 协议在**价格预言机、整数计算、新市场部署和合约审计**方面存在的潜在风险。特别是对于依赖复杂数学逻辑和外部合约交互的协议，必须进行更严格的审查和测试。

## 5. 技术总结

本次攻击的核心在于一个新部署的 ERC-4626 Vault 在其初始化阶段容易受到价格操纵。攻击者通过捐赠行为使得 `total_assets` 远大于 `totalSupply`，从而将每股价格抬高到超出合约处理能力的范围。最终，一个简单的整数除法错误成为了突破口，导致协议的借贷安全机制完全失效，造成了重大损失。

# 2025-08-04

# Web3 安全审计笔记：智能合约设计最佳实践

以下是基于课程transcript整理的Web3安全审计核心要点，聚焦智能合约设计原则和外部调用风险，旨在帮助开发者构建安全合约并指导审计实践。内容涵盖减少代码量、避免循环、限制输入、处理所有情况、避免平行数据结构，以及外部调用中的重入、DOS、返回值和gas风险。

1\. 智能合约设计核心原则

### 1.1 减少代码量（Less Code is Better）

- **原则**：代码越少，bug越少。研究表明，每1000行代码约17个bug。智能合约中，bug可能是严重漏洞（如资金窃取）。
  - 例：GMX V2（10,000+行）仍有50+ critical和30+ high漏洞。
  - 关系：bug数量与代码行数呈抛物线增长。
- **实现**：
  - **精简存储变量**：避免冗余存储。
    - 例：永久合约中，`orderToPositionId`映射重复订单结构体中的位置ID，导致不同步风险。解决：嵌入ID，省50-60行代码，降低审计成本（按行收费）。
  - **链外逻辑**：不必要计算（如流动性百分比转换）移到前端。
    - 例：集中流动性协议中，链上转换增加gas和bug风险（&gt;100%输入）。移除省50-60行。
- **审计建议**：检查存储变量冗余，搜索`mapping`和`struct`，确保单一数据源。

### 1.2 避免循环（Avoid Loops）

- **原则**：循环易导致DOS攻击或gas超限（EVM块gas上限）。
  - 例：不要循环计算总利息，用聚合变量（如总开仓利息）代替。
- **例外**：必要循环（如交换路径）需限长度，确保gas恒定。
- **审计建议**：搜索`for`关键字，检查是否可用累加器优化。

### 1.3 限制预期输入（Limit Expected Inputs）

- **原则**：禁止无意义/恶意输入，用`require`早切断。
  - 例：GMX允许零大小/零抵押位置，导致rounding操纵。解决：加最小大小阈值。
- **权衡**：多`require`安全但gas贵，可用`modifier`或custom errors优化。
- **审计建议**：检查`require`覆盖协议特定输入（如无重复市场）。

### 1.4 处理所有情况（Handle All Cases）

- **原则**：别假设理想情况（如USDC=1美元，keeper及时清算）。处理极端场景（如破产清算：损失&gt;抵押）。
  - 例：抵押$1000，损失$1100，不处理导致underflow。解决：限损失为抵押额。
- **审计建议**：模拟价格闪崩、keeper失败场景。

### 1.5 避免平行数据结构（Avoid Parallel Data Structures）

- **原则**：别用多结构（如mapping+array）跟踪同一数据，易不同步。
  - 例：Minimal Finance中，声明者映射+数组未更新索引，锁定资金。
- **替代**：用OpenZeppelin的`EnumerableMap`支持迭代。
- **审计建议**：检查多结构跟踪同一数据，搜索`mapping`和`array`。

## 2. 外部调用风险与防护

### 2.1 重入攻击（Reentrancy）

- **风险**：外部调用可能被攻击者重入，窃取资金。
  - 例：跨合约重入（系统多合约）或只读重入（操纵Curve池价格）。
- **防护**：
  - 用“检查-效果-交互”（CEI）模式：先检查、更新状态、最后交互。
  - 用OpenZeppelin的`ReentrancyGuard`（`nonReentrant` modifier）。
  - GMX V2用全局nonReentrant，跨合约保护。
- **审计建议**：检查所有`call`，确保CEI顺序和`nonReentrant`。

### 2.2 DOS攻击（Denial of Service）

- **风险**：外部调用失败（如恶意回调、不接受ETH的合约）导致关键功能瘫痪。
- **防护**：用`try-catch`捕获失败，只发事件，不revert主逻辑。
- **审计建议**：模拟调用失败，检查主逻辑是否继续。

### 2.3 返回值处理（Return Values）

- **风险**：不可信合约返回任意字节，可能导致解析失败或逻辑操纵。
  - 例：GMX V2解析错误数据（`bytes memory errorData`）可能revert。
- **防护**：用低级调用（`assembly`）设返回值大小为0，避免解析。
- **审计建议**：检查`bytes memory`，确保不解析不可信数据。

### 2.4 Gas消耗（Gas Considerations）

- **风险**：
  - 返回值加载到内存：EVM内存分配成本随数据大小二次增长，可能OOM。
    - 例：攻击者返回1MB数据，耗尽gas，revert交易。
  - 转发所有gas（`gasleft()`）：不可信合约可耗尽gas，DOS或增加keeper成本。
  - 例：GMX V2回调若不限gas，可能延迟市场订单。
- **防护**：
  - **指定gas限额**：用`call{gas: X}`，X为预期量。
  - **低级调用**：用`assembly`设返回值大小为0。
    - 例：

      ```solidity
      function safeCall(address recipient, uint256 amount) external {
          bool success;
          assembly {
              success := call(100000, recipient, amount, 0, 0, 0, 0)
          }
          if (!success) emit CallFailed(recipient);
      }
      ```
  - **Try-Catch**：捕获失败，保护主逻辑。
  - **区分上下文**：用户调用自担gas，keeper调用需严格限。
- **审计建议**：
  - 搜索`call`/`gasleft()`，检查gas限额。
  - 用Foundry模拟高gas场景（如1MB返回数据）。
  - 验证keeper路径的gas经济性。

### 2.5 邮寄检查（Post-Checks）

- **原则**：函数结束时验证不变性（如余额&gt;=预期），防止意外资金流出。
  - 例：GMX V2检查市场代币余额，但未更新抵押额导致正常行为revert。
- **防护**：确保检查覆盖所有场景，不引入DOS。
- **审计建议**：检查`assert`/`require`，模拟未更新状态场景。

## 3. 文档建议

- **优秀README**：包含协议摘要、组件描述、交互流程图、执行示例、预言机细节。
  - 例：Blueberry Bank的README有用户执行流程图，助审计和团队协作。
- **建议**：用PlantUML画流程图，审计时先读README。

## 4. GMX V2案例分析

- **背景**：GMX V2允许用户指定任意回调合约（`callbackContract`），用于第三方集成（如Dolomite）。
- **安全实现**：
  - **顺序**：回调在订单逻辑后（CEI）。
  - **重入防护**：全局nonReentrant，跨合约保护（订单/存款/提款处理器共享数据存储）。
  - **DOS**：try-catch捕获失败，只发事件。
  - **返回值**：不解析不可信数据，用低级调用设大小为0。
  - **Gas**：限gas转发，保护keeper。
- **审计**：花6个月验证回调后逻辑不可游戏化（non-gameable）。
- **学习建议**：阅读GMX V2的`OrderUtils.sol`，分析回调逻辑。

## 5. 审计实践清单

- **减少代码量**：检查存储变量冗余，搜索`mapping`/`struct`。
- **循环**：搜索`for`，验证是否可用累加器。
- **输入**：检查`require`覆盖协议特定输入。
- **所有情况**：模拟价格闪崩、keeper失败。
- **平行结构**：检查多结构跟踪同一数据。
- **外部调用**：
  - 搜索`call`/`delegatecall`，确保CEI和`nonReentrant`。
  - 检查`bytes memory`，避免解析不可信数据。
  - 验证gas限额，模拟高gas场景。
  - 检查`try-catch`，确保失败不影响主逻辑。
- **邮寄检查**：验证`assert`/`require`覆盖所有场景。
- **工具**：用Foundry/Hardhat测试，阅读EVM黄皮书了解`CALL` opcode。

## 6. 学习资源与建议

- **实践**：审计GMX V2或Uniswap V3代码，重点检查回调和gas。
- **工具**：Foundry模拟攻击场景，PlantUML画流程图。
- **深入**：学习EVM内存模型和gas成本（黄皮书）。
- **代码示例**（测试gas攻击）：

  ```solidity
  contract MaliciousCallback {
      function malicious() external {
          uint256[] memory data = new uint256[](1000000);
          for (uint256 i = 0; i < data.length; i++) {
              data[i] = i;
          }
      }
  }
  contract Test {
      event CallFailed(address);
      function testCallback(address callback) external {
          (bool success, ) = callback.call{gas: 100_000}(abi.encodeWithSignature("malicious()"));
          if (!success) emit CallFailed(callback);
      }
  }
  ```


# 2025.07.29


<!-- Content_END -->
