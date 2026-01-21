# 第 12 章：去中心化应用（DApps）

在本章中，我们将揭开 DApps 的神秘面纱，解释它们的定义、工作原理以及核心架构的构成。在涵盖了基础知识之后，我们将通过一个实际的动手示例，
带你从零开始构建你的第一个 DApp。这包括部署必要的智能合约、集成前端，并为准生产环境准备整个技术栈。到本章结束时，
你不仅将拥有一个可以运行的 DApp，还将对支撑这些创新应用的核心概念有深刻的理解。

## 什么是 DApp？
DApp 是 Decentralized Application（去中心化应用） 的缩写。与传统应用相比，它代表了一种完全的新范式。
在传统应用（我们通常称为“遗留应用”或 Web2 应用）中，架构通常由以下三部分组成（正如图 12-1 所示）：

* 逻辑部分的封闭源代码：应用程序的核心业务逻辑运行在私有服务器上，代码不透明且由公司控制。
* 中心化数据库：用于存储应用数据（如用户信息、交易记录），数据的所有权和修改权归运营方所有。
* 唯一的客户端前端：用户通过特定的前端界面访问应用，如果该前端被关闭，用户通常无法再与后台交互。

![Figure 12-1](<./images/figure 12-1.png>)
图 12-1. 传统应用的通用架构

想想 Instagram、TikTok、你的银行，或者你手机上现在的任何应用。它们可能都依赖于非常相似的架构。只有在背后的团队允许时，
你才能访问这些应用；如果官方网站停止服务，并没有其他的替代网站能让你登录 Instagram 账号。

DApps 有两个明确的目标：不设单点故障，以及成为一个即使整个开发团队消失，人们仍能继续使用的产品。它们的架构可以简化为以下方式，
正如图 12-2 所示：
* 核心逻辑：由数个以太坊智能合约构成。大多数情况下，Solidity（或 Vyper）代码也是开源的。
* 数据存储：智能合约也可以包含数据，充当正式的数据库并收集所有必要的用户信息。
* 访问入口：人们可以通过官方前端访问应用。如果主前端因任何原因无法工作，它可以轻易地被替代前端（甚至是社区制作的前端）所取代。

![Figure 12-2](<./images/figure 12-2.png>)
图 12-2. DApp 的通用架构

> [!Note]
> DApp 也可以包含一些具有一定中心化程度的链下组件，但通常这些组件对于应用程序的核心逻辑来说并非必不可少。
> 它们可能有助于在一般情况下提高应用的运行速度，但在最坏的情况下，应当始终能够完全依赖链上数据运行。然而事实并非总是如此，
> 确实有一些 DApp 的核心逻辑部分依赖于中心化组件。它们并非真正的去中心化应用，而更像是传统应用与完全去中心化应用之间的混合形态（Hybrid form）。

在接下来的章节中，我们将进一步探索 DApp 技术栈的每个组件，以便更好地理解它们的工作原理，以及它们如何相互关联并与以太坊协议交互。

## 后端（智能合约）

在 DApp 中，核心业务逻辑和数据存储被编码进智能合约中，并运行在以太坊区块链上，而不是驻留在中心化服务器。
区块链充当了去中心化的后端，交易执行、状态变更和记录保存由整个网络以去信任化的方式强制执行，而非依赖单一实体。

用户无需信任任何中心化团队即可访问 DApp，因为他们可以预见以太坊网络将始终正常运行，无论何时何地都能处理他们的交易并正确更新智能合约的状态。

此外，这种架构引入了一个非常强大的属性：抗审查性。在传统应用中，由于政府法律或其他原因，经常会有一份禁止访问该应用的国家名单。
例如，截至 2025 年，Facebook 在巴西、中国、伊朗、朝鲜、缅甸、俄罗斯、土库曼斯坦和乌干达等国被禁止使用。这些国家的人们无法创建个人资料或登录该平台。

对于构建在以太坊上的 DApp 来说，这种类型的审查已无法实现。尽管仍有可能封锁某个 DApp 的官方网站，但没有人能阻止一个地址与链上的随机智能合约进行交互。
任何人都可以介入并为该 DApp 创建替代前端，而所有人都可以再次通过它与 DApp 交互。

### Tornado Cash 传奇

在这里非常值得一提 Tornado Cash 的故事。Tornado Cash 是一种去中心化的混合（Mixing）服务，它使任何地方的任何人都能将可追溯或“被标记”的加密货币与其他资金混合。
通过切断资金真实发送者与接收者之间的所有联系，它能够模糊回溯至原始资金来源的路径。

2022 年 8 月 8 日，美国财政部海外资产控制办公室（OFAC）将 Tornado Cash 列入黑名单，实际上禁止了美国公民和公司使用它。
该平台被指控洗钱超过 70 亿美元的加密货币。两天后，即 8 月 10 日，Tornado Cash 的开发者之一 Alexey Pertsev 在阿姆斯特丹被捕，
其罪名仅仅是创建了该平台本身。官方的 GitHub 代码库被删除，开发者的账号被封禁。截至 2024 年 12 月，官方网站仍然无法访问。

鉴于这一切，你可能会认为 Tornado Cash DApp 已经停止运行了，但事实远非如此。虽然自这些事件发生以来，
该服务获得的关注（和流动性）有所减少，但该协议在功能上仍然完全正常，并且可以通过托管在 IPFS 上的各种网关进行访问。
换句话说，世界上的任何人仍然可以像以前一样使用 Tornado Cash，即使没有原始的官方网站。


## 数据存储

数据存储是指用于保存用户数据的解决方案。虽然在智能合约中存储和读取数据是可行的，但这是一种昂贵的操作，且扩展性较差；
此外，并非所有内容都需要保存在链上。

通常，智能合约存储关键的状态信息并执行 DApp 逻辑。核心信息（如账户余额、所有权记录或计算结果）直接存储在智能合约中。
这确保了敏感且具有价值的数据保持完全透明、抗篡改，并可供任何拥有以太坊节点的人访问。

所有其他信息可以保存在去中心化程度较低的存储方案中，例如传统数据库。例如，DApp 开发者经常使用**索引器**（Indexers）来实现快速数据查询，
或使用中心化数据库来存储用户信息，这样前端就完全不需要与以太坊链进行交互。

> [!Note]
> 区块链索引器（Blockchain Indexer） 是一种允许开发者以快速且高效的方式查询和分析链上数据的工具。
> 它提取交易数据，将其转换为机器和人类可读的数据，并将其加载到数据库中以便于查询。
>
> 事实上，你无法直接在区块链中“搜索”数据。例如，假设你想知道某个账户在第 15364050 个区块时持有多少 USDC 代币。
> 你不能直接去 USDC 智能合约里查找，因为它并不存储历史数据。你必须提取从该区块至今发生的所有交易，进行过滤，
> 提取出所有与该账户相关的 USDC 转账信息，最后才能计算出答案。可以想象，这绝非一种理想的方法。这正是索引器发挥作用的地方。
> 它们维护着类似数据库的结构，让你能够立即针对所需的任何信息（包括历史信息）运行查询，并迅速获得结果。

其核心理念是：你只需要在链上存储核心数据和应用逻辑，以便任何人都能验证 DApp 是否在正确运行；其他任何内容都可以、也应该留在链下。

### IPFS

IPFS（星际文件系统） 是一种去中心化的、基于内容寻址的存储系统，它在点对点（P2P）网络中的节点之间分发存储对象。
内容寻址（Content-addressable） 意味着每一部分内容（文件）都会被计算哈希值，并使用该哈希值作为该文件的唯一标识。
随后，你可以通过该哈希值向任何 IPFS 节点请求并检索该文件。

IPFS 旨在取代 HTTP，成为 Web 应用分发协议的首选。与其将 Web 应用存储在单一服务器上，
不如将文件存储在 IPFS 中，并从任何 IPFS 节点进行检索。欲了解更多信息，请参阅 IPFS 官方文档。

### 默克尔树

一种有趣且频繁使用的方案是：利用默克尔树结构将数据保存在链下，而仅在链上存储其默克尔根（Merkle Root）。
这样，你就不必将所有数据都存储在智能合约中（那会耗费巨额 Gas 费），但依然能够在链上执行某种形式的验证。

> [!Note]
> 编者注：以下代码示例在特定的技术语境下使用了“白名单（whitelist）”一词。虽然该术语带有歧义和偏见色彩，
> 但在整个行业及其文档中仍被广泛使用。尽管我们高度重视包容性，但为了确保技术概念表达的清晰度，作者选择在此保留原词。

最常见的用例是当你创建一个 NFT 系列，并希望为不同的地址设置“白名单”，以便他们在公开销售之前以更低的价格铸造（Mint）这些 NFT。你有两种选择：

第一种选择是在智能合约内部创建一个存储变量，将每个地址映射（Map）到一个布尔值（Boolean），所有白名单地址对应的布尔值为 true。
随后，你可以使用这个映射来验证某个地址是否确实在白名单中。用户在提交铸造交易时无需提供任何额外信息；
合约只需检查 msg.sender 是否包含在白名单映射中，如下所示：
```Solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.28;

// import OpenZeppelin contracts.
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

// A simplified NFT contract with a whitelist mint function that uses a mapping to 
// store whitelisted addresses.
contract MyWhitelistNFT is ERC721, Ownable {
    // ... rest of the contract
    
    // mapping for whitelisted addresses
    mapping(address => bool) public isWhitelisted;
    
    /**
     * @notice Whitelist mint function.
     */
    function whitelistMint() payable {
        // ... rest of the function
        
        // check if the user is whitelisted.
        require(isWhitelisted[msg.sender], "You are not whitelisted");
        
        // mint the NFT.
        _safeMint(msg.sender, nextTokenId);
        nextTokenId++;
    }
}
```
第二种选择是在链下创建一个默克尔树（Merkle Tree），并将默克尔根（Merkle Root）存储在链上的合约中。
随后，你向每个白名单用户提供其对应的默克尔证明（Merkle Proof）。用户将该证明提交给合约，
合约则在链上验证该证明的有效性（通常通过调用一些成熟的库来实现），如下所示：

```Solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.28;

// import OpenZeppelin contracts.
import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/cryptography/MerkleProof.sol";

// A simplified NFT contract with a whitelist mint function that uses a merkle tree to 
// store whitelisted addresses.
contract MyWhitelistNFT is ERC721, Ownable {
    // ... rest of the contract
    
    // the merkle root of the off-chain generated Merkle Tree.
    bytes32 public merkleRoot;
    
    /**
     * @notice Whitelist mint function.
     * @dev User must provide a merkle proof to prove they are whitelisted.
     * @param _merkleProof The proof that msg.sender is whitelisted.
     */
    function whitelistMint(bytes32[] calldata _merkleProof) external payable {
        // ... rest of the function
        
        // verify that (msg.sender) is in the merkle tree using the provided proof.
        bytes32 leaf = keccak256(abi.encodePacked(msg.sender));
        bool isValidLeaf = MerkleProof.verify(_merkleProof, merkleRoot, leaf);
        require(isValidLeaf, "Invalid Merkle Proof: Not whitelisted");
               
        // mint the NFT.
        _safeMint(msg.sender, nextTokenId);
    }  
}
```

相较于第一种方案，第二种方案要廉价且高效得多，尤其是在白名单规模庞大的情况下。
> [!Note]
> 这种方法极大地降低了 Gas 成本，但除非默克尔树和证明过程是可审计的，否则会引入白名单创建者造假的潜在风险。
> 理想情况下，用于生成默克尔树的原始列表应当开源且公开可访问，以便任何人都能验证生成的默克尔根是否有效。


## 前端（Web 用户界面）

DApp 的前端是使用任何最知名的 Web2 框架（如 React、Angular 或 Vue）创建的。如果需要与以太坊链进行交互，
这种交互通常通过 viem 或 ethers.js 等库进行抽象化处理。

构建 DApp 前端的一种初级方案是：仅直接从链上读取数据来更新所有组件。例如，如果你需要显示某个账户持有的某些代币余额，
你可以查询区块链并获取返回结果，并针对每一个新区块重复此步骤。

这种初级方案的主要问题在于，你的网站会变得非常缓慢，用户与这样的前端交互会感到非常沮丧。
这就是为什么（正如我们之前提到的）开发者经常在他们的 DApp 架构中使用中心化数据存储组件，以尽量减少与链的交互：这让整个用户体验变得更好。
因此，你最终得到的前端通常依赖于某些中心化组件，但你始终可以通过在区块链上进行双重检查来验证所有信息的准确性。
最终，如果你不信任官方前端，或者单纯不喜欢它，你甚至可以为特定的 DApp 创建自己的替代前端。
![Figure 12-3](<./images/figure 12-3.png>)



## A Basic DApp Example












