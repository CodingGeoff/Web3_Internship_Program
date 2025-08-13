---
timezone: UTC+8
---

# 若言

**GitHub ID:** cmu-ruoyan-lgl

**Telegram:** @ruoyan_ton

## Self-introduction

web2游戏大厂工作两年

## Notes

<!-- Content_START -->
# 2025-08-13

# DEX

## AMM（Automated Market Maker）

- Market Maker**（控制币对的交易价格比较平滑，避免出现价格从2美元突然降到1美元）**
- Liquidity
- Liquidity Provider

## 去中心化交易所核心要素

- 任何人都可以添加流动性，成为 LP，并拿到 LP Token
- LP 在任意时间可以移除流动性并销毁 LP Token，拿回自己的 Token
- 用户可以基于交易池来进行交易
- 交易时收取一定手续费，并且分配给 LP

## Constant Product Automated Market Maker（CPAMM，恒定乘积自动做市商）

- 添加流动性：注入流动性/初始价格确定
- 交易：$x*y=(x+\Delta x)*(y-\Delta y)=k$，滑点
- 移除流动性：无偿损失等

**这里我们要注意，swap 的时候会出现滑点，而在添加/移除流动性时则会出现无常损失，要区分好**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/49dfc960-b5ce-4462-be6c-aa0cb64627ca/Untitled.png)

## 公式推演

$x*y=k$，$(x+\Delta x)*(y+\Delta y)=k$，知$\Delta x$，求$\Delta y$

1. 交换

$$
\begin{aligned}
x*y &=(x + \Delta x) * (y - \Delta y) \\
&= x*y + \Delta x * y - x *  \Delta y - \Delta x * \Delta y
\end{aligned}
$$

由上面可得，

$$
x*\Delta y + \Delta x*\Delta y = \Delta x * y 
\Rightarrow 
\Delta y = \frac{\Delta x * y}{x + \Delta x}
$$

1. 添加流动性

$$
\frac{x}{y} = \frac{x+\Delta x}{y+\Delta y}
$$

由上面可得，

$$
x*y + x*\Delta y = x*y + \Delta x*y 
\Rightarrow 
\Delta y = \frac{\Delta x*y}{x}
\Rightarrow
\frac{\Delta x}{\Delta y} = \frac{x}{y}
$$

- 为什么在 $x*y=k$中，$k$ 实际取 $k = \sqrt{x * y}$，而不是取 $x * y$?

这里是因为想要让 k 与流动性保持一种线性的关系，而不是像 $y = x ^ 2$ 在后面随着 x 的增加，y 的数值会急剧增加

$L_0$：添加之前的 Liquidity，设为 T

$L_1$：添加之后的 Liquidity，设为 T+S，其中 S 为添加的流动性

$$
\frac{L_0}{L_1} = \frac{T}{T+S}
$$

由上面可得，

$$
\begin{aligned}
S &= \frac{(L_1-L_0)*T}{L_0} \\
  &= \frac{\sqrt{(x + \Delta x)*(y + \Delta y)} - \sqrt{x*y}}{\sqrt{x*y}} * T \\
  &= \frac{\sqrt{(x + \Delta x)*(y + \frac{\Delta x * y}{x})} - \sqrt{x*y}}{\sqrt{x*y}} * \frac{\sqrt{x}}{\sqrt{x}} * T \\
  &= \frac{\sqrt{x^2 * y + \Delta x *x*y + \Delta x *x*y + \Delta x^2 *y} - x * \sqrt{y}}{x * \sqrt{y}} (消掉\sqrt{y})\\
  &= \frac{\sqrt{x^2 + 2*\Delta x*x + \Delta x^2} - x}{x} \\
  &= \frac{(x + \Delta x) - x}{x} \\
  &= \frac{\Delta x}{x} * T = \frac{\Delta y}{y} * T(同理)
\end{aligned}
$$

1. 移除流动性

S ：share的数量  T：移除流动性之前 Liquidity 的总量  L：Liquidity 的数量

$$
\begin{aligned}
&\frac{\sqrt{\Delta x * \Delta y}}{\sqrt{x*y}}=\frac{S}{T} \\
&\sqrt{\Delta x * \Delta y} = \sqrt{x*y} * \frac{S}{T} \\
&\Delta x * \sqrt{\frac{y}{x}} = \sqrt{x*y} * \frac{S}{T}(消掉\sqrt{y}，并把\sqrt{x}化简) \\
&\Delta x = x * \frac{S}{T} \\
&\Delta y = y * \frac{S}{T}(同理)
\end{aligned}
$$

---

# Uniswap V2

## 手续费的计算机制

1. 手续费全给项目方

**因为项目方在池子中并没有 share，所以需要进行增发即 $S_m$ 部分**

$S_1$：LP 提供流动性得到的 share 数量   $S_m$：系统按手续费比例增发的 share 数量

$\sqrt{k_1}$：添加流动性对应的 k 值    $\sqrt{k_2}$：添加流动性加上增发的 share 所对应的 k 值

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/381e5453-b993-4092-acca-28f03b189227/Untitled.png)

由上面可得，

$$
\begin{aligned}
\frac{S_m}{S_1+S_m}&=\frac{\sqrt{k_2}-\sqrt{k_1}}{\sqrt{k_2}} \\
S_m*\sqrt{k_2}&=S_1*\sqrt{k_2}+S_m*\sqrt{k_2}-S_1*\sqrt{k_1}-S_m*\sqrt{k_1}(化简) \\
S_m&=\frac{\sqrt{k_2}-\sqrt{k_1}}{\sqrt{k_1}}*S_1
\end{aligned}
$$

1. 手续费全给 Liquidity Provider

**LP 的手续费并不是给 LP 增发新的 share 即 $S_m=0$，而是仍然是 $S_1$ 的数量，但随着手续费的累积，k 值会变大（此时可以理解为，LP 的 share 数量没有变化，在没有手续费收入的时候只共享 $S_1$ 价值，在有手续费收入的时候则共享 $S_1+手续费$ 价值 ）**

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/92756e00-9acd-4b72-b185-e87dafb958a4/Untitled.png)

所以，LP 为 0，项目方为 0

例子：最开始有100 DAI：1 ETH (k 值为 $\sqrt{100}$)，经过一系列的交换，此时池子中有 96 DAI：1.5 ETH (k 值为 $\sqrt{144}$)

1. 项目方拿取一定比例（用 $\phi$ 表示，为 $S_m$ 占整个增发部分的比例）的手续费

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d7feb827-17bb-4040-bd94-765c847ce3bb/Untitled.png)

由上面可得，

$$
\frac{S_m}{S_m+S_1}=\frac{\sqrt{k_2}-\sqrt{k_1}}{\sqrt{k_2}}*\phi
$$

$\frac{S_m}{S_m+S_1}$ 表示的是项目方增发的比例，$\frac{\sqrt{k_2}-\sqrt{k_1}}{\sqrt{k_1}}$ 表示的是 LP 增加的比例

由此可以推导出

$$
\begin{aligned}
S_m*\sqrt{k_2} &= S_m*\sqrt{k_2}*\phi + S_1*\sqrt{k_2}*\phi - S_m*\sqrt{k_1}*\phi - S_1*\sqrt{k_1}*\phi \\
S_m &= \frac{(\sqrt{k_2}-\sqrt{k_1})*S_1}{(\frac{1}{\phi}-1)\sqrt{k_2}+\sqrt{k_1}}
\end{aligned}
$$

当 $\phi=1$ 时，则为手续费全给项目方时所得到的公式

## Spot Price

在使用 x 交换成 y 的时候，显示的价格为 $P_0=\frac{y}{x}$，但实际成交价格为 $P_1=\frac{\Delta y}{\Delta x}$，$P_0$ 和 $P_1$ 之间的差值就是所谓的滑点

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4fb3d862-5dfe-4657-9bcb-03c4e64326b5/Untitled.png)

$$
\begin{aligned}
x*y &= (x+\Delta x)*(y-\Delta y) \\
\frac{\Delta y}{\Delta x} &= \frac{y}{x+\Delta x}
\end{aligned}
$$

当 $\Delta x$ 较小时，我们可以理解为是在计算 $\lim_{\Delta x \to 0}\frac{y}{x+\Delta x}$

## Price Oracle

TWAP (Time Weighted Average Price) 时间权重的平均价格

$P_n$ 是在 $t_n$ 时间点的价格

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6f70a6ef-a1c8-4b10-89e6-29ece68ed260/Untitled.png)

由上面可得，

$$
\sum_{i=0}^{n-1}P_i*(T_{i+1}-T_i)
$$

假如我们想要从 $t_k$ 计算，而不是从 0 计算

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/654bca94-e3fb-47c0-8127-6f4d168f7317/Untitled.png)

由上面可得，

$$
\begin{aligned}
P &= \frac{\sum_{i=k}^{n-1}P_i*(T_{i+1}-T_i)}{T_n-T_k} \\
  &= \frac{\sum_{i=0}^{n-1}P_i*(T_{i+1}-T_i) - \sum_{i=0}^{k-1}P_i*(T_{i+1}-T_i)}{T_n-T_k}
\end{aligned}
$$

通过这个公式，我们可以计算比如一个代币在一小时里的平均价格

## 如何计算 Uniswap V2 的无常损失

**无常损失是出现在添加/移除流动性的情况下，而滑点是出现在两个代币交换的情况下**

假设我们有一个初始 LP 为：100DAI：1ETH，此时 K = 100，$P_E=\frac{100}{1}=100$，两个代币总价值为 100 + 100 = $200

- 当 ETH 涨价时，LP 为：120DAI：0.83ETH，此时 K 不变，$P_E=\frac{120}{0.83}=144.58$，两个代币总价值为 120 + 120 = $240，但如果我们并没有添加流动性而是拿住最开始的 100DAI 和 1ETH，两个代币总价值为 100 + 1 * 144.58 = $244.58，那么 244.58 与 240 的差值就是无常损失的值
- 当 ETH 降价时，LP 为：80DAI：1.25ETH，此时 K 不变，$P_E=\frac{80}{1.25}=64$，两个代币总价值为 80 + 80 =$160，但如果我们并没有添加流动性而是拿住最开始的 100DAI 和 1ETH，两个代币总价值为 100 + 1 * 64 = $164，那么 164 与 160 的差值就是无常损失的值

即在添加流动性所产生的无常损失会导致 ETH 涨价时相比拿住赚得更少，ETH 降价时相比拿住亏得更多

通过上面的例子我们可以抽象出更通用的模型，我们可以列出下面这三个公式，$P_i$ 表示在 i 时刻某个代币的价格，d 表示价格变化的因素 **(当 $0 \lt d \lt 1$ 时表示降价，$d=1$ 时表示价格不变，$d \gt 1$ 时表示涨价)**

$$
\begin{align}
P_1 &= P_0*d \\
x*y &= k \\
P &= \frac{y}{x}
\end{align}
$$

由 (2) (3) 可以计算得到 x 和 y 用价格P 和流动性k 的表达式

$$
\begin{align}
x=\frac{k}{y} \Rightarrow \frac{k}{y}=\frac{y}{P} \Rightarrow y^2 &= k*P \Rightarrow y = \sqrt{k}*\sqrt{P} \\
x &= \frac{y}{P}=\frac{\sqrt{k}*\sqrt{P}}{P}=\frac{\sqrt{k}}{\sqrt{P}}
\end{align}
$$

假设一开始为 $t_0$、$x_0$、$y_0$、$P_0=\frac{y_0}{x_0}$

添加流动性（无手续费）之后为 $t_1$、$x_1$、$y_1$、$P_1=\frac{y_1}{x_1}$

拿住为 $t_{hodl}$、$x_0$、$y_0$、$P_1=\frac{y_1}{x_1}$

将无常损失与价格变化之间的关系函数设为 $f(d)$，V 表示为代币的 value，则

$$
f(d)=\frac{做LP的损失}{拿住之后的价值}=\frac{V_1-V_{hodl}}{V_{hodl}}=\frac{V_1}{V_{hodl}}-1
$$

由 (4) (5) 可以计算出 $V_1$，$V_{hodl}$

$$
\begin{aligned}
V_1=y_1+x_1*P_1 = \sqrt{k_1}*\sqrt{P_1}+\frac{\sqrt{k_1}}{\sqrt{P_1}}*P_1 &= 2*\sqrt{k_1}*\sqrt{P_1} \\
V_{hodl} = y_0+x_0*P_1=\sqrt{k_0}*\sqrt{P_0}+\frac{\sqrt{k_0}}{\sqrt{P_0}}*P_0*d &= (1+d)*\sqrt{k_0}*\sqrt{P_0}
\end{aligned}
$$

所以

$$
\begin{aligned}
f(d) &= \frac{2*\sqrt{k_1}*{\sqrt{P_1}}}{(1+d)*\sqrt{k_0}*{\sqrt{P_0}}}-1 \\
&= \frac{2*\sqrt{k_1}*{\sqrt{P_0*d}}}{(1+d)*\sqrt{k_0}*{\sqrt{P_0}}}-1(因为无手续费，所以k_1和k_0相同) \\
&=\frac{2*\sqrt{d}}{1+d}-1
\end{aligned}
$$

Uniswap 官方文档中给出的[无常损失与价格变化的关系曲线](https://docs.uniswap.org/contracts/v2/concepts/advanced-topics/understanding-returns)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/108e667d-c16a-4853-808a-8c5211f5156b/Untitled.png)

## Flash Swap

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/58c08e10-b85f-421f-912e-7bda2385851d/Untitled.png)

传统的借贷需要用户先超额抵押才能借出代币

**闪电交换是指用户无需质押一分钱即可借出一定数量的代币，例如当你发现有一个套利机会但没有足够的资金去进行超额抵押借贷时，我们可以通过闪电借贷借出 100 个 DAI，然后将这些 DAI 进行一系列的投资等去获取收益如变为 110 个 DAI，此时我们在把借出的 100 个 DAI 及其利息归还，剩下的收益就是自己的也就是 7 个 DAI**

## 使用 Flash Swap 加杠杆

用户持有 3 个 ETH，每个 ETH 的价格为 200 DAI，抵押率为 150%。用户想要 2 倍杠杆

传统借贷

1. 添加 3 ETH 到 Maker Vault
2. 借出 400 DAI 出来
3. 在 Uniswap 把 DAI 换成 ETH
4. 重复 1-3 步

闪电借贷

1. 跟 Uniswap 借 3 个 ETH
2. 把用户的 3 个 ETH 和借的 3 个 ETH 抵押到 Maker Vault
3. 借出 800 个 DAI 出来
4. 还给 Uniswap 600 DAI

## Uniswap V2 代码结构

![Untitled](images/image.png)

Uniswap V2 的核心合约就只有三个，分别是 Router、Factory 和 Pair 合约

图中的三个操作分别为 ① add liquidity ② swap ③ remove liquidity

**用户通过前端页面即页面右边的长方块进行上述的三种操作时，会通过 Router 合约来判断执行的操作并调用相关合约的函数，在路由时合约会判断当前用户执行操作所对应的 Pair 是否已创建，若没有则会先创建一个 Factory 合约来创建相关的 Pair 合约，若有则直接调用对应 Pair 合约的函数**

## Uniswap V2 总结

一个核心：CPAMM

三个操作：添加流动性 / 加密资产交易 / 移除流动性

几个概念：手续费 / Price Oracle / TWAP / Falsh Swap / 无常损失 / 滑点等

Source:https://blog.csdn.net/weixin_51306597/article/details/132067959
## Solidity 中，`memory`、`storage`、`calldata` 和 `indexed`详解

在 Solidity 中，`memory`、`storage`、`calldata` 和 `indexed` 是关键概念，分别涉及数据存储位置和事件日志优化。以下是详细解释：

---

### 1. **`memory`**
- **作用**：声明**临时变量**，仅在函数执行期间存在（类似 RAM）。
- **特点**：
  - 数据在函数调用结束后被清除。
  - 用于处理函数内的**引用类型**（如数组、结构体、字符串）。
  - 可读写。
- **Gas 成本**：读写操作相对便宜（但大数组可能昂贵）。
- **示例**：
  ```solidity
  function process(uint[] memory input) public pure returns (uint) {
      uint[] memory temp = new uint[](input.length); // 临时数组
      temp[0] = input[0]; // 可修改
      return temp[0];
  }
  ```

---

### 2. **`storage`**
- **作用**：指向**合约状态变量**的永久存储（类似硬盘）。
- **特点**：
  - 数据存储在区块链上，永久存在。
  - 用于修改合约的状态变量。
  - 可读写。
- **Gas 成本**：读写操作昂贵（尤其是写入）。
- **示例**：
  ```solidity
  contract Example {
      uint[] public data; // 状态变量（默认在 storage）

      function updateStorage() public {
          uint[] storage ref = data; // 指向 storage 的引用
          ref.push(10); // 直接修改状态变量
      }
  }
  ```

---

### 3. **`calldata`**
- **作用**：**只读**的临时数据位置，用于存储**函数参数**。
- **特点**：
  - 仅适用于**外部函数**（`external`）的参数。
  - 数据来自交易调用（`msg.data`），不可修改。
  - 最省 Gas 的选择（避免复制数据）。
- **示例**：
  ```solidity
  function readOnly(uint[] calldata arr) external pure returns (uint) {
      return arr[0]; // 只能读取，不能修改
  }
  ```

---

### 4. **`indexed`**
- **作用**：标记**事件参数**，使其可被索引（用于日志过滤）。
- **特点**：
  - 一个事件最多 3 个 `indexed` 参数。
  - 索引参数存入日志的 `topics`（而非 `data`），便于链下检索。
  - 适合过滤地址、标识符等简单类型（复杂类型会哈希）。
- **示例**：
  ```solidity
  event Transfer(
      address indexed from,    // 可索引（前端可过滤）
      address indexed to,      // 可索引
      uint value               // 非索引（存储在 data 中）
  );

  // 前端过滤（如 ethers.js）：
  contract.on("Transfer", (from, to, value) => { ... });
  contract.queryFilter("Transfer", { from: someAddress }); // 按 from 过滤
  ```

---

### 关键对比
| 特性          | `memory`         | `storage`       | `calldata`      | `indexed`（事件专用） |
|---------------|------------------|-----------------|-----------------|----------------------|
| **生命周期**  | 函数执行期间     | 永久（链上存储）| 函数执行期间    | 事件日志存储         |
| **读写权限**  | 读写             | 读写            | **只读**        | 只读（日志）         |
| **适用场景**  | 函数内部临时变量 | 状态变量        | 外部函数参数    | 事件参数             |
| **Gas 成本**  | 中等             | 高（尤其写入）  | **最低**        | 增加日志 Gas 成本    |
| **修改影响**  | 不影响状态       | 修改合约状态    | 无影响          | 无直接影响           |

---

### 最佳实践
1. **优先用 `calldata`**：  
   外部函数的参数（如数组）使用 `calldata` 节省 Gas。
2. **避免 `storage` 复制**：  
   直接操作状态变量时，用 `storage` 引用减少复制开销。
3. **`indexed` 用于高频过滤字段**：  
   如地址、交易 ID 等需快速检索的参数。

# 2025-08-12

参加技术和运营组会
整理前端常用命令
## Cursor

### 终端

在/Users/ruoyan/Library/Application Support/Cursor/User/keybindings.json中修改快捷键
#### 创建一个新的cusor内置终端并聚焦
```json
{
  "key": "ctrl+cmd+n",
  "command": "workbench.action.terminal.new"
}
```

#### 聚焦到cursor内置终端
```json
{
  "key": "alt+w",
  "command": "workbench.action.terminal.focus"
}
```

#### 在cursor内置的终端之间切换
```json
{
  "key": "alt+q",
  "command": "workbench.action.terminal.focusNext"
}
```

#### 关闭当前cursor内置的终端
```json
{
  "key": "alt+r",
  "command": "workbench.action.terminal.kill"
}
```

### AI CHAT

#### 打开ai chat页面，同时引用选定当前内容，如果已经打开则隐藏ai chat页面
```json
{
  "key": "cmd+l",
  "command": "aichat.newchataction"
}
```

### PAGE

#### 在打开的页面（标签）中切换
```json
{
  "key": "ctrl+tab",
  "command": "workbench.action.quickOpenNavigateNextInEditorPicker",
  "when": "inEditorsPicker && inQuickOpen"
}
```

#### 聚焦到当前页面（标签）
```json
{
  "key": "alt+e",
  "command": "workbench.action.focusActiveEditorGroup",
  "when": "terminalFocus || panelFocus || sideBarFocus"
}
```

#### 关闭当前页面（标签）
```json
{
  "key": "alt+x",
  "command": "workbench.action.closeActiveEditor",
  "when": "editorIsOpen"
}
```

#### 打开上一个页面（标签）
```json
{
  "key": "alt+cmd+left",
  "command": "workbench.action.nextEditor"
}
```

#### 打开下一个页面（标签）
```json
{
  "key": "alt+cmd+right",
  "command": "workbench.action.nextEditor"
}
```

#### 将焦点置于主侧栏
```json
{
  "key": "cmd+0",
  "command": "workbench.action.focusSideBar"
}
```

#### 查看: 聚焦到面板中
```json
{
  "key": "alt+t",
  "command": "workbench.action.focusPanel"
}
```


## foundry


### forge

**典型工作流**：
```bash
forge init my_project     # 初始化项目
forge build               # 编译合约
forge test                # 运行测试（支持模糊测试）
forge test --mt ${函数名称} -vvvvv  # 测试指定测试合约中过的函数
forge script Deploy       # 部署脚本执行
forge fmt                 # 代码格式化
forge install OpenZeppelin/openzeppelin-contracts
forge selectors find "balanceOf(address)"  # 0x70a08231
```

### cast

**常用场景**：
```bash
cast send <合约> "函数" 参数    # 发送交易
cast call <合约> "视图函数"      # 查询状态
cast block latest              # 获取最新区块
cast --to-base 0x1 hex         # 数据格式转换
cast sig "transfer(address)"   # 计算函数选择器
```

### anvil

```bash
anvil    # 本地虚拟网络
```

### make

生成如下makefile
```bash
include .env

deploy_ruoyancoin:
    @echo "deploying contract..."
    forge create src/ruoyancoin.sol:ruoyancoin --private-key ${OWNER_PRIVAYE_KEY} --broadcast --constructor-args ${OWNER_ADDRESS}

mint:
	@echo "Minting tokens..."
	cast send ${CONTRCT_ADDRESS} "mint(uint256)" ${MINT_AMOUNT} --private-key ${OWNER_PRIVATE_KEY}

```


## react



## next.js 15

## wagmi (建议用这个构建web3前端,底层是ether.js)

## rainbowkit

# 2025-08-11

- [ ] 回顾肖臻老师的比特币原理讲解
- [ ] Deploy myFirstNft
- [ ] 编写针对运营/BD方向的简历
- [ ] 参加以太坊中文周会，了解最新行业动态
https://cmu-ruoyan-lgl.github.io/Web3_Internship_Program_Notes/StudyNotes/day10

# 2025-08-10

部署在线简历
- github page
- 针对技术和运营方向
- 中英文版本i18n
- 申请新的邮箱

# 2025-08-09

# Day 8 - 2025年08月09日 

---

## 今日学习目标

- [ ] 深度研究web3前端开发主流框架
- [ ] 学习前端知识
- [ ] 整理相关框架笔记

## 明日学习目标 
- [ ] 黑客松组队设计
- [ ] 设计NFT交易市场项目： ERC721，IPFS，contract， wagmi， viem@2.x， react， tailwind css， gasp 实现加入到市场的nft能够滚动显示， 要求实现NFT定价代理出售， 撤回代理， 购买市场的NFT等功能

## 学习内容

[Build an Ethereum DApp with Next.js, Wagmi, RainbowKit, and Pinata to Mint NFTs on IPFS](https://www.youtube.com/watch?v=2BWdIrfJ3FI&t=60s)


## 学习笔记


---

### 🧩 **一、开源项目与模板（可快速复用）**  
1. **The Stripes NFT 铸造应用**  
   - **特点**：基于 React.js 的高度可配置铸造平台，支持自定义主题色、背景图和多链部署（如 Polygon）。  
   - **技术栈**：React.js + 智能合约集成，通过修改 `config.json` 即可对接自定义合约。  
   - **适用场景**：个人艺术家发行 NFT 或游戏道具确权平台，适合展示合约交互能力。  

2. **The-Weirdos-NFT-Website-Starter-Code**  
   - **特点**：响应式 NFT 集合展示页，含动态效果（打字机动画、彩带特效）和移动端适配。  
   - **技术栈**：React + styled-components + GSAP 动画库，Create React App 脚手架快速启动。  
   - **亮点**：社区活跃，被提名 GitHub 顶级 NFT 开源项目，提供详细视频教程。  

3. **Open Source NFT Marketplace（社区驱动型）**  
   - **特点**：开源 NFT 市场模板，支持 AR 预览 3D 作品，专注跨平台兼容性。  
   - **技术栈**：ReactJS + 以太坊钱包集成，未来计划扩展智能合约后端。  
   - **创新点**：增强现实技术提升用户体验，适合沉浸式艺术展示场景。  

---

### 💼 **二、商业模板（设计成熟，可直接购买）**  
1. **Xhibiter**  
   - **版本**：提供 **HTML 模板**（Tailwind CSS + Webpack）和 **React NextJS 模板**（v13.2）。  
   - **优势**：  
     - 极速加载（500ms 内），Google 评分 97，支持暗黑模式。  
     - 含 13 个主页变体、实时拍卖倒计时、Metamask 集成。  
   - **适用**：高并发交易市场，适合求职 DeFi/NFT 公司时展示性能优化能力。  

2. **Netstorm**  
   - **特点**：多页式 React 模板（17+ 内页），专为数字藏品竞拍设计，含现场拍卖过滤器和作者面板。  
   - **技术栈**：ReactJS + Bootstrap，支持 REST API 调用。  
   - **亮点**：CSS3 动画流畅，内置 1500+ 图标库，适合复杂业务场景前端架构参考。  

3. **Niopy**  
   - **特点**：Vue.js 构建的 15 合 1 模板，含用户面板和明暗模式切换，侧重创作者仪表盘设计。  
   - **技术栈**：Vue JS + Bootstrap 5 + SASS。  
   - **适用**：艺术家个人品牌或收藏品管理平台，突出用户中心功能。  

4. **NFTPO**  
   - **特点**：轻量级登陆页模板，专为 NFT 项目预热设计，含半透明导航栏与渐变按钮。  
   - **技术栈**：NextJS + TailwindCSS，优化 SEO 与图片加载速度。  
   - **优势**：适合快速搭建 MVP，强调第一视觉冲击力。  

---

### ✨ **三、设计亮点与趋势参考**  
- **视觉动效**：  
  - **GSAP 动画**（如 The-Weirdos 的滚动特效）提升用户参与感。  
  - **微交互设计**：NFTPO 的渐变按钮、Netstorm 的 CSS3 悬停动画。  
- **AR 与 3D 集成**：开源 NFT 市场的 AR 预览功能，适合结合 three.js 实现虚拟画廊。  
- **暗黑模式**：Xhibiter 和 Niopy 的双模式切换，适配加密用户偏好。  
- **模块化布局**：Xhibiter 的组件库（图表、下拉菜单）可复用性高。  

---

### ⚙️ **四、技术栈参考（2025 主流选择）**  
| **需求**         | **推荐技术**                     | **案例参考**              |  
|------------------|----------------------------------|---------------------------|  
| 多链兼容         | Wagmi + Viem                     | Xhibiter 多链资产展示     |  
| 动画与交互       | GSAP + Framer Motion             | The-Weirdos 动态效果      |  
| 样式管理         | Tailwind CSS + styled-components | NFTPO 渐变设计            |  
| 性能优化         | NextJS 服务端渲染                | Xhibiter React 版         |  
| 移动端优先       | 响应式框架 + 触控优化            | Niopy 跨设备适配          |  

---

### 💡 **实用建议**  
1. **优先学习开源项目**：克隆 [The-Weirdos 代码库](https://github.com/codebucks27/The-Weirdos-NFT-Website-Starter-Code) 修改组件，理解状态管理与动画实现。  
2. **商用模板二次开发**：购买 Xhibiter React 版（约 $60），在其模块化基础上扩展拍卖逻辑或 DAO 治理功能。  
3. **融合创新点**：  
   - 在铸造平台中增加 **AR 预览**（参考开源 NFT 市场）。  
   - 为交易市场添加 **Gas 优化提示**（类似 Netstorm 的实时过滤器）。  

根据目标公司调整设计重点：应聘艺术类平台侧重 3D/AR 能力；金融类 NFT 项目突出交易流程与数据可视化。


## 问题与思考


## 总结


---
https://cmu-ruoyan-lgl.github.io/Web3_Internship_Program_Notes/StudyNotes/day8

# 2025-08-08

- [ ] 复习react
- [ ] 研究以太坊官网文档，后续白皮书 + EPF wiki + EVM图解
- [ ] 研究设计dex实现
https://cmu-ruoyan-lgl.github.io/Web3_Internship_Program_Notes/StudyNotes/day7

# 2025-08-07

deploy myFirstNft
完成前端项目
复习react
https://cmu-ruoyan-lgl.github.io/Web3_Internship_Program_Notes/StudyNotes/day6

# 2025-08-06

- [ ] 参加会议
- [ ] 研究deploy myFirstNft
- [ ] 完成前端项目
https://cmu-ruoyan-lgl.github.io/Web3_Internship_Program_Notes/StudyNotes/day5

# 2025-08-05

参加web3安全分享会
写完solana monitor api代码
开始deploy myfirstnft
https://cmu-ruoyan-lgl.github.io/Web3_Internship_Program_Notes/StudyNotes/day4

# 2025-08-04

体验一次eth中文周会
学习成都信息工程大学梁培利老师的uniswapv1到v3的原理
设计agent去做NFT市场交换前端项目的工作流
学习笔记收录在https://cmu-ruoyan-lgl.github.io/Web3_Internship_Program_Notes/StudyNotes/day3


# 2025.07.31


<!-- Content_END -->
