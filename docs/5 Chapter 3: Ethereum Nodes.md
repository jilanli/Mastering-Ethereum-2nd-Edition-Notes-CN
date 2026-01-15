# 第三章：以太坊节点 (The Ethereum Node)

以太坊节点（Ethereum node）是一个实现了以太坊规范，并通过 **P2P**（点对点）网络与其他以太坊节点进行通信的软件应用程序。

最初，一个节点只需运行单个客户端即可完全实现作为以太坊生态系统成员的所有要求。2022 年 9 月 15 日，“合并”（The Merge）硬分叉发生，将共识协议从基于 **PoW**（工作量证明）的方案更改为 **Gasper**——即新的基于 **PoS**（权益证明）的共识协议。这也导致了关注点分离（separation of concerns）——即共识与执行的分离，并产生了一种新型的以太坊客户端：共识客户端（consensus client）。

因此，在撰写本文时，一个以太坊节点必须同时运行两个软件组件才能与最新的规范兼容，如图 3-1 所示，其定义如下：

**共识客户端 (Consensus client)**

这款新软件现在负责共识协议，让所有节点对区块链的单一历史记录达成一致。[^1]

**执行客户端 (Execution client)**

该软件专注于接收网络上发生的所有区块和交易，在 **EVM**（以太坊虚拟机）内部执行它们，并验证其正确性。
![Figure 3-1](<./images/figure 3-1.png>)
[图 3-1：合并后以太坊节点的架构 (The architecture of a post-Merge Ethereum node)]

不同的以太坊客户端——包括执行客户端和共识客户端——只要符合参考规范和标准化的通信协议，就可以互操作。虽然这些不同的客户端由不同的团队使用不同的编程语言开发，但它们都“说”同一种协议并遵循相同的规则。因此，它们都可以用于运行以太坊网络并与之交互。

以太坊是一个开源项目，所有主要客户端的源代码均在开源许可证（如 LGPL v3.0）下发布，可供任何人免费下载和用于任何目的。不过，开源不仅意味着免费使用，还意味着以太坊是由一个开放的志愿者社区开发的，任何人都可以对其进行修改。“更多的眼睛”意味着更值得信赖的代码。

以太坊最初由一份名为“黄皮书”（Yellow Paper）的正式规范定义，该规范由本书原作者之一加文·伍德（Gavin Wood）编写。尽管该规范会随着以太坊重大变更而定期更新，但现在已有清晰的路径指向两种不同的参考实现：一种用于执行客户端，另一种用于共识客户端。这些参考实现使用 Python 编写，并优先考虑可读性和简洁性。

> [!NOTE]
> 这些规范（Specs）并非旨在作为全节点的完整实现，而是作为“可执行的伪代码规范”。

这与比特币形成了对比，例如，比特币没有以任何正式方式进行定义。比特币的“规范”就是其参考实现 `Bitcoin Core`；而以太坊的执行规范记录在一篇结合了英文和数学（形式化）说明的论文中。这份形式化规范，加上各种**以太坊改进提案** (**EIPs**, Ethereum Improvement Proposals) 以及用 Python 编写的新共识规范，共同定义了以太坊节点的标准行为。

由于以太坊拥有清晰的形式化规范，产生了一系列独立开发但可互操作的客户端软件实现。以太坊在网络上运行的实现方案比任何其他区块链都更加多样化，这通常被认为是一件好事。事实上，这已被证明是抵御网络攻击的绝佳方式，因为针对特定客户端实现策略的漏洞利用只会让开发者忙于修补，而其他客户端则能让网络运行几乎不受影响。

---

> [!TIP]
> **译者注**：
> 1. **客户端多样性 (Client Diversity) 带来的鲁棒性**：从后端高可用架构的角度来看，以太坊的这种设计有效避免了“单点故障”。如果全网 90% 的节点都运行同一个客户端（如 `Geth`），一旦该客户端出现导致共识错误的逻辑漏洞，整个网络都会瘫痪。通过保持执行层（Geth, Nethermind, Besu 等）和共识层（Prysm, Lighthouse, Teku 等）的多样性，以太坊在协议层实现了真正的“多活冗余”。
> 2. **可执行规范 (Executable Spec) 的优势**：对于后端开发者而言，阅读复杂的数学公式（黄皮书）门槛较高。以太坊转向使用 Python 编写参考实现（如 `execution-specs`），意味着规范本身就是可以运行的代码。这极大地方便了开发者通过单元测试或模拟执行来验证自己的第三方实现是否完全符合共识规则。
> 3. **引擎 API (Engine API) 与解耦**：从后端架构视角看，合并后的节点实际上演变成了一个微服务架构。共识客户端和执行客户端之间通过 `Engine API`（基于 JSON-RPC）进行通信。这种解耦允许开发者自由组合不同的客户端（例如使用 `Lighthouse` 作为共识层，配合 `Geth` 作为执行层），极大地增强了系统的健壮性和客户端多样性。
> 4. **执行层的本质**：虽然共识层决定了区块的顺序，但作为后端开发者，我们交互最频繁的依然是执行层。执行层维护着完整的状态树（State Tree），所有的合约调用、账户余额查询和 `eth_call` 操作最终都是由执行客户端在本地 **EVM** 中跑完并返回结果的。
---
### 以太坊网络 (Ethereum Networks)

目前存在多种基于以太坊的网络，它们在很大程度上符合最初以太坊“黄皮书”（Yellow Paper）中定义的正式规范，但彼此之间可能互操作，也可能不互操作。

许多 **EVM** 兼容链（如 Ethereum Classic, BNB Chain 和 Polygon）共享大部分执行规范（execution spec），但在共识机制和参数上往往有所不同。虽然它们在协议层基本兼容，但这些网络通常具有某些特性或属性，要求以太坊客户端软件的维护者进行小幅修改以支持各个网络。正因如此，并非每个版本的以太坊客户端软件都能运行所有基于以太坊的区块链。

截至 2025 年 6 月，以太坊执行协议有五种主要实现，使用四种不同的语言编写；以太坊共识协议也有五种实现，使用五种不同的语言编写：

**执行客户端 (Execution clients) 包括：**

* **Geth**：使用 Go 语言编写
* **Nethermind**：使用 C# 语言编写
* **Besu**：使用 Java 语言编写
* **Erigon**：使用 Go 语言编写
* **Reth**：使用 Rust 语言编写

**共识客户端 (Consensus clients) 包括：**

* **Lighthouse**：使用 Rust 语言编写
* **Lodestar**：使用 TypeScript 语言编写
* **Nimbus**：使用 Nim 语言编写
* **Prysm**：使用 Go 语言编写
* **Teku**：使用 Java 语言编写

在本节中，我们将重点介绍以下两种执行客户端：

* **Geth**：最古老且使用最广泛的执行客户端，由以太坊基金会维护。
* **Reth**：在 Parity/OpenEthereum 停止维护后，由 Paradigm 创建的基于 Rust 的新型执行客户端。

以及以下两种共识客户端：

* **Prysm**：首个共识客户端，现由 Offchain Labs 维护。
* **Lighthouse**：目前使用率最高的共识客户端，由 Sigma Prime 维护。

我们将演示如何使用这些客户端搭建节点。具体来说，我们将使用 `Geth-Prysm` 和 `Reth-Lighthouse` 组合，并探索它们的部分命令行选项和 **API**。

> [!NOTE]
> 这些配对仅作为示例；你可以根据自己的喜好，选择组合任何执行客户端和共识客户端来运行以太坊节点。

---

> [!TIP]
> **译者注**：
> 1. **多语言实现的工程意义**：从后端稳定性角度看，这种“多语言实现”是极高等级的安全策略。如果某种编程语言的底层库（例如 Go 的内存分配器或 Java 的虚拟机）出现漏洞，只会影响特定客户端，而不会拖垮整个以太坊网络。在金融级后端设计中，这被称为“异构冗余”。
> 2. **Reth 的崛起与性能优化**：作为后端开发者，值得关注 **Reth** 这种新一代 Rust 客户端。它利用了 Rust 极致的性能和零成本抽象（Zero-cost abstractions），在数据存取性能和同步速度上远超传统客户端。在处理高并发 **JSON-RPC** 请求或构建高性能索引层时，选择 Rust 驱动的客户端已成为目前行业的技术风向标。

---
### 我应该运行全节点吗？ (Should I Run a Full Node?)

区块链的健康度、韧性和抗审查能力取决于是否有大量独立运营且地理分散的**全节点**（Full Nodes）——即下载整个区块链并无限期保留数据的节点。每个全节点都可以帮助其他新节点获取区块数据以引导其运行，并为运营商提供对所有交易和合约的权威且独立的验证。

> [!NOTE]
> 准确地说，这些节点之间存在细微区别：
> * **存档节点 (Archive nodes)**：无限期保留所有历史状态数据的以太坊节点。
> * **全节点 (Full nodes)**：会定期裁剪（Discard）历史状态和收据的以太坊节点——这通常是你启动节点时的默认选项。

然而，运行全节点会产生硬件资源和带宽成本。截至 2025 年 6 月（具体取决于客户端配置），全节点必须下载至少 **2 TB** 的数据并将其存储在本地硬盘上。随着新交易和区块的不断加入，这种数据负担每天都在迅速增加。我们将在后面的“全节点的硬件要求”章节中更详细地讨论这一主题。

对于以太坊开发来说，运行一个连接主网（Mainnet）的全节点并不是必须的。你几乎可以使用以下替代方案完成所有开发工作：

* **测试网节点**（连接到较小的公共测试区块链）。
* **本地私有区块链**（如 `Anvil`）。
* **托管节点 API**（由 **Infura** 或 **Alchemy** 等服务提供商提供）。

你还可以选择运行**远程客户端**（Remote Client），它不会在本地存储区块链副本，也不会验证区块和交易。这些客户端提供钱包功能，可以创建和广播交易。远程客户端可用于连接现有网络，例如你自己的全节点、公共区块链、公共或许可制（**PoA**，权威证明）测试网，或私有本地区块链。在实践中，你可能会使用 **MetaMask**、**Rabby Wallet** 或 **Coinbase Wallet** 等远程客户端，作为在不同节点选项之间切换的便捷方式。

“远程客户端”和“钱包”这两个术语常被互换使用，尽管它们存在细微差别。通常，远程客户端除了提供钱包的交易功能外，还会提供 **API**（如 `web3.js` API）。

不要将以太坊中的“远程客户端”概念与“轻客户端”（Light Client，类似于比特币中的 **SPV** 简单支付验证客户端）混淆。轻客户端会验证区块头，并使用 **Merkle** 证明来验证交易是否被包含在区块链中及其产生的影响，从而获得与全节点相似的安全级别。相反，以太坊远程客户端不验证区块头或交易。它们完全信任全节点来提供区块链访问权限，因此会丧失显著的安全性。你可以通过运行自己的全节点来缓解这些问题。

#### 全节点、公共测试网与本地仿真环境的优劣势分析 (Advantages and Disadvantages of Node Options)

选择如何连接以太坊取决于你的开发需求。你可以运行全节点来支持网络，也可以使用测试网或本地仿真环境来加速开发。

---

#### 全节点的优缺点 (Full Node Advantages and Disadvantages)

运行全节点有助于你所连接网络的运行，但也会为你带来轻度到中度的成本。

**优点：**

* **增强鲁棒性**：支持以太坊网络的韧性和抗审查能力。
* **权威验证**：权威地验证所有交易，不依赖任何第三方。
* **无中介交互**：无需中介即可与公共区块链上的任何合约交互。
* **直接部署**：可以直接将合约部署到公共区块链中。
* **离线查询**：可以离线查询（只读）区块链状态（账户、合约等）。
* **隐私保护**：查询区块链时不会让第三方知道你正在读取的信息。

**缺点：**

* **资源消耗**：需要大量且不断增长的硬件和带宽资源。
* **同步耗时**：首次启动时可能需要数天时间才能完全同步。
* **维护成本**：必须进行维护、升级并保持在线以维持同步状态。

---

#### 公共测试网的优缺点 (Public Testnet Advantages and Disadvantages)

无论你是否选择运行全节点，你可能都想运行一个公共测试网节点。

**优点：**

* **轻量化**：测试网节点需要同步和存储的数据少得多——根据网络不同，大约为 100–300 GB（截至 2025 年 6 月）。
* **快速启动**：测试网节点可以在几小时内完成全同步。
* **零成本开发**：部署合约或进行交易需要“测试以太币”（Test Ether），它没有价值，可以从各种“水龙头”（Faucets）免费获取。
* **真实环境**：测试网是拥有许多其他用户和合约的公共区块链，处于“生存”运行状态。

**缺点：**

* **无经济博弈**：测试网上不能使用“真钱”。因此，你无法针对真正的对手测试安全性，因为没有利益风险（Nothing at stake）。
* **仿真限制**：公共区块链的某些方面无法在测试网上进行真实测试。例如，虽然发送交易需要手续费，但在测试网上这并非考量因素（因为 Gas 是免费的）。此外，测试网不会像公共主网那样经常出现网络拥堵。
* **环境差异**：某些测试网是为特定目的设计的，可能与以太坊主网略有不同。

---

#### 本地区块链仿真的优缺点 (Local Blockchain Simulation Advantages and Disadvantages)

对于许多测试目的，最佳选择是启动单实例私有链。**Anvil** 是目前最流行的本地区块链仿真工具之一。

**优点：**

* **零延迟**：无需同步，磁盘上几乎没有数据；你自己产出第一个区块。
* **资金自由**：无需寻找水龙头；你可以给自己“奖励”区块奖励用于测试。
* **绝对私密**：没有其他用户，只有你。
* **按需部署**：没有其他合约，只有你在启动后亲自部署的那些。

**缺点：**

* **缺乏竞争**：没有其他用户意味着本地链的行为与公共链不同。不存在对交易空间或交易排序（Sequencing）的竞争。
* **出块预判**：除了你之外没有其他区块生产者，意味着出块更加可预测；因此，你无法测试某些在公共区块链上发生的极端场景。
* **冷启动负担**：没有其他合约意味着你必须部署所有想要测试的内容，包括依赖项和库。

---

> [!TIP]
> **译者注**：
> 幸运的是，像 **Anvil** 这样的工具允许你在任意区块高度“分叉”（Fork）以太坊主网链，让你在类主网的状态下实验你的智能合约。
> 1. **主网分叉（Mainnet Forking）的妙用**：从后端集成测试的角度看，`Anvil` 的分叉功能是神技。它允许你直接在本地模拟环境中调用主网上已有的 Uniswap 或 Aave 合约，而不需要自己在本地重新部署整套复杂的 DeFi 协议。这极大降低了编写集成测试的成本。
> 2. **隐私与 MEV 考量**：对于生产环境的后端服务，运行全节点的物理成本虽然高，但在隐私保护上具有不可替代的价值。如果你使用 Infura 等公共服务，你的所有查询请求（比如查询某个大户的余额）都会暴露给服务商。对于需要防御 **MEV**（最大可提取价值）攻击的交易发送逻辑，拥有自己的全节点是第一道防线。
> 3. **全节点 vs 存档节点的状态剪枝 (Pruning)**：从后端运维视角看，全节点之所以能在 2 TB 左右维持平衡，是因为它使用了“剪枝”技术，只保留最近的区块状态，而将旧的状态删除。如果你需要查询一年多以前某个账户在特定高度的余额（即执行 `eth_getBalance` 时的 `blockNumber` 参数很久远），普通全节点会返回错误，此时你必须使用占用空间高达 12 TB+ 的**存档节点**。
> 4. **JSON-RPC 的信任链**：作为后端开发者，当你调用托管服务（如 **Infura**）时，本质上是将你的逻辑层建立在对他人的“信任”之上。在处理大额资产充值回调或关键业务逻辑时，建议至少在本地部署一个全节点作为数据校验的“真相来源”（Source of Truth），以规避服务商接口延迟或被中间人攻击的风险。
---
### 运行以太坊节点 (Running an Ethereum Node)

如果你有足够的时间和资源，你应该尝试运行一个全节点（Full Node），哪怕仅仅是为了深入了解这一过程。在本节中，我们将介绍如何下载、编译并运行以太坊客户端组合：`Geth-Prysm` 和 `Reth-Lighthouse`。
这要求你对操作系统的**命令行界面**（CLI）有一定的了解。无论你选择将它们作为全节点、测试网节点，还是作为本地私有链的客户端运行，安装这些客户端都是非常值得的。

#### 全节点的硬件要求 (Hardware Requirements for a Full Node)

在开始之前，你应该确保你的计算机拥有足够的资源来运行一个以太坊全节点。你至少需要 **2 TB** 的磁盘空间来存储以太坊区块链的完整副本。如果你还想在以太坊测试网上运行全节点，则至少需要额外的 **100–400 GB**。下载 **2 TB** 的区块链数据可能需要很长时间，因此建议在高速网络连接下进行。

同步以太坊区块链是极度消耗输入/输出 (**I/O**) 性能的。最好配备固态硬盘 (**SSD**)。如果你使用的是机械硬盘 (**HDD**)，则至少需要 **8 GB** 的 **RAM** 作为缓存。否则，你可能会发现系统速度太慢，无法跟上进度并完成同步。

以下是同步以太坊区块链全副本的最低要求摘要：

* **CPU**：2 核或更多
* **存储空间**：至少 **2 TB** 剩余空间
* **内存 (RAM)**：配合 **SSD** 至少 **8 GB**；若使用 **HDD** 则需 **8 GB** 以上（强烈建议使用 **SSD**）
* **网络**：7 Mbps 以上的下载带宽

如果你希望在合理的时间内完成同步，并存储本书讨论的所有开发工具、库、客户端和区块链，你需要性能更强的计算机。以下是我们推荐的配置：

* **CPU**：4 核以上的高速 **CPU**——主频比核心数更重要
* **内存 (RAM)**：**16 GB** 以上
* **存储空间**：至少 **2 TB** 剩余空间的快速 **NVMe SSD**
* **网络**：24 Mbps 以上的下载带宽

很难预测区块链规模增长的速度以及何时需要更多磁盘空间，因此建议在开始同步之前检查区块链的最新大小。

> [!NOTE]
> 这里列出的磁盘大小要求假设你使用的是默认设置运行节点，即对旧的“状态数据”（State data）进行了“裁剪”（Pruned）。如果你运行的是“存档”（Archival）节点（保留磁盘上的所有状态），根据客户端的不同，可能需要超过 **2 TB**（甚至高达 **12–15 TB**）的空间。在运行节点之前，请务必参考官方客户端网站上的最新硬件要求。

#### 构建与运行客户端的软件要求 (Software Requirements for Building and Running a Client)

本节涵盖了 `Geth-Prysm` 和 `Reth-Lighthouse` 客户端软件。它还假设你使用的是类 **Unix** 命令行环境。示例显示了在运行 **Bash shell**（命令行执行环境）的 **macOS** 上的命令和输出。这些说明在大多数 **Linux** 发行版上无需修改即可运行。**Windows** 用户可以使用 **Windows Subsystem for Linux (WSL2)**。

> [!TIP]
> 在本章的许多示例中，我们将使用操作系统的 **CLI**（也称为 **shell**），通过终端应用程序访问。**shell** 会显示一个提示符；你输入命令，**shell** 回应一些文本并显示一个新提示符供你输入下一个命令。你的系统上的提示符可能看起来不同，但在以下示例中，它由 `$` 符号表示。在示例中，当你看到 `$` 符号后的文本时，请不要输入 `$` 符号，而是输入紧随其后的命令（以粗体显示），然后按回车键执行。示例中，每条命令下方的行是操作系统对该命令的响应。当你看到下一个 `$` 前缀时，你就知道这是一个新命令，应该重复此过程。

在开始之前，你可能需要安装一些软件。如果你从未在当前使用的计算机上进行过任何软件开发，你可能需要安装一些基础工具。对于接下来的示例，你需要安装 `git`（源代码管理系统）、`golang`（**Go** 语言及其标准库）以及 **Rust**（一种系统级编程语言）。

以下是本示例中我们将使用的四个客户端的文档页面：

* **Geth**
* **Prysm**
* **Reth**
* **Lighthouse**

你可以随时查阅这些网站以了解各客户端架构的更多细节，并在安装过程中进行故障排除。

#### 准备阶段 (Preparation Phase)

从你的家目录（home directory）开始，在计算机中创建一个名为 `ethereum-node1` 的文件夹，然后在其中创建两个名为 `execution`（执行层）和 `consensus`（共识层）的子文件夹：

```bash
$ mkdir ethereum-node1
$ cd ethereum-node1
$ mkdir execution
$ mkdir consensus

```

现在你的文件夹结构应该是这样的：

```text
ethereum-node1
├── consensus
└── execution

```

对一个名为 `ethereum-node2` 的新文件夹重复上述步骤：

```bash
$ cd .. # 此命令用于返回家目录
$ mkdir ethereum-node2
$ cd ethereum-node2
$ mkdir execution
$ mkdir consensus

```

最后，你应该有两个根文件夹——`ethereum-node1` 和 `ethereum-node2`——且每个根文件夹下都有两个子文件夹：`execution` 和 `consensus`。

你还需要安装 **Go** 和 **Rust**。你可以查看它们的官方网站以获取安装指南。

#### Geth-Prysm

进入新建的文件夹 `ethereum-node1`：

```bash
$ cd ethereum-node1

```

##### Geth

首先，我们将通过源代码编译来安装 **Geth**。**Geth** 是执行规范的 **Go** 语言实现，由以太坊基金会积极开发，因此被认为是以太坊客户端的“官方”实现。通常，每个基于以太坊的区块链都会有自己的 **Geth** 实现。如果你正在运行 **Geth**，那么你需要使用以下存储库链接之一来确保为你的区块链获取正确的版本：

* [Ethereum](https://github.com/ethereum/go-ethereum)
* [BNB Chain](https://github.com/bnb-chain/bsc)
* [Polygon PoS](https://github.com/maticnetwork/bor)

> [!NOTE]
> 你可以跳过这些指令，直接为你选择的平台安装预编译好的二进制文件。预编译版本更容易安装，可以在此处列出的任何存储库的“releases”部分找到。但是，通过自己下载并编译软件，你可以学到更多。

**克隆存储库**。第一步是克隆 **Git** 存储库以获取源代码副本。在 `execution` 子文件夹中使用如下 `git` 命令在你本地克隆所选存储库：

```bash
$ cd execution
$ git clone https://github.com/ethereum/go-ethereum.git

```

在存储库复制到本地系统时，你应该会看到进度报告。

**从源代码构建 Geth**。要构建 **Geth**，请切换到源代码下载目录，在选择最新版本（目前是 `v1.14.3`，但你可以随时检查最新版）后使用 `make` 命令：

```bash
$ cd go-ethereum
$ git checkout v1.14.3
$ make geth

```

如果一切顺利，你会看到 **Go** 编译器构建每个组件，直到生成 **Geth** 可执行文件。构建完成后，让我们在不实际启动它的情况下确保 **Geth** 工作正常：

```bash
$ ./build/bin/geth version

```

你应该会看到类似于 `GethVersion: 1.14.3-stable` 的版本报告。先不要运行 **Geth**，因为我们还需要安装共识客户端，才能让以太坊节点同步到链尖。

#### Prysm

现在轮到共识客户端了。**Prysm** 是共识规范的 **Go** 语言实现，由 **Offchain Labs** 积极开发。起初，它是合并（The Merge）后使用最广泛的共识客户端。现在，得益于社区提升客户端多样性的巨大努力，它的市场份额大幅下降，目前维持在 **37%**。

**安装二进制文件**。**Prysm** 可以像 **Geth** 那样从源代码构建，但过程稍微复杂一些。建议按照以下方法安装。首先进入 `consensus` 文件夹：

```bash
$ cd ../.. # 此命令用于返回 ethereum-node1 文件夹
$ cd consensus

```

运行以下命令：

```bash
$ curl https://raw.githubusercontent.com/prysmaticlabs/prysm/master/prysm.sh --output prysm.sh && chmod +x prysm.sh

```

**生成 JWT 密钥 (JWT Secret)**。组成以太坊节点的执行客户端和共识客户端是两个独立的软件，但它们必须相互交互。为此，执行客户端和共识客户端都会使用一种“密码”来验证它们的连接。现在我们需要生成它：

```bash
$ ./prysm.sh beacon-chain generate-auth-secret

```

一个 `jwt.hex` 文件会出现。让我们把它移动到父文件夹：

```bash
$ mv jwt.hex ../jwt.hex

```

#### 运行节点 (Run the node)

现在你已经拥有了执行和共识客户端，并正确生成了 **JWT** 密钥，你可以启动客户端并运行一个以太坊全节点了。

**运行执行客户端**。首先，你需要运行执行客户端 **Geth**。导航回 `execution` 文件夹并运行以下命令：

```bash
$ cd .. # this command is used to go back in the ethereum-node1 folder
$ cd execution
$ ./go-ethereum/build/bin/geth --mainnet \
    --http \
    --http.api eth,net,engine,admin,web3 \
    --authrpc.jwtsecret=../jwt.hex

```

如果你看到以下的日志，说明一切运行正常。
```
INFO [06-08|17:56:38.738] Starting Geth on Ethereum mainnet...
INFO [06-08|17:56:38.738] Bumping default cache on mainnet         provided=1024 updated=4096
INFO [06-08|17:56:38.740] Maximum peer count                       ETH=50 total=50
INFO [06-08|17:56:38.745] Set global gas cap                       cap=50,000,000
INFO [06-08|17:56:38.752] Initializing the KZG library             backend=gokzg
INFO [06-08|17:56:38.771] Allocated trie memory caches             clean=614.00MiB dirty=1024.00MiB
INFO [06-08|17:56:38.772] Using pebble as the backing database…
```

**运行共识客户端**。现在你应该运行共识客户端 **Prysm**。不要关闭执行客户端所在的终端标签页。只需打开一个新的终端窗口或标签页并导航到 `consensus` 文件夹：

```bash
$ cd ethereum-node1
$ cd consensus
$ ./prysm.sh beacon-chain \
    --execution-endpoint=http://localhost:8551 \
    --mainnet \
    --jwt-secret=../jwt.hex \
    --checkpoint-sync-url=https://beaconstate.info \
    --genesis-beacon-api-url=https://beaconstate.info

```

系统可能会要求你接受 **Prysm** 的服务条款，输入 `accept` 即可。现在你就大功告成了！你应该会看到执行客户端和共识客户端开始在终端输出大量日志。这意味着你的以太坊全节点正在同步到链尖。
执行客户端
```
INFO [06-08|18:08:49.039] Forkchoice requested sync to new head    number=20,048,206 hash=8df21a..4afb49 finalized=unknown
INFO [06-08|18:08:52.507] Syncing beacon headers                   downloaded=322,560 left=19,725,577 eta=42m4.183s
INFO [06-08|18:08:57.515] Looking for peers                        peercount=1 tried=42 static=0
INFO [06-08|18:09:00.508] Syncing beacon headers                   downloaded=370,688 left=19,677,449 eta=43m35.827s
INFO [06-08|18:09:01.637] Forkchoice requested sync to new head    number=20,048,207 hash=d99dab..0293c9 finalized=unknown

```

共识客户端
```
[2024-06-08 18:09:24]  INFO blockchain: Called new payload with optimistic block payloadBlockHash=0xd44520a09a7a slot=9253245
[2024-06-08 18:09:24]  INFO blockchain: Called fork choice updated with optimistic block finalizedPayloadBlockHash=0x38916be8a559 headPayloadBlockHash=0xd44520a09a7a headSlot=9253245
[2024-06-08 18:09:24]  INFO blockchain: Synced new block block=0xec930e7c... epoch=289163finalizedEpoch=289161 finalizedRoot=0xb8065a78... slot=9253245
[2024-06-08 18:09:24]  INFO blockchain: Finished applying state transition attestations=123 payloadHash=0xd44520a09a7a slot=9253245 syncBitsCount=510 txCount=212
[2024-06-08 18:09:24]  INFO p2p: Peer summary activePeers=64 inbound=0 outbound=63
[2024-06-08 18:09:28]  INFO sync: Subscribed to topic=/eth2/6a95a1a9/beacon_attestation_35/ssz_snappy[2024-06-08 18:09:36]  INFO blockchain: Called new payload with optimistic block payloadBlockHash=0xff879102f29e slot=9253246
```

现在，你已经拥有了一个正在同步至链尖（tip of the chain）的以太坊全节点。请注意，同步过程可能会耗费大量时间（根据你的硬件性能和网络连接状况，可能需要数小时甚至数天）。

[!NOTE] 如果你想深入了解本示例中使用的具体命令和 CLI 标志（flags），Geth 和 Prysm 的官方文档是最佳查阅途径。

--- 
### Reth-Lighthouse 客户端组合

让我们尝试使用另外两种不同的客户端执行相同的操作：使用 **Reth** 作为执行客户端（execution client），使用 **Lighthouse** 作为共识客户端（consensus client）。

#### Reth

首先，你需要安装 **Reth**。进入 `ethereum-node2` 文件夹，然后进入 `execution` 文件夹。

**克隆存储库**。第一步是克隆 **Git** 存储库以获取源代码副本。回到你的家目录（home directory）并输入以下命令：

```bash
$ cd ethereum-node2
$ cd execution
$ git clone https://github.com/paradigmxyz/reth

```

太棒了！现在你已经拥有了 **Reth** 的本地副本，可以为你的平台编译可执行文件了。

**从源代码构建 Reth**。要构建 **Reth**，你需要运行以下命令：

```bash
$ cd reth
$ cargo install --locked --path bin/reth --bin reth

```

安装可能需要 10 分钟以上的时间。完成后，你可以通过运行以下命令来检查 **Reth** 是否已正确安装：

```bash
$ reth --version

```

你应该会看到类似以下的内容（版本可能会有所变动）：

```text
reth Version: 0.2.0-beta.6-dev
Commit SHA: ac29b4b73
Build Timestamp: 2024-04-22T17:29:01.000000000Z
Build Features: jemalloc
Build Profile: maxperf+

```

---

#### Lighthouse

现在你需要安装 **Lighthouse**，即共识客户端。返回 `ethereum-node2` 文件夹并进入 `consensus` 文件夹：

```bash
$ cd .. # 此命令用于返回 ethereum-node2 文件夹
$ cd consensus

```

你首先需要安装一些依赖项。如果你使用的是 **macOS**，需要运行：

```bash
$ brew install cmake

```

如果你使用其他操作系统，可以参考 **Lighthouse** 的官方文档。

**克隆存储库**。第一步是克隆 **Git** 存储库以获取源代码副本：

```bash
$ git clone https://github.com/sigp/lighthouse.git

```

**从源代码构建 Lighthouse**。要构建 **Lighthouse**，你需要运行以下命令：

```bash
$ cd lighthouse
$ git checkout stable
$ make

```

这可能需要超过 10 分钟才能完成。

---

#### 运行节点 (Run the node)

同样地，你需要先运行执行客户端 **Reth**。

**运行执行客户端**。导航回 `execution` 文件夹并运行此命令：

```bash
$ cd ../.. # 此命令用于返回 ethereum-node2 文件夹
$ cp ../ethereum-node1/jwt.hex ./jwt.hex # 我们使用之前生成的同一个 jwt.hex 文件
$ cd execution
$ reth node --full --http --http.api all --authrpc.jwtsecret=../jwt.hex

```

如果你看到如下内容，说明一切运行正常：

```text
2024-06-08T16:58:43.498297Z  INFO Starting reth version="0.2.0-beta.6-dev (ac29b4b73)"
2024-06-08T16:58:43.498434Z  INFO Opening database path="/Users/alessandromazza/Library/Application Support/reth/mainnet/db"
2024-06-08T16:58:43.514141Z  INFO Configuration loaded path="/Users/alessandromazza/Library/Application Support/reth/mainnet/reth.toml"
2024-06-08T16:58:43.514778Z  INFO Database opened
2024-06-08T16:58:43.514917Z  INFO Pre-merge hard forks (block based):…

```

**运行共识客户端**。现在你应该运行共识客户端 **Lighthouse**。不要关闭运行执行客户端的终端标签页。只需打开一个新的终端窗口或标签页并进入 `consensus` 文件夹：

```bash
$ cd ethereum-node2
$ cd consensus
$ lighthouse bn \
    --network mainnet \
    --checkpoint-sync-url https://mainnet.checkpoint.sigp.io \
    --execution-endpoint http://localhost:8551 \
    --execution-jwt ../jwt.hex \
    --http

```

大功告成！你应该会看到执行客户端和共识客户端开始在终端中输出大量日志数据。

**执行客户端日志：**

```text
2024-06-08T17:03:03.355648Z  INFO Received headers total=10000 from_block=18458372 to_block=18448373
2024-06-08T17:03:04.792262Z  INFO Received headers total=10000 from_block=18448372 to_block=18438373
2024-06-08T17:03:04.800043Z  INFO Received headers total=10000 from_block=18438372 to_block=18428373
2024-06-08T17:03:04.913377Z  INFO Received headers total=10000 from_block=18428372 to_block=18418373

```

**共识客户端日志：**

```text
Jun 08 17:03:24.929 INFO New block received        root: 0xa49c057026cea3190df38548d49963e271ebdc4d6f93d2301adc4034d6563113, slot: 9253515
Jun 08 17:03:29.001 WARN Head is optimistic        execution_block_hash: 0x5a14bfcb9e74c5b3a5121f99ef461ae066262200c269b5d11475274eb78aa7a5, info: chain not fully verified, block and attestation production disabled untilexecution engine syncs, service: slot_notifier
Jun 08 17:03:29.001 INFO Synced                    slot: 9253515, block: 0xa49c…3113, epoch: 289172, finalized_epoch: 289170, finalized_root: 0xca35…2b06, exec_hash: 0x5a14…a7a5 (unverified), peers: 31, service: slot_notifier

```

现在你已经拥有一个正在同步到链尖的以太坊全节点了。请注意，同步可能需要很长时间（取决于你的硬件和网络连接，可能需要数小时或数天）。

> [!NOTE]
> 如果你想详细了解本示例中使用的具体命令和 **CLI** 标志（flags），**Reth** 和 **Lighthouse** 的官方文档是最佳查阅地点。

下一节将解释以太坊区块链初始同步所面临的挑战。

> [!TIP]
> 觉得这些步骤太复杂且令人困惑？但你仍想为网络做贡献，并真正不依赖任何受信任的第三方，运行自己的以太坊全节点？
> 这里有一个完美的解决方案：**BuidlGuidl Client** 项目。他们创建了一个单行命令，让你能够一键运行以太坊节点。不相信？去亲自看看吧。
> 另一个选择是使用 **Dappnode**。你可以选择两种不同的方案：
> 1. 购买一个内置以太坊全节点的即插即用（plug-n-play）设备。
> 2. 安装 **Dappnode Core** 软件，它可以让你非常轻松地启动以太坊全节点。
> 
> 

---

> [!TIP]
>  **译者注**：
> 1. **Rust 技术栈的工程优势**：**Reth** 和 **Lighthouse** 均采用 **Rust** 编写。从后端性能角度看，相比于 **Go (Geth)**，**Reth** 的设计采用了“阶段性同步”（Staged Sync），能更有效地平衡磁盘 **I/O** 和 **CPU** 负载。对于追求极致同步速度和 **RPC** 响应时间的开发者，这是目前的首选组合。
> 2. **Checkpoint Sync（检查点同步）的魔法**：在运行 **Lighthouse** 命令时，`--checkpoint-sync-url` 是关键。它允许节点直接从一个受信任的状态快照开始，而不是从 2015 年的第一个区块开始重跑所有共识逻辑。这不仅极大地缩短了同步时间（从数天缩短至分钟级），还避免了早期历史数据可能带来的长程攻击风险。
> 
>
---
### 以太坊区块链的初始同步 (The First Synchronization of Ethereum-Based Blockchains)

通常在同步以太坊区块链时，你的客户端会下载并验证自最初开始（即创世区块）以来的每一个区块和每一笔交易。虽然可以通过这种方式完成完全同步，但同步过程会耗费极长时间，且对资源要求极高（如果存储速度不够快，它将需要更多的 **RAM** 且同步时间会变得异常漫长）。

以太坊及其多个分支链在 2016 年底曾遭受过 **DoS**（拒绝服务）攻击。受影响的区块链在进行完整同步（full sync）时往往进度缓慢。以以太坊主网为例，一个新的客户端在到达第 2,283,397 号区块之前进度飞快。该区块于 2016 年 9 月 18 日产出，标志着 **DoS** 攻击的开始。从该区块到第 2,700,031 号区块（2016 年 11 月 26 日）之间，交易的验证变得极其缓慢，且高度消耗内存和 **I/O**。在 2016 年当时的硬件环境下，这导致每个区块的验证时间超过了一分钟。以太坊随后通过一系列硬分叉升级解决了被 **DoS** 攻击利用的底层漏洞。这些升级还通过删除由垃圾交易创建的约 2000 万个空账户，对区块链进行了清理。

如果你正在进行全验证同步（full validation sync），你的客户端速度会减慢，验证受 **DoS** 影响的区块可能需要数天甚至更久。幸运的是，大多数以太坊客户端都包含“快速”同步选项，在同步到区块链尖端之前跳过交易的全量验证，只有在到达最新链尖后才恢复全量验证。对于执行客户端，开启快速同步的典型选项是 **Snap Sync**（快照同步）；对于共识客户端，快速同步的选项则是 **Checkpoint Sync**（检查点同步）。

在本教程中，我们默认在执行客户端上使用了 **Snap Sync**，在共识客户端上使用了 **Checkpoint Sync**。唯一例外的是 **Reth**，截至 2025 年 6 月，它尚未支持 **Snap Sync**。

---

> [!TIP]
> **译者注**：
> 1. **DoS 攻击的历史债务**：作为后端开发者，理解 2016 年的那段历史有助于你理解为什么以太坊的 `Gasper` 共识和 `EVM` 操作码（如 `EXTCODECOPY`）后来都重新调整了 **Gas** 消耗量。那次攻击本质上是攻击者通过构造极其复杂的指令，让节点在处理极低手续费交易时产生极大的计算开销，这种“非对称成本”是区块链后端设计必须规避的深坑。
> 2. **Snap Sync vs Checkpoint Sync 的技术本质**：
> * **Snap Sync**：不再按顺序下载每个区块的交易并回放，而是直接下载当前状态树（State Tree）的“快照”分片。这类似于数据库的“镜像恢复”而非“日志重放”。
> * **Checkpoint Sync**：利用信标链（Beacon Chain）的弱主观性（Weak Subjectivity），让共识客户端直接信任一个已达成的历史检查点，从而规避了从创世区块开始计算验证者的复杂逻辑。

---
### JSON-RPC 接口 (The JSON-RPC Interface)

以太坊客户端提供了一个 **API** 和一系列 **RPC**（远程过程调用）命令，这些命令采用 **JSON** 格式进行编码。你通常会看到它被称为 **JSON-RPC API**。从本质上讲，**JSON-RPC API** 是一个允许我们编写程序的接口，通过该接口，我们将以太坊客户端作为访问以太坊网络和区块链的网关。

通常，**RPC** 接口作为 **HTTP** 服务在 `8545` 端口上提供。出于安全考虑，默认情况下它被限制为仅接受来自 `localhost`（你计算机自己的 IP 地址，即 `127.0.0.1`）的连接。

要访问 **JSON-RPC API**，你可以使用专门的库（用你选择的编程语言编写），这些库提供了与每个可用 **RPC** 命令对应的“存根”（stub）函数调用；或者你也可以手动构造 **HTTP** 请求并发送/接收 **JSON** 编码的请求。你甚至可以使用像 `curl` 这样的通用命令行 **HTTP** 客户端来调用 **RPC** 接口。让我们尝试一下。首先，确保你的执行客户端（execution client）已配置并运行。然后，切换到一个新的终端窗口并输入以下命令：

```bash
$ curl -X POST -H "Content-Type: application/json" --data \
    '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":1}' \
    http://localhost:8545

{"jsonrpc":"2.0","id":1,"result":"Geth/1.14.3-stable/darwin-arm64/go1.22.2"}

```

在这个例子中，我们使用 `curl` 建立了一个指向地址 `http://localhost:8545` 的 **HTTP** 连接。我们已经在运行执行客户端，它在 `8545` 端口上将 **JSON-RPC API** 作为 **HTTP** 服务提供。我们指示 `curl` 使用 **HTTP POST** 方法，并将内容类型标识为 `application/json`。最后，我们将一个 **JSON** 编码的请求作为 **HTTP** 请求的数据部分传递。我们命令行的大部分内容只是在设置 `curl` 以正确建立 **HTTP** 连接。有趣的部分是我们发出的实际 **JSON-RPC** 命令：

`{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":1}`

**JSON-RPC** 请求根据 **JSON-RPC 2.0** 规范进行格式化。每个请求包含四个元素：

* `jsonrpc`：**JSON-RPC** 协议的版本。这必须精确地为 `"2.0"`。
* `method`：要调用的方法名称。
* `params`：一个结构化值，持有在调用方法期间使用的参数值。该成员可以省略。
* `id`：由客户端建立的标识符，如果包含，必须包含字符串、数字或 **NULL** 值。如果包含，服务器必须在响应对象中回复相同的值。该成员用于关联两个对象之间的上下文。

> [!TIP]
> `id` 参数主要用于在单个 **JSON-RPC** 调用中发出多个请求时，这种做法称为**批处理**（batching）。批处理用于避免为每个请求建立新的 **HTTP** 和 **TCP** 连接所带来的开销。例如，在以太坊语境下，如果我们想通过一个 **HTTP** 连接检索成千上万笔交易，我们就会使用批处理。在批处理时，你为每个请求设置不同的 `id`，然后将其与 **JSON-RPC** 服务器每个响应中的 `id` 进行匹配。实现这一点最简单的方法是维护一个计数器，并为每个请求增加其值。

我们收到的响应是：

`{"jsonrpc":"2.0","id":1,"result":"Geth/1.14.3-stable/darwin-arm64/go1.22.2"}`

这告诉我们，**JSON-RPC API** 正由 **Geth** 客户端版本 `1.14.3-stable` 提供服务。

让我们尝试一些更有趣的东西。在下一个例子中，我们向 **JSON-RPC API** 查询当前的 **Gas** 价格（单位为 **wei**）：

```bash
$ curl -X POST -H "Content-Type: application/json" --data \
    '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":4213}' \
    http://localhost:8545
    
{"jsonrpc":"2.0","id":4213,"result":"0x1B1717FC7"}

```

响应值 `0x1B1717FC7` 告诉我们，当前的 **Gas** 价格是 **7.27 gwei**（gigawei 或十亿 wei）。如果你和我们一样不习惯十六进制思维，你可以用一点 **Bash** 技巧在命令行中将其转换为十进制：

```bash
$ echo $((0x1B1717FC7))
7271972807

```

完整的 **JSON-RPC API** 可以在以太坊维基上进行研究。

> [!TIP]
> 在本节中，我们使用原始的 `curl` 请求来展示以太坊 **JSON-RPC** 接口。在现实生活中，你可能希望通过一种更好、更符合编程规范的方式来访问它。这就是“库”发挥作用的地方。你可以随意探索这三个最著名且使用最广泛的库：
> * **ethers.js** (JavaScript/TypeScript)
> * **web3.py** (Python)
> * **alloy** (Rust)
> 
> 

---

> [!TIP]
> **译者注**：
> 1. **无状态与安全加固**：作为后端开发者，必须注意默认的 `8545` 端口没有身份验证机制。在生产环境中，**严禁**直接将此端口暴露在公网。通常的做法是使用 **Nginx** 建立反向代理，并在代理层添加 **基本身份验证**（Basic Auth）或白名单 **IP** 过滤，或者通过 **SSH Tunnel** 进行远程连接。
> 2. **十六进制与大数处理**：以太坊返回的所有数值（如余额、**Gas** 价格）都是十六进制字符串（带有 `0x` 前缀）。在编写后端逻辑时，千万不要直接将其转换为标准的 64 位整数，因为以太坊的数值高达 256 位（`uint256`），这会超出大多数编程语言原生整数类型的表示范围。务必使用专门的“大数”（BigNumber/BigInt）库来处理这些结果。
> 
---
### 远程以太坊客户端 (Remote Ethereum Clients)

远程客户端仅提供完整客户端功能的一个子集。由于它们不存储完整的以太坊区块链数据，因此设置速度更快，且对数据存储空间的要求极低。

这类客户端通常提供以下一项或多项功能：

* 在钱包中管理私钥和以太坊地址。
* 创建、签名并广播交易。
* 使用数据负载（data payload）与智能合约交互。
* 浏览并与去中心化应用（DApps）交互。
* 提供指向区块浏览器等外部服务的链接。
* 进行以太币单位转换并从外部源获取汇率。
* 将 `Web3` 实例作为 JavaScript 对象注入 Web 浏览器。
* 使用由其他客户端提供或注入浏览器的 `Web3` 实例。
* 访问本地或远程以太坊节点上的 **RPC** 服务。

远程客户端通常通过连接到运行在其他地方的全节点（例如，你本地机器上运行的节点、Web 服务器上的节点，或第三方服务器上的节点）来提供功能，而无需同步本地的以太坊区块链副本。

以下是一些最流行的远程客户端及其提供的功能：

#### 移动端（智能手机）钱包 (Mobile Wallets)

大多数生产环境下的移动钱包都作为远程客户端运行，因为智能手机没有足够的资源来运行完整的以太坊客户端。轻客户端（Light clients）仍在开发中，尚未在以太坊中广泛使用。最著名的是 **Helios**，但它目前仍属于实验性软件。

流行的移动钱包包括（仅作为示例列出，不代表对其安全性或功能的认可）：

* **Coinbase Wallet**：支持多种不同链的移动钱包，如以太坊（及所有 **L2**）、**EVM** 兼容的 **L1**、比特币、**Solana**、莱特币和狗狗币。它还可以连接到 Coinbase 账户。
* **Phantom**：另一款多链钱包，兼容以太坊、**Solana**、比特币和 **Polygon**。
* **Trust Wallet**：支持一百多个区块链的移动多链钱包，适用于 iOS 和 Android。
* **Uniswap Wallet**：由 Uniswap 团队开发，仅支持以太坊及其兼容的 **L2** 和 **L1**。这是一款较新的钱包，支持 iOS 和 Android。

#### 浏览器钱包 (Browser Wallets)

许多钱包和 **DApp** 浏览器以 Chrome 或 Firefox 等浏览器的插件或扩展程序形式存在。这些是在浏览器内部运行的远程客户端。较流行的包括：

* **MetaMask**：在第 2 章中介绍过，是一款功能多样的浏览器钱包、**RPC** 客户端和基础合约浏览器。支持 Chrome、Firefox、Opera 和 Brave 浏览器。
* **Phantom**：也拥有 Web 浏览器钱包，界面设计非常整洁。
* **Rabby Wallet**：一款新型多链浏览器钱包，支持超过一百个不同的区块链（**EVM** 兼容链）。
* **Coinbase Wallet**：同样拥有浏览器版本，功能与移动端一致。

#### 硬件钱包 (Hardware Wallets)

大多数移动钱包和浏览器钱包都可以与安全性更高的硬件钱包配合使用。硬件钱包是旨在永不连接互联网的离线设备，专门用于抵御篡改和其他形式的物理攻击，从而提供更高级别的安全性。多家公司正在制造此类设备，其中使用最广泛的两家是 **Ledger** 和 **Trezor**。

---

> [!TIP]
> **译者注**：
> 1. **Web3 注入的原理**：从前端和后端交互的角度看，远程客户端（如 **MetaMask**）的核心功能是在浏览器环境中注入 `window.ethereum` 对象。这使得网页端的 **DApp** 能够通过标准的 **JSON-RPC** 协议请求用户对交易进行签名。后端开发者应注意，这种注入模式正逐渐向 **EIP-6963**（多钱包发现规范）演进，以解决多个钱包插件之间的冲突。
> 2. **硬件钱包的安全边界**：硬件钱包执行的是“盲签”或“解析签名”逻辑。其核心后端逻辑在于：私钥永远存储在受保护的**安全芯片（Secure Element）**中，外部程序只能将未签名的交易数据发送给硬件，由硬件内部完成签名并返回结果。这对于需要管理高价值资产的后端自动化脚本（如多签热钱包控制台）来说，是最终的安全屏障。
> 

---

### 总结 (Conclusion)

在本章中，我们深入探讨了以太坊客户端。你已经亲自下载、安装并同步了一个客户端，正式成为了以太坊网络的一员。通过在自己的计算机上备份区块链数据，你为整个系统的健康和稳定性做出了贡献。

由于以太坊周边的研发投入巨大，未来将会出现更多新型的以太坊客户端。目前最引人注目的研究领域包括：

* **历史数据裁剪 (History pruning)**：通过裁剪历史数据来降低全节点对存储空间的需求。
* **Verkle 树与无状态性 (Verkle trees and statelessness)**：使节点能够在不持有完整以太坊状态的情况下验证区块。
* **zk-EVM**：通过验证零知识证明（Zero-knowledge proof）来确认区块的正确性，而无需重新执行区块中的所有交易。

我们将在后续章节中逐一探索这些概念。但在此之前，我们需要先揭开让这一切成为可能的真正“魔法”：**密码学 (Cryptography)**。

---

> [!TIP]
> **译者注**：
> 1. **从“全员重跑”到“全员验证”**：正如我们在本章开头讨论的，当前以太坊最大的开销在于每个节点都要“重复执行”合约。而 **zk-EVM** 的引入将彻底改变这一范式——它将计算过程压缩为一个极小的数学证明。对于后端架构而言，这意味着节点的性能要求将大幅降低，因为“验证”证明的计算复杂度远低于“执行”原始逻辑。
> 2. **无状态性的工程影响**：**Verkle 树**是实现“无状态以太坊”的关键技术栈。作为后端开发者，一旦无状态性得以实现，我们甚至可以运行一个仅占用几百 MB 内存、几秒钟即可启动完成的“即时全节点”。这将极大地降低 DApp 的基础设施运维门槛，让“人人都能跑节点”真正从愿景变为现实。
> 
> 
---


[^1]: 共识客户端验证成功后会调用执行客户端，如果执行客户端返回无效，则区块为无效。
