# 第六章： 交易

交易是由外部账户（EOA）发起的签名消息，经由以太坊网络传输，并最终记录在以太坊区块链上。这个基本定义背后隐藏着许多令人着迷的细节。换个角度来看，交易是唯一能够触发状态变更或导致合约在以太坊虚拟机（EVM）中执行的行为。以太坊是一个全球唯一的单体状态机，而交易正是让这个状态机“跳动”并改变其状态的脉搏。合约不会自行运行，以太坊也不会自动运转——一切始于交易。

在本章中，我们将剖析交易，展示它们的工作原理并审视其细节。需要注意的是，本章的大部分内容是针对那些有兴趣在底层管理交易的人（例如正在开发钱包应用的开发者）。如果你习惯于使用现有的钱包应用，则无需担心这些细节，尽管你可能会发现这些底层机制非常有趣。

## 交易结构（The Structure of a Transaction）
首先，让我们看看交易在以太坊网络中进行序列化和传输时的基本结构。每个接收到序列化交易的客户端都会使用其内部的数据结构将其存储在内存中，并可能添加一些网络序列化交易本身并不存在的元数据。网络序列化（Network Serialization）是交易唯一的标准形式。在以太坊运行初期，只有单一类型的交易。但 EIP-2718 引入了一种处理不同交易类型的方法，并允许以不同的方式处理它们。具体而言，每笔交易都以一个指定交易类型的单字节（Byte）开头：
$$transaction = tx\_type \ || \ tx\_payload$$
在编写本书时（2025 年 6 月），共存在五种交易类型，如表 6-1 所示。

表 6-1. EIP-2718 交易类型
![Table 6-1](<./images/table 6-1.png>)

让我们详细分析这些交易组成部分。

### 遗留交易（Legacy Transactions）
一笔遗留交易[^1]是一个序列化的二进制消息，包含以下数据：
* **链 ID** (Chain ID)：你发送交易的目标网络标识。它是通过 EIP-155 引入的，作为一种简单的重放攻击保护机制。
* **Nonce**：由发起账户（EOA）发出的序列号，用于防止消息重放。[^2]
* **Gas价格** (Gas price)：发起者愿意支付的单位 Gas 单价（以 wei 为单位）。
* **Gas限制** (Gas limit)：发起者愿意为这笔交易购买的最大 Gas 数量。请注意，你只需为实际消耗的 Gas 付费，Gas 限制仅代表你的出价上限。
* **接收者** (Recipient)：目标以太坊地址。
* **数额** (Value)：发送到目的地的以太币数量。
* **数据** (Data)：可变长度的二进制数据负载（Payload）。
* **v, r, s**：发起账户（EOA）的 ECDSA 数字签名的三个组成部分。

交易消息的结构使用**递归长度前缀**（RLP）编码方案进行序列化。RLP 是专为以太坊设计的，旨在实现简单且字节精准的数据序列化。以太坊中的所有数字都编码为大端（big-endian）整数，长度为 8 位（1 字节）的倍数。

请注意，为了清晰起见，这里显示了字段标签（如 to、gas limit 等），但它们并不属于交易序列化数据的一部分。序列化数据仅包含 RLP 编码后的字段值。通常，RLP 不包含任何字段分隔符或标签。它使用长度前缀来识别每个字段的长度，超出定义长度的内容即属于结构中的下一个字段。

虽然这是实际传输的交易结构，但大多数内部表示和用户界面可视化都会添加一些从交易或区块链中派生出的额外信息。关于“From”地址：你可能会注意到地址数据中没有“from”字段。这是因为发起账户（EOA）的公钥可以从 ECDSA 签名的 $v, r, s$ 分量中派生出来。随后，地址又可以从公钥中派生。当你看到交易显示 from 字段时，那是可视化软件自行添加的。其他元数据：客户端软件经常添加的其他元数据包括块号（一旦交易发布并被包含在区块链中）和交易 ID（计算得出的哈希值）。同样，这些数据是从交易中派生出来的，并不构成交易消息本身的一部分。

### EIP-2930 交易
[EIP-2930](https://oreil.ly/1dSza) 交易是首个使用 EIP-2718 类型化交易信封（Typed Transaction Envelope）的交易，其交易类型标识为 0x01。它们在本质上与之前的交易类型相同，但增加了一个名为 访问列表（Access List） 的新字段。这是一个由（地址、存储槽）组成的数组，允许用户为交易将要触达的地址和存储槽进行“预付费”。通过这种方式，在以太坊虚拟机（EVM）的执行过程中，用户被收取的 Gas 费用会更低。

>[!NOTE]
>更准确地说，访问列表中包含的地址及其存储槽分别被计入 accessed_addresses（已访问地址）和 accessed_storage_keys（已访问存储键）中。EVM 利用这些列表来区分热访问（Warm Access）和冷访问（Cold Access）。冷访问收取的 Gas 远高于热访问。例如，SLOAD 操作码在访问热存储槽时仅收取 100 Gas，而冷访问则需收取 2,100 Gas。

这种新交易类型的引入主要是为了解决由 EIP-2929 引起的问题。EIP-2929 提高了状态访问操作码的 Gas 成本，导致一些智能合约在处理交易时因 Gas 不足（Out-of-Gas）错误而失败。通过引入访问列表，用户可以为其交易将访问的地址和存储槽预先付费，从而防止这些执行失败。

### EIP-1559 交易
[EIP-1559](https://oreil.ly/ZBON6) 交易于 2021 年 8 月 5 日在“伦敦”硬分叉中引入，其交易类型标识为 0x02。它们通过引入一个新的协议参数——基础费（Base Fee），彻底改变了以太坊的手续费市场结构。

基础费代表了在以太坊网络上发送交易所需支付的最低费用。在本次升级中，区块 Gas 限制（Block Gas Limit）从 1,500 万翻倍至 3,000 万 Gas，并引入了等于 Gas 限制一半的区块 Gas 目标（Block Gas Target），即 1,500 万 Gas。其核心设计理念是保持以太坊网络平均负载与以往持平，但在需要时允许区块容量变得更大（最高可达两倍）。

为了使区块的平均 Gas 使用量维持在 1,500 万，基础费并不是一个固定值：它会根据区块的利用率动态调整。如果一个区块的实际 Gas 使用量高于目标值，基础费就会增加；如果低于目标值，基础费则会减少。

区块 Gas 限制（几乎）总是会在特定区块被调整为固定的、圆整的数值：1,000 万、1,250 万、1,500 万以及 3,000 万，具体趋势如图 6-1 所示。事实上，尽管验证者（以及旧 PoW 共识协议下的矿工）可以在每个区块微调 Gas 目标（这会直接转化为 Gas 限制），但区块 Gas 限制是一个至关重要的数值，通常所有人都会遵循核心开发者的建议。
![Figure 6-1](<./images/figure 6-1.png>)
图 6-1. 以太坊区块 Gas 限制历史演进
基础费（Base Fee）并不会流向创建区块的验证者（或矿工）；相反，它们会被立即销毁（Burned），从而减少以太坊的总供应量。同时，协议引入了一项新费用——优先费（Priority Fee），你可以将其理解为支付给验证者（或矿工）的小费，用以激励他们将你的交易包含在下一个区块中。
>[!TIP]
>理论上，你可以创建仅支付强制性基础费而“零小费”的交易。协议并不强求你向验证者支付小费。但在现实操作中，你应当始终包含优先费，以确保交易能在合理的时间内得到确认。需要注意的是，钱包通常会自动为你处理基础费和优先费，并将其设置为合适的数值。

一笔 EIP-1559 交易是一个序列化的二进制消息，包含以下数据：

* **链 ID** (Chain ID)：与遗留交易相同。
* **Nonce**：与遗留交易相同。
* **每单位 Gas 的最大优先费** (Max priority fee per gas)：发起者愿意直接支付给验证者的 Gas 单价（以 wei 为单位），作为将其交易纳入区块的小费。
* **每单位 Gas 的最大总费用** (Max fee per gas)：发起者愿意支付的总 Gas 单价（以 wei 为单位），包含基础费和优先费。
* **Gas 限制** (Gas limit)：与遗留交易相同。
* **接收者** (Recipient)：与遗留交易相同。
* **访问列表** (Access list)：与 EIP-2930 交易相同。
* **数额** (Value)：与遗留交易相同。
* **数据** (Data)：与遗留交易相同。
* **v, r, s**：与遗留交易相同。

与所有交易类型一样，其消息结构采用 RLP 编码方案进行序列化。

### EIP-4844 交易
[EIP-4844](https://oreil.ly/JMJmB) 交易随 2024 年 3 月 13 日的“坎昆（Cancun）”硬分叉引入，交易类型标识为 0x03。我们在第四章“KZG 承诺”一节中已经提到过它们，并将在第十六章进行深入探讨。这类交易也被称为携带 Blob 的交易（blob-carrying transactions），因为它们带有一个“边车（sidecar）”——即 Blob。每个 Blob 包含大量数据（约 131,000 字节），这些数据无法被 EVM（以太坊虚拟机）直接访问，但其对应的“承诺（commitment）”是可以被访问的。

为了处理 Blob，协议引入了一种新型 Gas——Blob Gas。它与普通 Gas 完全分离且相互独立。尽管它深受 EIP-1559 的启发并遵循类似的定价规则，但它有自己的目标调节逻辑：如果区块中使用的 Blob Gas 超过了目标值，Blob Gas 的价格就会上涨；反之则下降。

其序列化二进制消息格式与 EIP-1559 类似，但新增了两个字段：

* **每单位 Blob Gas 的最大费用** (Max fee per blob gas)：发起者愿意为 Blob 支付的 Blob Gas 单价（以 wei 为单位）。
* **Blob 版本化哈希列表** (Blob versioned hashes)：一个 32 字节值的列表，代表与该交易所携带的每个 Blob 相关联的 KZG 承诺的版本化哈希。

>[!NOTE]
>随着坎昆硬分叉和 EIP-4844 交易的引入，区块头增加了两个新元素：
>
> **已使用的 Blob Gas** (Blob gas used)：
>
>区块内所有 EIP-4844 交易消耗的 Blob Gas 总量。
>
> **超额 Blob Gas** (Excess blob gas)：
>
>在该区块之前，累积消耗的超出目标值的 Blob Gas 总量。

### EIP-7702 交易
[EIP-7702](https://oreil.ly/W_28X) 交易随 2025 年 5 月 7 日的“Pectra”硬分叉引入，交易类型标识为 0x04。它们允许外部账户（EOA）在自己的账户中设置代码。传统上，EOA 的代码字段是空的；它们只能发起交易，除非与智能合约交互，否则无法执行复杂操作。EIP-7702 改变了这一现状，使 EOA 能够执行如下操作：

* **批量处理**（Batching）：允许同一用户在单笔原子交易中完成多个操作。例如，先进行 ERC-20 授权（Approve），紧接着完成资产消费。这是许多去中心化交易所中非常常见的流程。
* **代付**（Sponsorship）：账户 X 可以代表账户 Y 支付交易费用。
* **权限降级**（Privilege deescalation）：用户可以签署子密钥并赋予其特定的受限权限，这些权限远弱于对账户的全局访问权。例如，限制每天只能消费总余额的 1%，或者只能与特定的应用程序交互。

底层的实现细节相当复杂，如果你感兴趣，我们建议阅读 EIP 官方网站。不过，从高层概述来看，它的原理简单却非常强大。EIP-7702 允许 EOA 为自己分配一个委派标识符（delegation designator）。该标识符指向一个（已存在于以太坊主网上的）智能合约。当一笔交易发送至该 EOA 时，它会执行指定地址的代码，就好像那是该 EOA 自身的实际代码一样，如图 6-2 所示。
![Figure 6-2](<./images/figure 6-2.png>)
图 6-2. EIP-7702 委派机制


## 交易 Nonce（The Transaction Nonce）
Nonce 是交易中最重要但也最常被误解的组成部分之一。在以太坊“黄皮书”中，它的定义如下：

> Nonce：一个标量值，等于从该地址发送出的交易数量；或者对于有关联代码的账户（合约账户）而言，是指由该账户创建的合约数量。

严格来说，Nonce 是发起地址的一个属性——即它仅在发送地址的语境下才有意义。然而，Nonce 并不作为账户状态的一部分显式地存储在区块链上，而是通过计算从该地址发出的已确认交易数量动态得出的。

Nonce 的存在对于两个场景至关重要：一是确保交易按创建顺序被包含的可用性功能，二是防止交易重复攻击的核心安全功能。让我们分别为这两个功能看一个示例场景：

场景1：顺序执行

假设你想进行两笔交易：一笔是 6 ETH 的重要付款，另一笔是 8 ETH 的付款。你先签署并广播了 6 ETH 的交易，因为它更重要。随后你签署并广播了 8 ETH 的交易。遗憾的是，你忽略了账户里只有 10 ETH，因此网络无法同时接受这两笔交易：其中一笔必然失败。

因为你先发送了更重要的 6 ETH 交易，你自然希望这笔成功，而 8 ETH 的被拒绝。但在像以太坊这样的去中心化系统中，节点接收交易的顺序是随机的，无法保证某个节点一定会先收到哪一笔。如果没有 Nonce，哪一笔被接受将变成一场随机的博弈。

但是，有了 Nonce 之后，你发送的第一笔交易 Nonce 为 3（假设），而 8 ETH 交易的 Nonce 就是 4。即便节点先收到了 Nonce 为 4 的交易，它也会将其忽略，直到 Nonce 0 到 3 的交易全部处理完毕。

场景 2：重放保护

假设你的账户里有 100 ETH。你从网上买了一个心仪的零件并支付了 2 ETH。你签署并广播了这笔交易。如果没有 Nonce，当你第二次给同一个地址发送 2 ETH 时，这笔交易看起来和第一笔完全一样。

这意味着任何在网络上看到你交易的人（包括收款人或你的对手），只需简单地复制并粘贴你原始交易的签名数据，就可以一遍又一遍地“重放”这笔交易，直到榨干你账户里所有的钱。然而，由于交易数据中包含了 Nonce 且每笔交易递增，每一笔交易都是唯一的。即使是向同一地址发送相同金额，其 Nonce 也不同，这让任何人都不可能“复制”你的支付行为。

总结，需要特别指出的是，对于以太坊这种基于账户模型的协议，使用 Nonce 是至关重要的；这与比特币协议所使用的 UTXO（未花费交易输出） 机制形成了鲜明对比。

### 追踪 Nonce（Keeping Track of Nonces）
在本节及后续章节中，我们将使用 Foundry 工具套件——特别是 cast 工具。它能以非常便捷的方式与区块链进行交互。如果你想复现以下示例，请务必安装它。

首先，我们需要设置本章将要使用的钱包。打开终端窗口并输入：

```Bash
$ cast wallet new
Successfully created new keypair.
Address:     0x7e41354AfE84800680ceB104c5Fc99eCB98A25f0
Private key: 0xd6d2672c6b4489e6bcd4e93b9af620fa0204b639b7d7f93765479c0846be0b58
```
> [!CAUTION]
> 如果你向上述地址发送资金，那是在浪费钱，因为其私钥已公开，任何人都可以利用它将资金转走。

现在，我们需要将私钥导入电脑的密钥库（Keystore），以便稍后轻松调用：

```Bash
$ cast wallet import example \
    --private-key 0xd6d2672c6b4489e6bcd4e93b9af620fa0204b639b7d7f93765479c0846be0b58
Enter password:
`example` keystore was saved successfully. Address: 0x7e41354afe84800680ceb104c5fc99ecb98a25f0
```
你可以选择（并推荐）设置一个密码，在后续使用该账户创建交易时将需要输入此密码。现在我们已正确设置，但账户里还没有任何 ETH。

你可以随时检查余额。首先，获取与账户关联的地址：
```Bash
$ cast wallet address --account example
Enter keystore password:
0x7e41354AfE84800680ceB104c5Fc99eCB98A25f0
```
接着，你可以向区块链查询余额。在本章的所有示例中，我们将使用以太坊 Sepolia 测试网：
```Bash
$ cast balance 0x7e41354AfE84800680ceB104c5Fc99eCB98A25f0 --rpc-url https://ethereum-sepolia-rpc.publicnode.com
0
```
> [!NOTE]
> 请留意上一个 cast 命令中的 --rpc-url 标志。它应该指向你感兴趣的区块链的 RPC 端点。可靠的 RPC 端点通常需要付费，但如果你只是想进>行实验（如本章所述），有很多免费选项，例如：
>
> * Public Node
> * LlamaNodes
> * ChainList

为了获取一些免费的 Sepolia ETH 代币，你可以使用在线水龙头（Faucet）。我们将使用 Google Cloud Web3 水龙头（如图 6-3 所示），它会提供 0.05 ETH。访问 Ethereum Sepolia Faucet，粘贴你的地址并点击 "Receive 0.05 Sepolia ETH" 按钮。你应该很快就能收到 0.05 ETH。
![Figure 6-3](<./images/figure 6-3.png>)
图 6-3. Google Cloud Web3 水龙头界面

在成功申领测试币后，你可以验证余额已从 0 变为 $0.05 \text{ ETH}$（即 $5 \times 10^{16} \text{ Wei}$ ）：
```Bash
$ cast balance 0x7e41354AfE84800680ceB104c5Fc99eCB98A25f0 --rpc-url https://ethereum-sepolia-rpc.publicnode.com
50000000000000000
```
太好了！现在我们已完全准备就绪，可以回到关于交易 Nonce 的实验中。

从实际操作的角度来看，Nonce 是指从某个账户发出的**已确认**（即已上链）交易的最新计数。要查询 Nonce，你可以使用 cast 询问区块链。只需打开一个新的终端窗口并输入：
```Bash
$ cast nonce 0x7e41354AfE84800680ceB104c5Fc99eCB98A25f0 --rpc-url https://ethereum-sepolia-rpc.publicnode.com
0
```

> [!TIP]
> Nonce 是一个从零开始的计数器，这意味着第一笔交易的 Nonce 为 0。在本例中，由于尚未发送过交易，返回值为 0。此外，RPC 响应总是指向下一个可用的 Nonce。例如，如果地址已发送 10 笔交易（Nonce 0-9），查询将返回 10。

我们可以实际动手尝试发送一笔以太币（ETH）了。我们将向 `vitalik.eth` 发送 0.001 ETH，这是以太坊联合创始人 Vitalik Buterin 的 ENS（以太坊域名服务）地址：
```Bash
$ cast send --account example vitalik.eth --value 0.001ether --rpc-url https://ethereum-sepolia-rpc.publicnode.com
blockHash               0xa1171309fd406e44e86be9695a597d2bf5c728738d140b9958cfb50276c32b1b
blockNumber             6989355
contractAddress
cumulativeGasUsed       18009816
effectiveGasPrice       11163498011
from                    0x7e41354AfE84800680ceB104c5Fc99eCB98A25f0
gasUsed                 21000
logs                    []
logsBloom               0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
root
status                  1 (success)
transactionHash         0xeb7bb0322858a4e1ed85271a60d2f8353075dc0bcd0c80448ee1d5ca0bb85def
transactionIndex        60
type                    2
blobGasPrice
blobGasUsed
authorizationList
to                      0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045
```

你的钱包会为你管理的每个地址自动追踪 Nonce。只要你仅从单一终端（例如只用这台电脑上的 cast）发起交易，这个过程非常简单。但假设你正在编写自己的钱包软件或某些需要发起交易的应用程序，你该如何管理 Nonce？

当你创建一笔新交易时，你会按顺序分配下一个 Nonce。但在该交易被确认（即正式入块）之前，它并不会计入链上的 Nonce 总数。

为了观察这一现象，我们可以尝试快速连续执行以下命令：
```Bash
$ cast nonce 0x7e41354AfE84800680ceB104c5Fc99eCB98A25f0 --rpc-url https://ethereum-sepolia-rpc.publicnode.com
10
$ cast send --account example vitalik.eth --value 0.001ether --async --rpc-url https://ethereum-sepolia-rpc.publicnode.com
0x85f5b0db44407a6e9252590dc809087a2e232e00a951c9cb8853a109da5ddad4
$ cast nonce 0x7e41354AfE84800680ceB104c5Fc99eCB98A25f0 --rpc-url https://ethereum-sepolia-rpc.publicnode.com
10
$ cast nonce 0x7e41354AfE84800680ceB104c5Fc99eCB98A25f0 --rpc-url https://ethereum-sepolia-rpc.publicnode.com
11
```
如你所见，我们发送的交易并没有立即增加 Nonce 计数；即使在发送交易之后，它依然保持为 10。如果我们等待几秒钟，让网络通信稳定下来并将交易包含在区块中，再次调用 Nonce 查询才会返回预期的数字 11。

> [!NOTE]
> 请留意 `cast send` 命令中使用的 `--async` 标志：如果你不使用它，`cast` 会在终端中持续阻塞，直到交易在区块内得到确认。使用该标志后，它会将交易推送到网络并立即返回交易哈希（Transaction Hash），而无需等待入块。

现在让我们看一个不同的例子：
```Bash
$ cast nonce 0x7e41354AfE84800680ceB104c5Fc99eCB98A25f0 --rpc-url https://ethereum-sepolia-rpc.publicnode.com
11
$ cast send --account example vitalik.eth --value 0.001ether --async --rpc-url https://ethereum-sepolia-rpc.publicnode.com
0x63188aa73247ffe06388a9adf399fa715e42fbc37ca53f77642a7860c80feb9d
$ cast nonce 0x7e41354AfE84800680ceB104c5Fc99eCB98A25f0 --rpc-url https://ethereum-sepolia-rpc.publicnode.com
11
$ cast rpc eth_getTransactionCount 0x7e41354AfE84800680ceB104c5Fc99eCB98A25f0 pending --rpc-url https://ethereum-sepolia-rpc.publicnode.com
"0xc"
$ cast nonce 0x7e41354AfE84800680ceB104c5Fc99eCB98A25f0 --rpc-url https://ethereum-sepolia-rpc.publicnode.com
12
```

在发送交易之前，我们的 Nonce 计数是 11；发送交易后，我们立即查询新的 Nonce。正如从前一个示例中所预期的，由于交易仍处于内存池（Mempool）中待处理，尚未被包含在区块内，因此 Nonce 尚未更新。然而，我们使用了一种新的查询方式，它实际上能够获取到真实的 Nonce 数量，即便交易尚未得到确认（返回的 0xc 是十六进制的 12）。几秒钟后，交易被添加到区块中，此时调用 cast nonce 就会返回新的正确值。

`cast nonce` 与 `eth_getTransactionCount pending` 之间的区别很简单：前者仅考虑已确认的交易（即已入块的交易），而后者则尝试包含那些仍在内存池中待处理（Pending）的交易。

> [!Warning]
> 在使用 eth_getTransactionCount pending 来统计待处理交易时务必小心。事实上，尽管它试图返回一个地址真实的 Nonce 值，但没有任何办法能百分之百确定内存池中是否还隐藏着其他等待确认的待处理交易。
>
> 公共内存池（Public Mempool）并不是一个全球统一的实体。 每个节点都有自己的内存池：这是一种用于存放待处理交易的动态仓库，在它们被正式确认上链前进行临时保存。每个节点都可以通过设置不同的规则（例如不同的最低 Gas 价格限制或账户限制）来定制自己接受或拒绝新交易的逻辑。虽然 RPC 服务商拥有庞大的节点网络，并且理论上应该对所有待处理交易拥有（近乎）完整的视角，但你仍应警惕，不要将该返回值视为 100% 准确。

### Nonce 间隙、重复 Nonce 与确认（Gaps, Duplicates, and Confirmation）
如果你是通过编程方式创建交易，追踪 nonce 就变得至关重要，特别是当你试图从多个独立的进程同时发起交易时。

以太坊网络根据 nonce 严格顺序地处理交易。这意味着，如果你发送了一笔 nonce 为 0 的交易，接着又发送了一笔 nonce 为 2 的交易，那么第二笔交易不会被包含在任何区块中。它会被存储在内存池（mempool）中，此时以太坊网络会等待缺失的 nonce 出现。所有节点都会假设缺失的 nonce 只是被延迟了，而 nonce 为 2 的交易只是乱序到达了。

如果你随后发送了缺失的 nonce 1，那么两笔交易（nonce 1 和 2）都将被处理并包含在区块中（当然前提是它们合法）。一旦你填补了间隙，网络就能打包之前保存在内存池中的乱序交易。

这意味着，如果你按顺序创建了几笔交易，而其中一笔没有被正式包含在区块中，那么所有后续交易都会被“卡住”，等待那个缺失的 nonce。一笔交易可能因为无效或 gas 不足而产生无意的“间隙”。要让交易重新流动，你必须发送一笔具有缺失 nonce 的有效交易。你同样需要警惕：一旦这个“缺失”的 nonce 被网络验证，所有已广播的后续 nonce 交易都会依次生效——撤回一笔已发送的交易是不可能的！

另一方面，如果你不小心重复了 nonce —— 例如发送了两笔具有相同 nonce 但收款人或金额不同的交易 —— 那么其中一笔会被确认，另一笔会被拒绝。哪一笔被确认取决于它们到达第一个接收它们的验证节点的顺序 —— 也就是说，这具有相当大的随机性。

如你所见，追踪 nonce 是必不可少的。如果你的应用程序无法正确管理这一过程，就会遇到麻烦。遗憾的是，正如我们将在下一节看到的，如果你尝试并发执行此操作，情况会变得更加复杂。

### 并发、交易发起与 Nonce（Concurrency, Transaction Origination, and Nonces）
并发是计算机科学中一个复杂的方面，它有时会出人意料地出现，尤其是在像以太坊这样去中心化和分布式的实时系统中。

简单来说，并发是指多个独立系统同时进行计算。这些系统可以在同一个程序中（如多线程）、同一个 CPU 上（如多进程），或者在不同的计算机上（如分布式系统）。从定义上讲，以太坊是一个允许操作（节点、客户端、DApps）并发的系统，但通过共识机制强制执行单体状态（Singleton State）。

现在，想象一下你有多个独立的钱包应用程序，它们正在从同一个或多个地址生成交易。这种情况的一个例子是交易所处理来自其热钱包（hot wallet）的提现。理想情况下，你会希望有多台计算机处理提现，这样它就不会成为瓶颈或单点故障。然而，这很快就会变得棘手，因为多台计算机产生提现会导致一些棘手的并发问题，其中最核心的就是 nonce 的选择。多台从同一个热钱包账户生成、签名并广播交易的计算机如何进行协调？

你可以使用一台计算机按先来后到的顺序，为负责签名交易的计算机分配 nonce。然而，这台计算机现在成了单点故障（single point of failure）。更糟的是，如果分配了几个 nonce，但其中一个由于处理该交易的计算机发生故障而从未被使用，那么所有后续交易都会被卡住。

另一种方法是生成交易但不分配 nonce（因此不签名——请记住，nonce 是交易数据的组成部分，因此必须包含在验证交易的数字签名中）。然后，你可以将它们排队发送到一个负责签名并追踪 nonce 的单一节点。但同样，这会成为流程中的一个瓶颈：在高负载下，签名和追踪 nonce 的部分最容易发生拥塞，而生成未签名交易的部分其实并不真正需要并行化。你虽然实现了一定程度的并发，但在流程的关键部分却缺乏并发能力。

最终，这些并发问题，加上在独立进程中追踪账户余额和交易确认的难度，迫使大多数实现方案趋向于规避并发并创造瓶颈，例如使用单一进程处理交易所的所有提现交易，或者设置多个完全独立工作的热钱包进行提现，仅需定期进行余额重平衡（rebalanced）。

## 交易 Gas（Transaction Gas）
我们在前面的章节中简要提到过 Gas，并将在第 14 章对其进行更详细的讨论。不过，让我们先了解一下交易中 gasPrice 和 gasLimit 组件所起作用的基础知识。

Gas 是以太坊的燃料。Gas 并不是以太币（Ether）：它是一种独立的虚拟货币，拥有自己对以太币的汇率。以太坊使用 Gas 来控制交易可以消耗的资源量，因为交易将在全球数千台计算机上处理。这种开放式（图灵完备）的计算模型需要某种形式的计量，以避免拒绝服务（DoS）攻击或无意间消耗大量资源的交易。

Gas 与以太币分离是为了保护系统免受以太币价值剧烈波动的影响，并以此管理 Gas 所支付的各种资源（计算、内存和存储）成本之间重要且敏感的比例关系。

交易中的 gasPrice 字段允许交易发起者设置他们愿意支付的 Gas 单价。该价格以每单位 Gas 多少 Wei 来衡量。

> [!TIP]
> 知名网站 Etherscan 提供了以太坊主网当前 Gas 价格及其他相关 Gas 指标的实时信息。

钱包可以在其发起的交易中调整 gasPrice，以实现更快的交易确认。gasPrice 设置得越高，交易被确认的速度通常就越快。相反，低优先级的交易可以携带较低的价格，从而导致确认速度变慢。gasPrice 可设置的最小值等于交易被包含区块的基础费用（Base Fee）（我们在 EIP-1559 交易中介绍过这一概念）。

> [!NOTE]
> 在伦敦硬分叉（London Hard Fork）和 EIP-1559 协议实施之前，最低可接受的 gasPrice 是零。这意味着钱包可以生成完全免费的交易。根据当时网络的处理能力，这些交易可能永远不会被确认，但协议本身并不禁止免费交易。在以太坊运行的前几个月里，你可以从区块链上找到好几个成功入块的零费用交易示例。

### EIP-1559：基础费用与优先费用（Base Fee and Priority Fee）
正如我们之前简要解释过的，EIP-1559 通过引入一个新的协议参数——基础费用（Base Fee），彻底改变了以太坊费用市场的结构。它代表了一笔交易若要被视为有效并被包含在区块中，所需支付的最低 Gas 价格。

基础费用与交易实际支付的 Gas 费用之间的差额被称为优先费用（Priority Fee）。这部分费用直接流向创建该交易所属区块的验证者（Validator）。

### 如何确定“正确”的 Gas 价格（How to Know the "Correct" Gas Price）

Cast 可以通过计算多个区块的平均价格来提供 Gas 价格建议：
```Bash
$ cast gas-price --rpc-url https://ethereum-sepolia-rpc.publicnode.com
4845187414
```
与 Gas 相关的第二个重要字段是 gasLimit。简单来说，gasLimit 指的是交易发起者为了完成交易而愿意购买的 Gas 单位的最大数量。对于简单支付（即从一个外部账户 EOA 向另一个 EOA 转账以太币），所需的 Gas 量固定为 21,000 个单位。要计算这笔交易的花费，只需将 21,000 乘以你愿意支付的 gasPrice（如果是 EIP-1559 交易，则乘以 maxFeePerGas）。

如果你的交易目标地址是一个合约，那么所需的 Gas 量可以被估算，但无法被精确确定。这是因为合约可以评估不同的条件，从而导致不同的执行路径，进而产生不同的总 Gas 成本。合约可能只执行简单的计算，也可能执行更复杂的计算，这取决于你无法控制且无法预测的条件。为了证明这一点，让我们看一个例子：我们可以编写一个智能合约，每次被调用时都会递增一个计数器，并执行一个循环，循环次数等于当前的调用次数。也许在第 100 次调用时，它会像抽奖一样发放一份特别奖品，但它需要进行额外的计算来算出这份奖品。如果你调用合约 99 次，会发生一种情况，但在第 100 次调用时，情况会大不相同。你为此支付的 Gas 量取决于在你的交易被包含进区块之前，有多少其他交易调用了该函数。也许你的估算是基于自己是第 99 次交易，但就在你的交易被确认之前，其他人抢先完成了第 99 次调用。现在你成了第 100 次调用的那个人，计算量（以及 Gas 成本）就会高得多。

借用以太坊中常用的一个类比，你可以将 gasLimit 想象成汽车（交易）中油箱的容量。你在油箱中加入你认为旅程（验证交易所需的计算）所需的油量。你可以一定程度上估算这个量，但旅途中可能会出现意外变化，比如绕路（更复杂的执行路径），从而增加了燃料消耗。

然而，油箱的类比在某种程度上是有误导性的。它实际上更像是加油站公司的信用账户，你在行程结束后根据实际消耗的油量进行结算。当你传输交易时，首要的验证步骤之一是检查发起账户是否有足够的以太币来支付 $maxFeePerGas \times gasLimit$（或旧版交易的 $gasPrice \times gasLimit$）的费用。但这笔金额直到交易执行完毕才真正从你的账户中扣除。你只需为交易实际消耗的 Gas 付费，但在发送交易之前，你必须拥有足以支付你所设定的最大金额的余额。

## 交易接收者（Transaction Recipient）
交易的接收者在 to 字段中指定。该字段包含一个 20 字节的以太坊地址。该地址可以是一个外部账户（EOA）或一个合约地址。

以太坊不对该字段进行进一步的验证。任何 20 字节的值都被认为是有效的。如果该 20 字节的值对应的地址没有相应的私钥，或者没有相应的合约，交易依然有效。以太坊无法得知一个地址是否是从现有的公钥（进而从私钥）中正确推导出来的。

> [!WARNING]
> 以太坊协议不会验证交易中的接收地址。你可以将资金发送到一个没有对应私钥或合约的地址，从而“销毁”（burning）这些以太币，使其永远无法被花费。验证工作应当在用户界面（UI）层级完成。

将交易发送到错误的地址很可能会销毁所发送的以太币，使其永远无法访问（无法花费），因为大多数地址都没有已知的私钥，因此无法生成签名来花费它。通常认为地址的验证发生在用户界面层级（参见“ERC-55：带校验和的十六进制编码”）。事实上，销毁以太币也有一些正当理由——例如，作为支付通道或其他智能合约中作弊行为的惩罚手段。由于以太币的总量是有限的，销毁以太币实际上将销毁的价值分配给了所有以太币持有者（与他们持有的以太币数量成正比）。

## 交易金额与数据（Transaction Value and Data）
交易的核心“载荷”包含在两个字段中：value（金额）和 data（数据）。交易可以同时包含金额和数据，也可以只包含其中之一，或者两者都不包含。这四种组合都是有效的。

仅包含金额：这是一笔支付（Payment）。仅包含数据：这是一次调用（Invocation）。同时包含金额和数据：既是支付也是调用。既无金额也无数据：这可能只是在浪费 Gas！但协议层面依然允许。

让我们尝试所有这些组合。我们将像之前一样，使用 cast 在 Sepolia 测试网上发送交易。

我们的第一笔交易只包含金额（支付），没有数据载荷：

```Bash
$ cast send --account example vitalik.eth --value 0.001ether --rpc-url https://ethereum-sepolia-rpc.publicnode.com
```
在图 6-4 中，你可以看到发送的金额为 0.001 ether，而数据载荷（在 Etherscan 上显示为 Input Data）是空的（0x00）。
![Figure 6-4](<./images/figure 6-4.png>)
图 6-4. 仅包含金额的交易（支付）

下一个示例同时指定了金额和数据载荷（尽管由于我们是向外部账户 EOA 发送交易，该载荷将被忽略）：
```Bash
$ cast send --account example vitalik.eth 0x0001 --value 0.001ether --rpc-url https://ethereum-sepolia-rpc.publicnode.com
```
在图 6-5 中，你可以看到输入数据（Input Data）现在包含了一些值，具体为 `0x0001`。
![Figure 6-5](<./images/figure 6-5.png>)
图 6-5. 同时包含金额和数据的交易
接下来的交易包含数据载荷，但指定的金额为零：
```Bash
$ cast send --account example vitalik.eth 0x0001 --rpc-url https://ethereum-sepolia-rpc.publicnode.com
```
图 6-6 显示的确认界面表明，交易中发送的以太币金额为零，而数据载荷等于 `0x0001`。
![Figure 6-6](<./images/figure 6-6.png>)
图 6-6. 仅包含数据的交易（调用）
最后，最后一笔交易既不包含发送金额，也不包含数据载荷：
```Bash
$ cast send --account example vitalik.eth --rpc-url https://ethereum-sepolia-rpc.publicnode.com
```
图 6-7 展示了我们这笔发送了零以太币且载荷为空的交易。
![Figure 6-7](<./images/figure 6-7.png>)
图 6-7. 既无金额也无数据的交易

## 向外部账户 (EOA) 和合约发送金额（Transmitting Value to EOAs and Contracts）
当你构建一笔包含金额（value）的以太坊交易时，这就等同于一笔支付。这类交易的行为取决于目标地址是否为合约地址。

对于 EOA 地址——或者更准确地说，对于任何在区块链上未被标记为合约的地址——以太坊将记录一次状态变更，将你发送的金额增加到该地址的余额中。如果该地址之前从未出现过，它将被添加到客户端的状态表示中，其余额将被初始化为你支付的金额。

如果目标地址（to）是一个合约（或者是此前通过 EIP-7702 交易委托了合约的 EOA），那么 EVM 将执行该合约，并尝试调用交易数据载荷（data payload）中指定的函数。如果你的交易中没有数据，EVM 将调用一个回退函数（fallback function）；如果该函数是可支付的（payable），则会执行它以决定下一步操作。如果没有回退函数，那么交易的效果将是增加该合约的余额，就像向钱包支付一样。

合约可以通过在函数被调用时立即抛出异常，或根据函数中编写的逻辑条件来拒绝接收付款。如果函数成功终止（未产生异常），那么合约的状态将更新，以反映合约以太币余额的增加。

## 向外部账户 (EOA) 或合约传输数据载荷（Transmitting a Data Payload to an EOA or Contract）
当你的交易包含数据（data）时，它极有可能是发往一个合约地址的。这并不意味着你不能向 EOA 发送数据载荷——在以太坊协议中这完全是有效的。然而，在这种情况下，对数据的解释完全取决于你用来访问该 EOA 的钱包。以太坊协议本身会忽略这些数据。大多数钱包也会忽略其控制的 EOA 收到的交易中的任何数据。未来可能会出现允许钱包像合约一样解释数据的标准，从而允许交易调用在用户钱包内运行的函数。关键区别在于，EOA 对数据载荷的任何解释都不受以太坊共识规则的约束，这与合约执行完全不同。

目前，我们假设你的交易是将数据发送到合约地址。在这种情况下，数据将被 EVM 解释为合约调用。大多数合约将这些数据具体用作函数调用，即调用指定的函数并将任何编码后的参数传递给该函数。

发送给符合 ABI（应用二进制接口） 标准合约（你可以假设所有合约都符合该标准）的数据载荷，是以下内容的十六进制序列化编码：

**函数选择器**（Function Selector） 
函数原型（prototype）的 Keccak-256 哈希值的前 4 个字节。这使得合约能够明确识别你想调用哪个函数。

**函数参数**（Function Arguments） 
根据 ABI 规范中定义的各种基本类型的规则进行编码的函数参数。

在示例 2-1 中，我们定义了一个用于提现的函数：
```
function withdraw(uint256 _withdrawAmount, address payable _to) public {
```
一个函数的原型（prototype）被定义为一个字符串，包含函数名称，后接其每个参数的数据类型，参数类型被包含在括号内并用逗号分隔。这里的函数名是 withdraw，它接受两个参数：

* `_withdrawAmount`：其类型为 `uint256`

* `_to`：其类型为 `address`

因此，withdraw 的函数原型应为：
```
withdraw(uint256,address)
```
> [!NOTE]
> 在 Solidity 中，payable 关键字用于指示地址可以接收以太币，但它不参与函数选择器的计算。在函数原型中仅包含基础类型 address。

让我们计算该字符串的 Keccak-256 哈希值：
```Bash
$ cast keccak256 "withdraw(uint256,address)"
0x00f714ce93c4a188ecc0c802ca78036f638c1c4b3ee9b98f3ed75364b45f50b1
```
该哈希的前 4 个字节是 0x00f714ce。这就是我们的**函数选择器**（Function Selector）值，它将告知合约我们要调用哪个函数。

接下来，让我们计算作为参数传递的 _withdrawAmount 和 _to 的值。我们想要提现 0.000001 ether 到地址 vitalik.eth（其对应的地址为 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045）。我们将它们与上一步计算出的函数选择器编码在一起，以获得最终的数据载荷（Data Payload）（也称为 calldata）：
```Bash
$ cast calldata "withdraw(uint256,address)" 0.000001ether 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045
0x00f714ce000000000000000000000000000000000000000000000000000000e8d4a51000000000000000000000000000d8da6bf26964af9d7eed9e03e53415d37aa96045
```
这就是我们交易的数据载荷，它调用了 `withdraw` 函数，并请求将 0.000001 ether 作为 `withdraw_amount` 提取到 `vitalik.eth` 指向的地址中。

## 特殊交易：合约创建（Special Transaction: Contract Creation）
有一种我们应当提到的特殊情况，即在区块链上创建并部署新合约以供未来使用的交易。合约创建交易的识别标志是 to 字段为空（null）。当 to 字段为空时，以太坊协议会将其解释为部署新合约的请求，而 data 字段中提供的则是该合约的字节码（bytecode）。

值得注意的是，零地址（0x0000000000000000000000000000000000000000）是一个截然不同的概念：它是一个有效的 20 字节地址，可以接收以太币。向零地址发送交易并不会创建合约，它只是像其他交易一样将以太币转移到该地址。零地址有时会收到来自各种地址的付款。这有两种解释：要么是操作失误导致以太币丢失，要么是故意的以太币销毁（通过发送到无法花费的地址来故意销毁以太币）。然而，如果你想进行有意的以太币销毁，你应该向网络明确你的意图，并使用专门指定的销毁地址：
```
0x000000000000000000000000000000000000dEaD
```
> [!WARNING]
> 任何发送到指定销毁地址的以太币都将变得无法花费，并永久丢失。

合约创建交易只需包含一个 data 载荷，该载荷包含用于创建合约的已编译字节码。此类交易的唯一效果就是创建合约。如果你想为新合约设置初始余额，可以在 value 字段中包含一定数量的以太币，但这完全是可选的。如果你向合约创建地址发送了金额（以太币）但没有数据载荷（没有合约代码），其效果等同于发送到销毁地址——因为没有合约可以接收这笔款项，以太币将会丢失。

例如，我们可以通过手动创建一笔 to 字段为空且 data 载荷为合约字节码的交易，来创建第 2 章中使用的 Faucet.sol 合约。首先，合约需要编译为字节码表示。这可以使用 Solidity 编译器（solc）完成：
```Bash
$ solc --bin Faucet.sol
Binary:
6080604052348015600e575f5ffd5…0033
```
同样的信息也可以从 Remix 在线编译器中获得。

现在我们可以使用二进制输出来创建交易：
```Bash
$ cast send --account example --rpc-url https://ethereum-sepolia-rpc.publicnode.com --create 6080604052348015600e575f5ffd5…0033
```
一旦合约发布，我们就可以在 Etherscan 区块浏览器上看到它，如图 6-8 所示。
![Figure 6-8](<./images/figure 6-8.png>)
图 6-8. Etherscan 上的合约创建交易
我们可以通过查看交易收据（使用交易哈希进行引用）来获取有关该合约的信息：
```Bash
$ cast receipt 0xa6b077d7d0ea21ff5f32a5a7243a81f0ab63e3b5e09c8e388c230fb067967cbb \
    --rpc-url https://ethereum-sepolia-rpc.publicnode.com
blockHash               0x6eb071eac79a84793321b086af96b32c1d861f04b0efc7354d0f6b8d5a8fa36a
blockNumber             7135544
contractAddress         0x4658eD241397F08cba8d5F3a69c7774cebE7f67F
cumulativeGasUsed       28390874
effectiveGasPrice       8867964529
from                    0x7e41354AfE84800680ceB104c5Fc99eCB98A25f0
gasUsed                 145123
logs                    []
logsBloom               0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
root
status                  1 (success)
transactionHash         0xa6b077d7d0ea21ff5f32a5a7243a81f0ab63e3b5e09c8e388c230fb067967cbb
transactionIndex        136
type                    2
blobGasPrice
blobGasUsed            
authorizationList
```

这包括合约的地址（参见 contractAddress 字段），我们可以使用该地址按照前一节所示的方式向合约发送资金或从合约接收资金。

让我们首先将新创建的合约地址保存到一个变量中：
```Bash
$ CONTRACT_ADDRESS=0x4658eD241397F08cba8d5F3a69c7774cebE7f67F
```
现在我们可以为该合约注入一些以太币：
```Bash
$ CONTRACT_ADDRESS=0x4658eD241397F08cba8d5F3a69c7774cebE7f67F
```
最后，让我们使用之前计算的数据载荷（data payload）来调用 withdraw 函数，将 0.000001 ether 提取到 vitalik.eth 地址：
```Bash
$ cast send --account example $CONTRACT_ADDRESS \
0x00f714ce000000000000000000000000000000000000000000000000000000e8d4a51000000000000000000000000000d8da6bf26964af9d7eed9e03e53415d37aa96045 \
  --rpc-url https://ethereum-sepolia-rpc.publicnode.com
```
过一段时间后，这两笔交易都可以在 Etherscan 上看到，如图 6-9 所示。
![Figure 6-9](<./images/figure 6-9.png>)
图 6-9. 合约注资与提现交易

## 数字签名（Digital Signatures）
到目前为止，我们还没有深入探讨数字签名的细节。在本节中，我们将研究数字签名是如何工作的，以及如何利用它在不泄露私钥的情况下证明对私钥的所有权。

### 椭圆曲线数字签名算法（ECDSA）
以太坊中使用的数字签名算法是椭圆曲线数字签名算法（ECDSA）。它基于椭圆曲线私钥-公钥对，正如“椭圆曲线密码学解释”中所描述的那样。

在以太坊中，数字签名有三个用途（见下文侧栏）。首先，签名证明了私钥的所有者（暗示也是以太坊账户的所有者）已授权支付以太币或执行合约。其次，它保证了不可否认性：授权证明是不可推翻的。第三，签名证明交易数据在签名后没有、也不可能被任何人修改。

#### 数字签名的定义
根据维基百科的说法，数字签名是一种用于表现数字消息或文档真实性的数学方案。一个有效的数字签名使接收者有理由相信：消息是由已知的发送者创建的（身份验证），发送者不能否认发送过该消息（不可否认性），且消息在传输过程中未被篡改（完整性）。

### 数字签名如何工作
数字签名是一种由两部分组成的数学方案。第一部分是使用私钥（签名密钥）从消息（在我们的案例中是交易）中创建签名的算法。第二部分是允许任何人仅使用消息和公钥来验证签名的算法。

#### 创建数字签名
在以太坊的 ECDSA 实现中，被签名的“消息”是交易，或者更准确地说，是交易中经过 RLP 编码数据的 Keccak-256 哈希值。签名密钥是 EOA 的私钥。其结果就是签名：
```
Sig = Fsig (Fkeccak256 (m), k)
```
其中：
* k 是签名私钥
* m 是经过 RLP 编码的交易
* Fkeccak256 是 Keccak-256 哈希函数
* Fsig 是签名算法
* Sig 是生成的签名
函数 Fsig 生成的签名 Sig 由两个值组成，通常被称为 r 和 s：
```
Sig = (r, s)
```
#### 验证签名（Verifying the signature）
要验证签名，你必须拥有签名（r 和 s）、序列化后的交易，以及与创建该签名时所用私钥相对应的公钥。从本质上讲，签名的验证意味着：只有生成该公钥的私钥持有者，才能针对这笔交易产生这个签名。

签名验证算法接受消息（在我们的用途中即交易的哈希值）、签名者的公钥以及签名（r 和 s 值），如果该签名对于此消息和公钥是有效的，算法则返回 true（真）。

#### ECDSA 数学原理（ECDSA Math）
正如前文所述，签名是由数学函数 Fsig 生成的，该函数产生由 r 和 s 两个值组成的签名。在本节中，我们将更详细地探讨函数 Fsig。签名算法首先以密码学安全的方式生成一个**临时**（ephemeral）私钥。这个临时密钥用于计算 r 和 s 的值，以确保攻击者在观察以太坊网络上的签名交易时，无法推算出发送者真正的私钥。

正如我们在第 4 章中所知，临时私钥用于推导出相应的（临时）公钥，因此我们有：
* 一个密码学安全的随机数 q，用作临时私钥。
* 相应的临时公钥 Q，由 q 和椭圆曲线生成点 G 生成。

数字签名的 r 值即为临时公钥 Q 的 x 坐标。
随后，算法计算签名的 s 值，满足：
$$s \equiv q^{-1} (Keccak256(m) + r \cdot k) \pmod{p}$$

其中：
* q 是临时私钥
* r 是临时公钥的 x 坐标
* k 是签名者（EOA 所有者）的私钥
* m 是交易数据
* p 是椭圆曲线的素数阶

验证是签名生成函数的逆过程，使用 r 和 s 值以及发送者的公钥来计算出一个值 Q，该值是椭圆曲线上的一个点（即创建签名时使用的临时公钥）。步骤如下：
1. 检查所有输入是否格式正确。
2. 计算 $w = s^{-1} \pmod{p}$ 。
3. 计算 $u_1 = Keccak256(m) \cdot w \pmod{p}$ 。
4. 计算 $u_2 = r \cdot w \pmod{p}$ 。
5. 最后，计算椭圆曲线上的点 $Q \equiv u_1 \cdot G + u_2 \cdot K \pmod{p}$ 。
其中：
* r 和 s 是签名值
* K 是签名者（EOA 所有者）的公钥
* m 是被签名的交易数据
* G 是椭圆曲线生成点
* p 是椭圆曲线的素数阶

如果计算出的点 Q 的 x 坐标等于 r，那么验证者就可以断定该签名是有效的。请注意，在验证签名的过程中，私钥既不为人所知，也不会被泄露。

> [!TIP]
> ECDSA 必然涉及相当复杂的数学运算；完整的解释超出了本书的范围。网上有许多逐步指导：搜索“ECDSA explained”或尝试参考相关教程。
>
> **译者** 也可以看下我自己总结的密码学笔记哦。

## 交易签名的实际操作（Transaction Signing in Practice）
要产生一笔有效的交易，发起者必须使用 ECDSA 算法对消息进行数字签名。当我们说“对交易签名”时，实际上是指“对 RLP 序列化交易数据的 Keccak-256 哈希值进行签名”。签名是作用于交易数据的哈希值，而不是交易本身。

在以太坊中签名一笔交易，发起者必须执行以下步骤：
1. 创建交易数据结构，包含该特定交易类型所需的所有字段。
2. 产生 RLP 编码的序列化消息，将交易数据结构转化为二进制流。
3. 计算该序列化消息的 Keccak-256 哈希值。
4. 计算 ECDSA 签名，使用发起方 EOA 的私钥对该哈希值进行签名。
5. 将计算出的 ECDSA 签名分量 $v$、$r$ 和 $s$ 添加到交易中。

特殊的签名变量 v 指示了两件事：链 ID（Chain ID）和用于帮助 `ecrecover` 函数校验签名的恢复标识符。在非遗留交易（non-legacy transactions）中，v 变量不再编码链 ID，因为链 ID 已作为交易本身的组成项之一被直接包含。有关链 ID 的更多信息，请参阅“使用 EIP-155 创建原始交易”。恢复标识符用于指示公钥 y 分量的奇偶性（详见“签名对称前缀值 (v) 与公钥恢复”, "The Signature Prefix Value (v) and Public Key Recovery"）。

> [!NOTE]
> 在区块高度 2,675,000 处，以太坊实施了伪龙（Spurious Dragon）硬分叉。在多项变更中，该分叉引入了一种新的签名方案，其中包含了交易重放保护（防止原本发送至某个网络的交易在其他网络上被重复执行）。这一新签名方案在 EIP-155 规范中定义。此项变更影响了交易的形式及其签名，因此必须特别注意三个签名变量中的第一个（即 v），它采用两种形式之一，并指示了被哈希的交易消息中包含哪些数据字段。

### 原始交易的创建与签名（Raw Transaction Creation and Signing）
在本节中，我们将使用 ethers.js 库来创建一笔原始交易（Raw Transaction）并对其进行签名。示例 6-1 展示了通常在钱包或代表用户签名交易的应用内部所使用的函数。

示例 6-1. 创建并签名一笔原始以太坊交易
```js
// 加载需求：
// npm install ethers
//
// Run with: node eip1559_tx.js
import { ethers } from "ethers";

// 使用你的 RPC 终端创建 provider
const provider = new ethers.JsonRpcProvider("https://ethereum-sepolia-rpc.publicnode.com");

// 私钥
const privKey = "0xd6d2672c6b4489e6bcd4e93b9af620fa0204b639b7d7f93765479c0846be0b58";

// 创建钱包实例
const wallet = new ethers.Wallet(privKey);

// 获取 nonce 并创建交易数据
const txData = {
  nonce: await provider.getTransactionCount(wallet.address), // Get nonce from provider
  to: "0xb0920c523d582040f2bcb1bd7fb1c7c1ecebdb34", // Receiver address
  value: ethers.parseEther("0.0001"), // Amount to send (0.0001 ETH here)
  gasLimit: ethers.toBeHex(0x30000), // Gas limit
  maxFeePerGas: ethers.parseUnits("100", "gwei"), // Max fee per gas
  maxPriorityFeePerGas: ethers.parseUnits("2", "gwei"), // Max priority fee
  data: "0x", // Optional data
  chainId: 11155111, // Sepolia chain ID
};

// 计算 RLP 编码的交易哈希（预签名）
const unsignedTx = ethers.Transaction.from(txData).unsignedSerialized;
console.log("RLP-Encoded Tx (Unsigned): " + unsignedTx);
const txHash = ethers.keccak256(unsignedTx);
console.log("Tx Hash (Unsigned): " + txHash);

// 签名并发送交易
async function signAndSend() {
  // 使用钱包对交易进行签名
  const signedTx = await wallet.signTransaction(txData);
  console.log("Signed Raw Transaction: " + signedTx);

  // 将签名后的交易广播到以太坊网络
  const txResponse = await provider.broadcastTransaction(signedTx);
  console.log("Transaction Hash: " + txResponse.hash);

  // 等待交易被打包（矿工处理）
  const receipt = await txResponse.wait();
  console.log("Transaction Receipt: ", receipt);
}

signAndSend().catch(console.error);
```
Running the example code produces the following results:
```Bash
$ node eip1559_tx.js 
RLP-Encoded Tx (Unsigned): 0x02f283aa36a714847735940085174876e8008303000094b0920c523d582040f2bcb1bd7fb1c7c1ecebdb34865af3107a400080c0
Tx Hash (Unsigned): 0x31d43a580534a77c71324a8434df6f2df993b3d551b29d4b70d8a889768a53f7
Signed Raw Transaction: 0x02f87583aa36a714847735940085174876e8008303000094b0920c523d582040f2bcb1bd7fb1c7c1ecebdb34865af3107a400080c001a03f8ed18cb03ee0fe3fbc3f0a7477a2f68db6ec84450e77e702b82a3f2c873aa4a0205c4f6a16ea8ad13a148cc3105814cd4a6860cd26a771651199c85ccb7c7f0f
Transaction Hash: 0x07bfbeb337e19763a1f74d989dae2953807dcb06822354cfefb16405a11beb93
Transaction Receipt:  TransactionReceipt {
  provider: JsonRpcProvider {},
  to: '0xB0920c523d582040f2BCB1bD7FB1c7C1ECEbdB34',
  from: '0x7e41354AfE84800680ceB104c5Fc99eCB98A25f0',
  contractAddress: null,
  hash: '0x07bfbeb337e19763a1f74d989dae2953807dcb06822354cfefb16405a11beb93',
  index: 1,
  blockHash: '0x0ac051e8f615805c69eec6e193e39637adeb7cf314a0098d455e7d9ac395a7ee',
  blockNumber: 7135937,
  logsBloom: '0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000',
  gasUsed: 21000n,
  blobGasUsed: null,
  cumulativeGasUsed: 42000n,
  gasPrice: 3096751769n,
  blobGasPrice: null,
  type: 2,
  status: 1,
  root: undefined
}
```
### 底层手动构建 EIP-1559 原始交易（Low-Level Manual Construction of an EIP-1559 Raw Transaction）
以下示例演示了创建并签名一笔 EIP-1559（类型为 0x02）交易的完整底层流程。通过手动构建字段列表、进行 RLP 编码并对类型化载荷（typed payload）进行签名，这一方法揭示了 wallet.signTransaction 内部究竟发生了什么。
```js
// npm install ethers@6
import { ethers } from "ethers";
const RLP = require('@ethereumjs/rlp');

// 替换为你自己的 RPC 和私钥
const rpcUrl = "https://ethereum-sepolia-rpc.publicnode.com"; // 或主网等
const provider = new ethers.JsonRpcProvider(rpcUrl);
const privateKey = "0xYOUR_PRIVATE_KEY_HERE";
const wallet = new ethers.Wallet(privateKey, provider);

const recipient = "0xRECIPENT_ADDRESS"; // example address

async function createAndSendRawEip1559Tx() {
  const chainId = await provider.getChainId(); / 这是一个 RPC 调用
  const nonce = await provider.getTransactionCount(wallet.address); // 这是一个 RPC 调用

  // 示例参数 —— 根据需要调整（或使用 provider.estimateGas + getFeeData）
  const maxPriorityFeePerGas = ethers.parseUnits("2", "gwei");   // 小费 (tip)
  const maxFeePerGas = ethers.parseUnits("30", "gwei");         // 费用上限 (基础费 + 小费)
  const gasLimit = 21000n;
  const value = ethers.parseEther("0.01"); // 0.01 ETH
  const data = "0x"; // 或合约调用数据 (calldata)
  const accessList = []; // [] 或用于 EIP-2930 类型节省费用的适当访问列表

  // 交易类型 2 (Transaction Type 2) 严格排序的未签名字段
  const unsignedFields = [
    ethers.toBeHex(chainId),              // 链 ID
    ethers.toBeHex(nonce),                // nonce
    ethers.toBeHex(gasLimit),             // gas 限制
    ethers.toBeHex(maxPriorityFeePerGas), // 最大优先费用 (maxPriorityFeePerGas)
    ethers.toBeHex(maxFeePerGas),         // 最大费用 (maxFeePerGas)
    recipient,                            // 接收者 (to)
    ethers.toBeHex(value),                // 金额 (value)
    data,                                 // 数据 (data)
    accessList                            // 访问列表 (accessList)
  ];

  // 对未签名字段进行 RLP 编码
  const encodedUnsigned = RLP.encode(unsignedFields);

  // 在开头添加交易类型字节 (0x02)
  const txType = Buffer.from('02', 'hex');
  const fullTx = Buffer.concat([txType, encodedUnsigned]);

  // 这是用于签名的哈希
  const signingHash = ethers.keccak256(fullTx);

  // 对哈希进行签名
  const signingKey = new ethers.SigningKey(wallet.privateKey);
  const signature = signingKey.sign(ethers.getBytes(signingHash));

  // 对于类型化交易，我们使用 yParity (0 或 1) 来替代传统的 v
  const signedFields = [
    ...unsignedFields,
    sig.v - 27, // yParity (0 or 1)
    sig.r,
    sig.s
  ];

  const encodedSigned = RLP.encode(signedFields);

  const rawTransaction = '0x02' + Buffer.from(encodedSigned).toString('hex');

  console.log("Raw transaction hex:\n", rawTransaction);

  // 发送原始交易
  const txHash = await provider.send("eth_sendRawTransaction", [rawTransaction]);
  console.log("Transaction sent! Hash:", txHash);

  const receipt = await provider.waitForTransaction(txHash);
  console.log("Mined in block", receipt.blockNumber);
}

createAndSendRawEip1559Tx().catch(console.error);
```
## 交易的反序列化（Deserializing the Transaction）
现在我们已经从零开始创建并向以太坊网络发送了一笔交易，我们可以遵循其逆过程，尝试从上一个示例中获得的已签名原始交易（signed raw transaction）开始，重新构建交易的每一个字段。这将帮助你理解交易的各个字段是如何实际包含在交易本身之中的——你只需要以正确的方式将其提取出来。

让我们从完整的已签名原始交易字符串开始：
```
0x02f87583aa36a714847735940085174876e8008303000094b0920c523d582040f2bcb1bd7fb1c7c1ecebdb34865af3107a400080c001a03f8ed18cb03ee0fe3fbc3f0a7477a2f68db6ec84450e77e702b82a3f2c873aa4a0205c4f6a16ea8ad13a148cc3105814cd4a6860cd26a771651199c85ccb7c7f0f
```
回想一下 EIP-2718，所有以太坊交易都由以下部分组成：
* 首字节：指定交易类型。
* 交易类型载荷 (Payload)

这几乎总是转化为该特定交易类型所有字段的 RLP 编码。我们的交易以 0x02 开头。这代表了交易类型，而 0x02 意味着它是一个 EIP-1559 交易。接下来的部分是构成 EIP-1559 交易的所有交易字段的 RLP 编码。

为了快速解码这些字段，我们可以使用 cast 工具的 from-rlp 命令，并将交易的剩余部分作为输入：
```Bash
$ cast from-rlp f87583aa36a714847735940085174876e8008303000094b0920c523d582040f2bcb1bd7fb1c7c1ecebdb34865af3107a400080c001a03f8ed18cb03ee0fe3fbc3f0a7477a2f68db6ec84450e77e702b82a3f2c873aa4a0205c4f6a16ea8ad13a148cc3105814cd4a6860cd26a771651199c85ccb7c7f0f
["0xaa36a7","0x14","0x77359400","0x174876e800","0x030000","0xb0920c523d582040f2bcb1bd7fb1c7c1ecebdb34","0x5af3107a4000","0x",[],"0x01","0x3f8ed18cb03ee0fe3fbc3f0a7477a2f68db6ec84450e77e702b82a3f2c873aa4","0x205c4f6a16ea8ad13a148cc3105814cd4a6860cd26a771651199c85ccb7c7f0f"]
```
你可以看到上一个 cast 命令的输出包含了一系列十六进制项：它们正是 EIP-1559 交易的各个字段。让我们逐一分析并重构这笔交易：
```
0xaa36a7Sepolia
```
测试网的链 ID（Chain ID）：转为十进制是 11155111。
```
0x14
```
交易使用的随机数（Nonce）：转为十进制是 20。你可以去区块浏览器核实，这个数值是完全正确的。
```
0x77359400
```
最大优先费用（Max Priority Fee per gas）：十进制为 2,000,000,000，即每单位 Gas 2 Gwei。
```
0x174876e800
```
最大费用（Max Fee per gas）：十进制为 100,000,000,000，即每单位 Gas 100 Gwei。
```
0x030000
```
Gas 限制（Gas Limit）：十进制为 196,608。
```
0xb0920c523d582040f2bcb1bd7fb1c7c1ecebdb34
```
接收者地址。
```
0x5af3107a4000
```
发送的以太币数量（以 Wei 为单位）：十进制为 $10^{14}$ 。这等同于 0.0001 ETH。
```
0x
```
空的数据载荷（Data Payload），说明这是一笔简单的转账。
```
[]
```
空的访问列表（Access List）。
```
0x01
```
签名的 $v$ 值；0x01 表示椭圆曲线公钥的 $y$ 坐标为奇数（0x00 则表示偶数）。
```
0x3f8ed18cb03ee0fe3fbc3f0a7477a2f68db6ec84450e77e702b82a3f2c873aa4
```
签名的 $r$ 值，代表椭圆曲线上随机点的 $x$ 坐标。
```
0x205c4f6a16ea8ad13a148cc3105814cd4a6860cd26a771651199c85ccb7c7f0f
```
签名的 $s$ 值，代表签名的证明部分。

### 使用 EIP-155 创建原始交易（Raw Transaction Creation with EIP-155）
EIP-155 “简单重放攻击保护”标准规定了一种受重放攻击保护的交易编码方式，它在签名之前的交易数据中包含了一个链 ID（Chain ID）。这确保了为某个区块链（例如以太坊主网）创建的交易在另一个区块链（例如以太坊经典或 Sepolia 测试网）上是无效的。因此，在一个网络上广播的交易无法在另一个网络上被重放，该标准也因此得名。

通过在被签名的数据中包含链 ID，交易签名可以防止任何篡改，因为一旦修改链 ID，签名就会失效。因此，EIP-155 使得交易无法在其他链上重放，因为签名的有效性取决于链 ID。

链 ID 字段根据交易预定的网络取值，如表 6-2 所示：

表 6-2. 常见链标识符（Chain Identifiers）

![Table 6-2](<./images/table 6-2.png>)

欲了解详尽的链标识符列表，请参阅 [ChainList](https://chainlist.org/)。
最终的交易结构会经过 RLP 编码、哈希处理并进行签名。更多细节请参考 EIP-155 规范。

### 签名对称前缀值 (v) 与公钥恢复 (Public Key Recovery)
正如“交易的结构”中所提到的，以太坊的交易消息中并不包含 from（发送者）字段。这是因为发起者的公钥可以直接从 ECDSA 签名中计算出来。一旦获得了公钥，就可以轻松计算出地址。这种从签名中恢复签名者公钥的过程被称为公钥恢复。给定在“ECDSA 数学原理”中计算出的 $r$ 和 $s$ 值，我们可以计算出两个可能的公钥。首先，我们根据签名中的 $x$ 坐标 $r$ 值，计算出两个椭圆曲线上的点 $R$ 和 $R'$。之所以存在两个点，是因为椭圆曲线相对于 $x$ 轴是对称的。因此，对于任何 $x$ 值（即 $r$），在曲线上都有两个可能的 $y$ 值与之对应，分别位于 $x$ 轴的两侧。接着，我们根据 $r$ 计算出 $r^{-1}$，即 $r$ 的乘法逆元。最后，我们计算 $z$，它是消息哈希值的低 $n$ 位，其中 $n$ 是椭圆曲线的阶。这两个可能的公钥计算公式如下：

$$K_1 = r^{-1} (sR - zG)$$

和

$$K_2 = r^{-1} (sR' - zG)$$

其中：
* $K_1$ 和 $K_2$ 是签名者公钥的两种可能性。
* $r^{-1}$ 是签名 $r$ 值的乘法逆元。
* $s$ 是签名的 $s$ 值。
* $R$ 和 $R'$ 是临时公钥 $Q$ 的两种可能性。
* $z$ 是消息哈希的低 $n$ 位。
* $G$ 是椭圆曲线生成点。

为了提高效率，交易签名包含了一个前缀值 $v$，它告诉我们两个可能的 $R$ 值中哪一个是真正的临时公钥。如果 $v$ 是偶数，那么 $R$ 就是正确的值；如果 $v$ 是奇数，那么正确的就是 $R'$。通过这种方式，我们只需要计算一个 $R$ 值和唯一的一个公钥 $K$。

## 签名与传输的分离（离线签名）
一旦交易完成签名，它就可以传输到以太坊网络。通常，创建、签名和广播交易这三个步骤是作为单一操作发生的（例如使用 `cast send` 命令）。然而，正如你在“原始交易的创建与签名”中所看到的，你可以分两个独立的步骤来创建和签名交易。一旦你拥有了已签名的交易，就可以使用 `ethers.JsonRpcProvider("...").broadcastTransaction` 进行传输，该函数接受十六进制编码且已签名的交易并将其传输到以太坊网络。

为什么需要将签名的生成与传输分离？最常见的原因是安全性。对交易进行签名的计算机必须在内存中加载解锁的私钥。而执行传输的计算机必须连接到互联网（并运行以太坊客户端）。如果这两个功能在同一台计算机上，你的私钥就暴露在联网系统中，这是非常危险的。
> [!WARNING]
> 如果你将私钥保存在联网设备中，你将面临多种形式的攻击，如恶意软件和远程黑客攻击，同时也更容易受到钓鱼攻击。

将签名和传输功能分离并在不同的机器上执行（分别在离线和在线设备上），被称为离线签名（Offline Signing），这是一种常见的安全实践。

图 6-10 展示了该过程，具体步骤如下：

1. 在线计算机：创建一个未签名的交易。在此处可以获取账户的当前状态，特别是当前的随机数（nonce）和可用资金。

2. 离线设备：将未签名的交易转移到“气隙（air-gapped）”离线设备进行交易签名（例如通过二维码或 USB 闪存驱动器）。

3. 在线广播：将已签名的交易传输（回）到在线设备，以便在以太坊区块链上广播（例如通过二维码或 USB 闪存驱动器）。

![Figure 6-10](<./images/figure 6-10.png>)
图 6-10. 离线签名流程

根据你对安全级别的需求，“离线签名”计算机与在线计算机的分离程度可以有所不同。其范围可以从受防火墙保护的隔离子网（在线但隔离），到完全不联网的系统，即所谓的气隙系统（Air-gapped System）。在气隙系统中，不存在任何网络连接——计算机通过一段“空气”与在线环境完全隔绝。为了签名交易，你需要使用数据存储介质或（更好的方式）通过摄像头扫描二维码来往返传输数据。当然，这意味着你必须手动传输每一笔需要签名的交易，因此这种方式不具备大规模扩展性。

虽然并非所有环境都能利用完全的气隙系统，但即使是小程度的隔离也能带来显著的安全收益。例如，一个带有防火墙的隔离子网，如果仅允许通过某种消息队列协议，那么与在联网系统上签名相比，它能提供更小的攻击面和更高的安全性。许多公司为此使用 ZeroMQ (0MQ) 等协议。在这种架构下，交易会被序列化并进入待签名队列。队列协议类似于 TCP 套接字，将序列化后的消息传输给签名计算机。签名计算机（谨慎地）从队列中读取序列化交易，使用相应的密钥实施签名，然后将其放入传出队列。传出队列再将已签名的交易传输给运行以太坊客户端的计算机，由后者将其从队列中取出并广播到网络。

## 交易生命周期（Transaction Life Cycle）
在本节中，我们将探索一笔交易的完整生命周期：从它被签名的那一刻起，到它被包含进区块，直至该区块最终被确认（Finalized）。

### 创建并签名交易（Creating and Signing the Transaction）
第一步是创建交易，选择交易类型并填写该类型所需的所有字段。例如，在“原始交易的创建与签名”一节中，我们创建了一笔 EIP-1559 交易。

一旦拥有了交易数据，我们就需要使用正确的私钥对其进行签名（否则交易将因签名无效而变为废件），从而获得最终且确定的“已签名交易”。这才是我们真正需要发送给网络并等待被打包进区块的原始数据。

### 将交易发送至网络（Sending the Transaction to the Network）
交易必须被包含在一个区块中，才能被以太坊协议视为“已确认”；否则，它仅仅是一段只有我们自己知道的已签名数据。

> [!TIP]
> 极其重要的一点是：以太坊协议仅认可那些被包含在有效链区块内的交易。为了更新以太坊的状态（例如完成转账或执行合约），你必须将已签名的交易发送到网络，并等待它被打包进区块。单凭一串已签名的交易数据本身不会产生任何效果，除非它被成功纳入区块。

因此，我们需要将签名后的交易发送到网络。为此，我们只需将交易发送到一个以太坊节点即可：这可以是您自己的节点，也可以是第三方节点，例如 Alchemy、Infura 或 Public Node（我们在之前的示例中使用的就是后者）。
> [!NOTE]
> 默认情况下，几乎所有钱包都使用第三方节点，这样用户无需安装任何软件即可开始使用以太坊网络。尽管如此，如果您希望最大限度地保护隐私并真正做到不依赖任何人，您应该使用自己的客户端。您可以参考第 3 章，了解如何安装您的第一个以太坊节点的详细指南。

当我们的签名交易到达第一个节点时，该节点会执行一些验证，以便立即清除垃圾交易或无效交易。如果验证通过，该节点会将交易添加到其**内存池**（mempool）中，并将其传播给它的一部分对等节点（peers）。每个收到交易的节点都会对其进行验证，将其加入自己的内存池，并进一步传播。

这个过程是以太坊 **P2P Gossip 协议**（流言协议）的一部分。其结果是，在短短几秒钟内，一笔以太坊交易就能传播到全球所有的以太坊节点。从每个节点的角度来看，无法辨别交易的来源。向该节点发送交易的邻居节点可能是交易的发起者，也可能是从它的另一个邻居那里接收到的。为了能够追踪交易的起源或干扰传播，攻击者必须控制全网相当大比例的节点。这是 P2P 网络安全和隐私设计的一部分，尤其适用于区块链网络。

> [!NOTE]
> 你可能会好奇，为什么节点不将交易“淹没式”地发送给所有邻居，而仅仅发送给一部分邻居。答案是效率和带宽保护。事实上，将所有交易发送给所有节点是极低效的：这会产生大量的重复消息，导致网络流量激增，并且随着交易数量的增加，网络流量会呈指数级增长，从而导致扩展性变得极差。

### 构建区块（Building the Block）
此时此刻，我们的交易已经到达了几乎所有的以太坊节点，但它仍然未被确认，因为它尚未被包含在区块中。这种情况会一直持续，直到某位被选中提议下一个区块的**验证者**（Validator）最终从其自身的内存池中取出交易，将它们添加到区块中，并将该区块发布到网络上。

一旦交易被包含在区块中，它们就会修改以太坊的状态。这种修改可能是简单的账户余额变动（如普通转账），也可能是调用智能合约从而改变其内部状态存储。这些执行结果并不会直接保存在交易数据中，而是以**交易收据**（Transaction Receipt）的形式记录。收据详细说明了交易执行的结果，包括是否成功、消耗的 Gas 以及产生的事件。

### 交易的最终确认（Finalizing the Transaction）
现在，我们的交易已被包含在一个区块中，并且已经修改了以太坊的状态。然而，尽管可能性极低，包含该交易的区块仍有可能被撤销，并被另一个不包含该交易的区块所取代。这种情况被称为区块重组（Block Reorganization），简称为 Reorg。

为了完全确定交易不可逆转，我们需要等待包含该交易的区块被以太坊共识协议最终确认（Finalized）（我们将在第 15 章详细探讨这一机制）。在以太坊目前的权益证明（PoS）机制下，这通常需要大约 12.8 分钟（即两个 Epoch）。

> [!NOTE]
> 虽然只有等到“最终确认”才能百分之百确定交易已在区块链上定格，但通常情况下，你只需要等待几个区块即可。大多数钱包会在交易被纳入区块后立即将其显示为“已确认”。

## 另一种生命周期
我们在前一节中探讨的生命周期是交易遵循的传统标准流程。然而，在过去的三四年里，尤其是在当下，一种全新的交易生命周期已经建立。为了解释这一点，我们需要引入一个核心概念：提案者与构建者分离（Proposer-Builder Separation，简称 PBS）。

### MEV 与提案者-构建者分离（PBS）
正如我们将在第 15 章深入探讨的那样，以太坊每 12 秒就需要一名验证者提议一个新区块，以推动链的增长。验证者从其内存池中收集交易，将它们组织起来填满区块，并将区块发布到网络中，以便所有其他节点进行验证和传播。

传统上，以太坊节点为了实现利润最大化，会根据用户支付的交易费用（具体而言，是 EIP-1559 交易中的优先费/小费）对交易进行排序。多年来，这一直是优化矿工和验证者收益的标准方法。

然而，随着 2020 年“DeFi 之夏”以及 2021 年牛市期间以太坊普及度的飙升，出现了一种新现象：最大可提取价值（Maximal Extractable Value，简称 MEV，此前称为矿工可提取价值）。这一概念从根本上改变了以太坊矿工和验证者创建区块的方式。

MEV 指的是区块生产者通过以下战略性决策，能从区块中提取的最大价值：
* 包含（Inclusion）：决定哪些交易可以进入区块。
* 排序（Ordering）：调整交易在区块内的先后位置。
* 排除（Exclusion）：故意忽略某些高费用的交易。

虽然这听起来似乎无伤大雅，但它对区块生产的动态产生了（并且仍在产生）巨大的影响。例如，设想有一笔交易准备在像 Uniswap 这样的去中心化交易所购买大量特定的代币 X。验证者可以看到这笔交易，并预先知道它会导致价格大幅上涨。这意味着验证者可以添加两笔交易来从中获利：
1. 第一笔交易：买入一些代币 X，并将其顺序排在大额买单交易之前。
2. 第二笔交易：卖出在前一笔交易中买入的代币 X，并将其顺序排在大额买单交易之后。

这就是所谓的三明治攻击（Sandwich Attack）：验证者利用大额买单交易所产生的滑点（Slippage）来获利。让我们通过一个简化的例子来演示这个概念：
假设一名用户提交了一笔交易（tx1），购买 1,000 个 xyz 代币（如图 6-11 所示）。假设 xyz 代币的价格是 1,000 美元，因此该用户将购买价值 100 万美元的代币。这种买入压力会将代币价格推高至 1,010 美元。
![Figure 6-11](<./images/figure 6-11.png>)
图 6-11. MEV 干预前的用户交易
但验证者看到了通过“抢跑” tx1 赚钱的机会。他们创建了两笔交易：tx0 和 tx2。其中 tx0 包含 10 个 xyz 代币的买单，而 tx2 是等额代币的卖单。验证者将这些交易精确地放置在用户 tx1 的前后，如 所示。请记住，验证者之所以能做到这一点，是因为他们是区块的实际创建者，因此可以自由选择区块内交易的排序。

在这种情况下，验证者的 tx0 会以较低的价格买入，随后用户的 tx1 执行并大幅抬高价格，最后验证者的 tx2 以最高价卖出。通过这种方式，验证者无风险地赚取了价差。
![Figure 6-12](<./images/figure 6-12.png>)
图 6-12. 验证者的三明治攻击
结果是，验证者能够以 1,000 美元的价格买入 10 个 xyz 代币，并以 1,010 美元的价格卖出，从而赚取了 $10 \times 10 = 100$ 美元的利润。

这是验证者用来实现利润最大化的一种非常简单的策略。实际上还存在许多其他更复杂的策略。由于竞争异常激烈，一个新的参与者出现了：构建者（Builders）。事实上，运行所有这些策略需要大量的计算能力，远超普通验证者所拥有的资源（请记住，只需 16 GB 内存即可运行一个验证者节点）。这意味着 MEV 也在威胁验证者的去中心化，因为它更偏向于那些能够支付数百万美元用于基础设施、并雇佣人员专门研究高利润策略以构建“更好”区块的大型实体。

这一问题的解决方案出现在 2021 年 1 月，随着 Flashbots v0.1 的发布。它使区块提议者（起初是矿工，现在是验证者）能够以去信任的方式，将寻找最优区块构建的任务外包给这些被称为“构建者”的新实体。得益于这种职责分离——由构建者负责挑选交易填满区块并创建支付给验证者费用最高的区块（构建者也会从中抽取一部分费用），而验证者仅负责向网络提议区块——验证者仍然可以在普通的商用机器上运行。

### 私人内存池 (Private Mempools)
MEV 生态系统已经发展到如此庞大的规模，以至于现在构建者之间的竞争不仅体现在寻找最大化利润的最优策略上，还体现在他们能获取哪些交易来填充区块。他们掌握的交易越多，就越能更好地实施其策略。

这导致了私人内存池的出现。这些内存池允许用户或实体将交易直接提交给区块生产者，而无需将其暴露给公共网络。它们在缓解抢跑风险和实现隐私工作流方面发挥着重要作用。

Flashbots 开发了自己的解决方案，称为 [Flashbots Protect](https://oreil.ly/Jq4rT)。目前最流行的钱包 MetaMask 也已开始为用户默认使用私人内存池（称为[智能交易](https://oreil.ly/Jq4rT)）。

### 新的交易生命周期
MEV、提案者与构建者分离（PBS）以及私人内存池的出现，彻底改变了区块生产的环境。如今，大量交易不再遵循我们之前解释的常规生命周期；在创建并签名后，它们会通过私人内存池直接发送给构建者（Builder）。构建者会处理这些交易，尝试将它们纳入一个最优化的区块中，并最终将整个区块发送给负责提议下一个区块的验证者（Validator）。在这种模式下，交易跳过了公共内存池的传播过程，直接出现在已构建好的区块中。

## 多重签名交易 (Multiple-Signature Transactions)
如果你熟悉比特币的脚本功能，就会知道可以创建一个比特币多重签名（Multisig）账户，只有在多个参与方共同签署交易时（例如 2/2 或 3/4 签名）才能动用资金。以太坊基础的 EOA（外部账户）价值交易并没有多重签名的直接条款；然而，任何复杂的签名限制都可以通过智能合约来实现，你可以设定任何能想到的条件来处理以太币和代币的转账。

要利用这一功能，以太币必须先转移到一个预先编写好支出规则的钱包合约中，比如多重签名要求或支出限额（或两者的结合）。一旦支出条件得到满足，钱包合约就会在经授权的 EOA 触发下发送资金。例如，为了通过多签条件保护你的以太币，你需要将其转入一个多签合约。每当你想要向另一个账户发送资金时，所有要求的授权用户都需要使用普通的钱包应用向该合约发送交易，这本质上是授权该合约执行最终的转账。

这些合约还可以设计为在执行本地代码或触发其他合约之前需要多个签名。该方案的安全性最终取决于多签合约的代码。

以智能合约的形式实现多重签名交易，充分展示了以太坊极高的灵活性。目前，Safe（原名 Gnosis Safe）已成为创建多重签名账户的行业事实标准。这套经过严苛实战检验的智能合约被各大主流协议和 DAO 广泛采用。如图 6-13 所示，截至 2024 年 11 月，它保护着超过 60 亿美元的 ETH 以及价值超过 740 亿美元的 ERC-20 代币。

![Figure 6-13](<./images/figure 6-13.png>)
图 6-11. Gnosis Safe 保护着价值数十亿美元的资产

> [!NOTE]
> 使用 Gnosis Safe 执行交易的常规流程如下：
> 1. 发起提议：Safe 的其中一名签名者发起一笔他们想要签署并发送到以太坊网络的交易。
> 2. 收集签名：其他签名者查看该交易，如果同意该交易的意图，则进行签名。
> 3. 执行交易：当达到法定人数（Quorum）（即预设的最小签名数量）后，交易最终被发送到网络进行处理和执行。

## 结论
交易是以太坊系统中每一项活动的起点。交易就像是“输入”，它们驱动 EVM（以太坊虚拟机）去评估合约、更新余额，并从更广泛的层面修改以太坊区块链的状态。接下来，我们将更详细地深入研究智能合约，并学习如何使用面向合约的编程语言——Solidity 进行编程开发。


---
[^1]: 也可以理解为经典交易，不过在以太坊社区中多用遗留交易。
[^2]: 可以简单地理解为交易计数，但是社区更喜欢直接引用原词。
