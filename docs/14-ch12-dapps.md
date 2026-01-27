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



## 一个基础 DApp 示例

到目前为止，我们已经探讨了 DApp 背后的基本概念。现在，是时候卷起袖子，亲自动手构建一个 DApp 了。

你可以在网上找到许多教程来帮助你从零开始构建你的第一个以太坊 DApp，但我们强烈推荐 [Speedrun Ethereum](https://oreil.ly/Onygc)。这是快速学习并立即开始构建酷炫作品最有效的方式。为了提升你在以太坊上构建 DApp 的知识，我们建议你完成 Speedrun Ethereum 上的所有挑战，并加入 [BuidlGuidl 社区](https://buidlguidl.com/)。

在本节中，我们将构建一个非常基础的去中心化应用，类似于 DApp 界的“Hello World”。你不需要任何先前的经验；所需要的只是一台电脑和互联网连接。

### 安装要求

要学习本教程，你需要在电脑上安装 [node.js](http://node.js/) 和 [yarn](https://oreil.ly/dq_hw)。请参考官方网站进行下载和安装。我们将使用 [Scaffold-ETH 2](https://scaffoldeth.io/)，这是一个非常酷的工具，可以让你极其快速地搭建开发环境。

### 创建 DApp

让我们打开终端，运行以下命令：
```Bash
$ npx create-eth@latest
```
系统会要求输入项目名称。在本演示中，我们选择 "mastering-ethereum"：
```
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+ 
 | Create Scaffold-ETH 2 app | 
 +-+-+-+-+-+-+-+-+-+-+-+-+-+-+

? Your project name: mastering-ethereum
```
接着，它会询问你想使用哪种 Solidity 开发框架。这里我们选择 Hardhat，但如果你对 Foundry 更熟悉，也可以自由选择：
```
? What solidity framework do you want to use?
❯ hardhat
   foundry
  none
```
几秒钟后，你应该会看到“Congratulations（恭喜）”字样以及类似于以下的“Next steps（后续步骤）”提示：
```
  Congratulations! Your project has been scaffolded! 🎉

  Next steps:

  cd mastering-ethereum
  
        Start the local development node
  
        yarn chain
  
        In a new terminal window, deploy your contracts
  
        yarn deploy
  
    In a new terminal window, start the frontend
  
    yarn start
  
 Thanks for using Scaffold-ETH 2 🙏 , Happy Building!
```

### 启动区块链

现在，我们已经准备就绪，可以开始构建 DApp 了。首先进入项目文件夹：
```Bash
$ cd mastering-ethereum
```
在这里，如果你进入 packages/hardhat/contracts 目录，可以找到一个名为 YourContract.sol 的示例智能合约。contracts 文件夹是你存放 DApp 项目所需的所有智能合约的地方。

在 packages/nextjs 文件夹中，你会发现为 DApp 前端预先搭建好的 Next.js 框架结构。

由于这是一个非常基础的教程，我们不会从头开始编写任何合约，也不会修改前端。我们将使用默认设置，以快速演示常规的开发流程。

首先，你需要启动一条用于本地开发的区块链。事实上，尽管最终产品会使用部署在以太坊主网上的智能合约，但你不应该使用真实的链来构建和测试 DApp。
那会非常缓慢，而且会浪费大量的资金。Scaffold-ETH 提供了一个非常实用且简单的命令，可以立即启动一条新的本地开发链。你只需要运行：
```
$ yarn chain
```

### 部署你的合约

现在，你需要将合约部署到上一步搭建的本地链上。同样地，Scaffold-ETH 为此提供了一个简单的命令。请打开一个新的终端窗口并输入：
```
$ yarn deploy
```
你应该会看到类似这样的输出：
```
Generating typings for: 2 artifacts in dir: typechain-types for target: ethers-v6
Successfully generated 6 typings!
Compiled 2 Solidity files successfully (evm target: paris).
deploying "YourContract" (tx: 0x8ec9ba16869588c2826118a0043f63bc679a4e947f739e8032e911475e77dcb4)...: deployed at 0x5FbDB2315678afecb367f032d93F642f64180aa3 with 532743 gas
👋  Initial greeting: Building Unstoppable Apps!!!
📝  Updated TypeScript contract definition file on ../nextjs/contracts/deployedContracts.ts
```
正如你所见，该命令在本地链上部署了名为 YourContract 的示例智能合约。未来当你构建新的 DApp 时，你需要进入 `packages/hardhat/deploy` 目录并修改 `00_deploy_your_contract.ts` 文件，以便部署你实际需要的合约。

如果你回到之前运行 yarn chain 命令的那个终端，你会发现产生了一些新的日志，特别是类似下面这条：
```
eth_sendTransaction
  Contract deployment: <UnrecognizedContract>
  Contract address:    0x5fbdb2315678afecb367f032d93f642f64180aa3
  Transaction:         0x8ec9ba16869588c2826118a0043f63bc679a4e947f739e8032e911475e77dcb4
  From:                0xf39fd6e51aad88f6f4ce6ab8827279cfffb92266
  Value:               0 ETH
  Gas used:            532743 of 532743
  Block #1:            0xa7b8e3b6f82eccb3542279573dbf8efa2b876ff00807a8619feef191007e06d9
```
注意：请留意 Contract deployment（合约部署） 和 Contract address（合约地址） 这几行。这证明你已经成功地在本地链的特定合约地址上部署了你的合约。

### 启动前端

Scaffold-ETH 示例自带了一个基础的内置前端，让你可以立即通过图形界面与合约进行交互。请打开第三个终端窗口并输入：
```
$ yarn start
```
它应该会返回类似以下的内容：
```
yarn start
  ▲ Next.js 14.2.21
  - Local:        http://localhost:3000
 
 ✓ Starting...
 ✓ Ready in 1767ms
```
现在，复制 localhost 对应的 URL，打开浏览器并粘贴该链接。你应该能看到前端界面，正如图 12-4 所示。
![Figure 12-4](<./images/figure 12-4.png>)
图 12-4. Scaffold-ETH 前端界面

### 与你的合约交互

恭喜你，一切准备就绪！你现在可以开始体验并与你的 DApp 进行交互了。你会发现有两个非常实用的功能，它们对你的开发流程至关重要：
* 燃烧钱包 (Burner Wallets)
* 调试合约 (Debug Contracts) 部分

正如图 12-4 所示，在右上角，我们已经通过一个看起来随机且陌生的钱包连接到了网站。那是因为它是一个燃烧钱包（Burner Wallet）：
这是一个自动生成的地址，其私钥暂时保存在你的浏览器中。事实上，如果你尝试刷新页面，你会发现燃烧钱包的地址并没有改变。

燃烧钱包是开发流程中的一项“杀手锏”功能，因为你不需要每次都打开 Web3 钱包（如 MetaMask）并手动连接。当你准备好使用真实钱包时，
只需点击下拉菜单并选择 Disconnect（断开连接）；然后点击 Connect Wallet（连接钱包） 并从列表中选择你心仪的钱包，
正如图 12-5、12-6 和 12-7 所示。

![Figure 12-5](<./images/figure 12-5.png>)
图 12-5. 断开燃烧钱包
![Figure 12-6](<./images/figure 12-6.png>)
图 12-6. “连接钱包”按钮

![Figure 12-7](<./images/figure 12-7.png>)
图 12-7. 选择钱包

第二个杀手锏功能是 Debug Contracts（调试合约） 栏目。只需点击页面中心的“Debug Contracts”链接即可打开它。
在默认示例中，你应该会看到类似图 12-8 所示的内容。
![Figure 12-8](<./images/figure 12-8.png>)
图 12-8. 调试合约栏目

在这里，你无需构建任何前端界面，即可轻松地与所有合约进行交互。这在开发过程中非常有用，可以让你不断检查合约是否如预期般运行。

让我们进行一个小小的演示。首先，我们需要为我们的燃烧钱包注入资金，以便随后能发送交易与部署好的合约交互。为此，
你只需点击最右侧的按钮，正如图 12-9 所示。你几乎会立即收到一些 ETH，并看到你的 ETH 余额有所增加。

![Figure 12-9](<./images/figure 12-9.png>)
图 12-9. 获取资金（Grab funds）按钮

现在进入 setGreeting 栏目，在 _newGreeting 字段输入 hello world，在 payable value 字段输入 0.1。
接着，点击 payable value 字段右侧的星号按钮：
它会将 ETH 数值转换为等额的 wei 单位表示（ 1 ETH = 10¹⁸ wei ）。
最后，点击 Send（发送） 来提交你的交易，观察它是如何改变合约状态的。
图 12-10 展示了点击发送按钮前的合约状态。你可以看到合约目前持有 0 ETH，
问候语（greeting）是 "Building Unstoppable Apps!!!"，
溢价状态（premium）为 false，且计数器总数（totalCounter）等于 0。

![Figure 12-10](<./images/figure 12-10.png>)
图 12-10. 交易前的合约状态

图 12-11 捕捉了发送交易后的合约状态。你可以直观地看到，你的合约现在持有 0.1 ETH，问候语（greeting）已变为 "hello world"，溢价状态（premium）变为 true，且计数器总数（totalCounter）等于 1。
![Figure 12-11](<./images/figure 12-11.png>)
图 12-11. 交易后的合约状态

你可以尽情尝试各种操作，观察合约如何根据你的输入和行为做出响应。

### 部署到 Vercel

当你对自己的去中心化应用感到满意时，就可以将其发布到生产环境，例如 Vercel。Vercel 是一款非常实用的“前端即服务（frontend-as-a-service）”工具，让你能够轻松地将应用部署到互联网上。你还可以绑定自己购买的自定义域名，这样人们只需输入域名即可访问你的 DApp。

Scaffold-ETH 再次通过提供一条简单的命令来协助你，实现将 DApp 立即部署到 Vercel。打开终端并输入：
```
$ yarn vercel:yolo
```
你需要关联你的 Vercel 账户（如果没有，则创建一个新账户）并为你的项目选择一个名称——仅此而已。几分钟后，你的整个 DApp 就会部署到 Vercel 上，全世界的任何人都可以去尝试运行它。

如果你进入 Vercel 的个人主页，现在就可以看到新创建的项目。正如图 12-12 所示，在 Domains（域名）字段中，你可以找到 Vercel 为你自动生成的网站域名。
![Figure 12-12](<./images/figure 12-12.png>)
图 12-12. Vercel 项目页

## 进一步实现 DApp 的去中心化

在上一节中，我们将 DApp 部署到了 Vercel。虽然 Vercel 是目前托管 DApp 最流行的解决方案之一，但它并不是一项去中心化服务。所有内容都存储在其私有服务器上，这意味着 Vercel 可能会根据其政策审查任何用户，并追踪每个与你的 DApp 交互的人的 IP 地址。

为了进一步提升 DApp 的去中心化程度，我们可以将前端托管在 IPFS（星际文件系统）等解决方案上，这是一个全球性的点对点（P2P）节点网络。此外，一个值得关注的新兴服务是 eth.limo，它旨在将主流网站的用户体验（通常由中心化平台提供）与 IPFS 等技术所提供的稳健性和去中心化特性相结合。

### 去中心化网站

[Eth.limo](https://eth.limo/) 是创建更优质的去中心化网站（也称为 DWebsites）过程中缺失的关键一环，它让你可以像访问传统应用一样访问这些网站。它基于 ENS（以太坊域名服务） 技术，该技术通过 .eth 域名使以太坊地址变得更加用户友好。以太坊联合创始人之一 Vitalik Buterin 就使用 ENS 将其钱包地址 `0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045` 与易于记忆的名称 `vitalik.eth` 绑定在一起。

ENS 的功能不仅限于将域名与以太坊地址链接：它还可以解析为 IPFS 网站（实际上是内容哈希），例如 `bafybeif3dy…ga4zu`。与 IPFS 网站相关的主要问题是，大多数主流浏览器无法正确解析它们并向用户展示内容。这正是 eth.limo 发挥作用的地方：它为 ENS 名称和 IPFS 内容运行一个反向代理。它会捕获所有对 `*.eth.limo`（基本上是所有以 `.eth.limo` 结尾的网站）的请求，自动解析所请求 ENS 记录中的 IPFS 内容哈希（contenthash），并通过 HTTPS 返回相应的静态内容。

例如，当你访问 [Vitalik Buterin 的官方博客](https://vitalik.eth.limo/)时，幕后发生的逻辑如下：
1. Eth.limo 监测到发往 vitalik.eth ENS 域名的传入请求。
2. 它将其解析为包含该网站主页的 IPFS 内容哈希（这是 Buterin 预先设置好的）。
3. 它通过 HTTPS 返回相应的静态内容。

通过这种方式，世界上任何拥有浏览器的人都可以轻松访问存储在 IPFS 上的内容，且无需进行任何复杂的配置或设置。

### 局限性

尽管去中心化网站（DWebsites）技术——特别是 eth.limo——正在迅速演进和完善，但在编写本章时（2025 年 6 月），仍存在一些局限性。首先，IPFS 只能处理静态文件，因此无法执行任何形式的服务端计算。此外，要使用 eth.limo，你需要购买一个 ENS 域名并将其链接到前端的 IPFS 内容哈希上。而且你必须始终使用 *.eth.limo 这个自定义域名；你不能去掉末尾的 .limo 部分，否则你的 Web 浏览器将无法将 ENS 名称解析为 DApp 的 IPFS 前端。这可能也是大多数 DApp 目前尚未采用 `eth.limo` 的原因。

> [!Tip]
> 尽管大多数 Web 浏览器目前还不兼容 ENS 和 IPFS，但某些浏览器已经开始增加对它们的支持，[Brave 浏览器](https://oreil.ly/LlJT7)就是一个典型例子。

必须指出的是，eth.limo 也是一个潜在的中心化第三方，可能会在没有任何通知的情况下停止工作。如果发生这种情况，你的 DApp 仍然可以通过 IPFS 访问，但 .eth.limo 这个 URL 将无法再将用户重定向到这些应用。

如果你对此感兴趣并想深入研究这种构建完全去中心化网站的方案，可以在其[官方网站](https://eth.limo/)上找到更多详细信息。

### 部署到 IPFS

Scaffold-ETH 2 提供了另一个简单的命令，让你可以快速地将 DApp 推送到 IPFS。你只需打开一个新的终端并输入：
```
$ yarn ipfs
```
大功告成！你应该会看到类似这样的输出
```
   Creating an optimized production build ...
 
 ✓ Compiled successfully
 ✓ Linting and checking validity of types
    
 ✓ Collecting page data
    
 ✓ Generating static pages (8/8)
 ✓ Collecting build traces
    
 ✓ Finalizing page optimization

…

🚀  Upload complete! Your site is now available at: https://community.bgipfs.com/ipfs/bafybei…
```
如果你访问显示的网址，你会发现你的 DApp 运行良好，其前端托管在 IPFS 上。字符串 "bafy…" 就是 IPFS 内容哈希（contenthash）。如果你拥有个人 ENS 域名并希望将其重定向到这个由 IPFS 托管的站点，你仍需要配置 eth.limo。

以下是 yarn ipfs 命令在幕后实际执行的操作：
1. 构建前端：将前端代码进行编译打包，生成准备上传至 IPFS 的静态文件。
2. 上传文件：通过 BuidlGuidl（Scaffold-ETH 2 的维护团队）的 IPFS 社区节点将静态文件上传至 IPFS。
3. 返回 URL：返回一个重定向至这些静态文件的 URL，该 URL充当了 IPFS 内容的反向代理（格式为 `community.bgipfs.com/<ipfs-内容哈希>`）。

如果你想学习如何运行自己的 IPFS 节点并进行集群固定（pin a cluster），可以参考 [BuidlGuidl IPFS](https://www.bgipfs.com/)。

## 从应用（App）到去中心化应用（DApp）

在过去的几个章节中，我们逐步构建了一个去中心化应用。我们利用 Scaffold-ETH 这一工具极大地简化了开发工作流：从启动本地区块链开始，到部署合约，再到启动本地前端以立即进行交互测试。随后，我们将 DApp 的前端发布到了 Vercel，展示了在生产级环境中部署 DApp 是多么简单。最后，我们探索了如何通过将前端发布到 IPFS，并结合 ENS 和 eth.limo 等解决方案来进一步实现去中心化，从而让任何人无需安装特殊应用即可访问它。

图 12-13 简明地概述了创建一个完全去中心化应用所需的工程技术栈。
![Figure 12-13](<./images/figure 12-13.png>)
图 12-13. DApp 完整工程技术栈简明概述

## 结语
在本章中，我们探索了如何利用现代开发工具来简化工作流，并从零开始构建一个基础的 DApp。在下一章中，我们将深入研究以太坊上一些最重要的 DApp 及其分类，它们共同构成了众所周知的去中心化金融（DeFi）。
