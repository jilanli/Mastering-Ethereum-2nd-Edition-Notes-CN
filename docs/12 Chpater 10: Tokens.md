# 第十章：代币

“Token”（代币）一词源于古英语中的 tācen[^1]，意为符号或象征。它通常被用来指代由私人发行、具有特殊用途、且本身内在价值微乎其微的类硬币物品，例如交通代币（车票）、洗衣代币或游戏机代币。如今，在区块链上运行的代币正在重新定义这个词：它们是基于区块链的抽象概念，可以被所有，并代表资产、货币或访问权限。

“代币”一词与“价值微不足道”之间的联系，在很大程度上与物理代币的有限用途有关。物理代币通常局限于特定的商家、组织或地点，不易交换，且通常只有单一功能。而通过区块链代币，这些限制被解除，或者更准确地说，被完全重新定义了。许多区块链代币在全球范围内具有多种用途，并可以在全球流动性市场中互相交易，或兑换成其他货币。随着使用和所有权限制的消失，对其“价值微不足道”的预期也成为了过去。

在本章中，我们将研究代币的各种用途及其创建方式。我们还将讨论代币的属性，如同质化（Fungibility）和内在性（Intrinsicality）。最后，我们将研究它们所基于的标准和技术，并尝试构建我们自己的代币。

## 代币的用途

代币最显而易见的用途是作为数字私人货币。然而，这仅仅是其可能的用途之一。代币可以通过编程来实现许多不同的功能，而且这些功能往往相互重叠。例如，一个代币可以同时传达投票权、访问权和资源所有权。如下表所示，货币仅仅是代币的第一个“应用”：

* 货币（Currency）：代币可以作为一种货币形式，其价值通过私人贸易决定。

* 资源（Resource）：代币可以代表在分享经济或资源共享环境中赚取或生产的资源——例如，代表可在网络上共享的存储或 CPU 资源的代币。

* 资产（Asset）：代币可以代表对内在或外在、有形或无形资产的所有权——例如黄金、房地产、汽车、石油、能源、大型多人在线游戏（MMOG）物品等。

* 访问权（Access）：代币可以代表访问权限，并允许进入数字或物理财产，如讨论论坛、专属网站、酒店房间或租赁车辆。

* 股权（Equity）：代币可以代表数字组织（如 DAO）或法律实体（如公司）中的股东权益。

* 投票（Voting）：代币可以代表数字或法律系统中的投票权。

* 收藏品（Collectible）：代币可以代表数字收藏品（如 CryptoPunks）或实物收藏品（如画作）。

* 身份（Identity）：代币可以代表数字身份（如虚拟头像）或法律身份（如国民身份证）。

* 证明（Attestation）：代币可以代表由某些机构或去中心化信誉系统出具的事实认证或证明（例如结婚记录、出生证明或大学学位）。

* 实用性（Utility）：代币可以用于访问服务或为服务付费。

通常，单个代币会涵盖其中的多项功能。有时很难将它们区分开来，因为在物理世界中，它们的等价物总是密不可分的。
例如，在物理世界中，驾照（证明）同时也是身份证明文件（身份），两者无法分离。但在数字领域，此前交织在一起的功能可以被分离并独立开发（例如，匿名证明）。

## 代币与同质化

[维基百科](https://oreil.ly/ge7zP)指出：“在经济学中，同质化（Fungibility）是指一种商品或货物的属性，其个体单位在本质上是可以互相互换的。”当我们可以将代币的任何单一单位替换为另一单位，且其价值或功能没有任何区别时，这种代币就是同质化的。

非同质化代币（NFTs） 是指每一个都代表唯一的有形或无形物品的代币，因此它们是不可互换的。例如，一个代表特定梵高画作所有权的代币，并不等同于另一个代表毕加索画作的代币，尽管它们可能属于同一个“艺术品所有权代币”系统。
同样，代表特定数字收藏品的代币（如某只特定的 CryptoKitty 加密猫），也无法与任何其他 CryptoKitty 互换。每个 NFT 都关联着一个唯一的标识符，例如序列号。

我们将在本章稍后部分看到同质化和非同质化代币的示例。
> [!Note]
> 请注意，“Fungible”一词在日常生活中常被误用来表示“可直接兑换成金钱”（例如，赌场代币可以“兑现”，而洗衣代币通常不行）。这不是我们在此处使用该词的含义。

## 对手方风险

对手方风险是指交易中的另一方未能履行其义务的风险。某些类型的交易会面临额外的对手方风险，因为其中涉及两个以上的参与方。
例如，如果你持有一份贵金属的存款证明并将其卖给他人，那么该交易中至少存在三个参与方：卖家、买家以及贵金属的托管人。由于有人持有实物资产，他们必然会成为交易履行的一方，并为任何涉及该资产的交易增加对手方风险。

通常，当资产通过所有权代币进行间接交易时，资产托管人会带来额外的对手方风险。他们真的持有该资产吗？
他们是否会承认（或允许）基于代币（如凭证、契据、产权证或数字代币）转移而发生的所有权变更？在代表资产的数字代币领域，与非数字世界一样，了解谁持有代币所代表的资产，以及适用于该底层资产的规则是什么，都至关重要。

## 代币与内在性

“Intrinsic”（内在的）一词源于拉丁语的 intra，意为“来自内部”。有些代币代表的是区块链内部固有的数字物品。这些数字资产与代币本身一样，都受共识规则的管辖。这产生了一个重要的影响：代表内在资产的代币不承担额外的对手方风险。
如果你持有一个 CryptoKitty 的私钥，并没有任何其他方为你代管这只加密猫——你直接拥有它。区块链共识规则适用，你对私钥的所有权（即控制权）等同于对该资产的所有权，无需任何中间人。

相反，许多代币被用来代表外在（Extrinsic）事物，例如房地产、公司投票股、商标、金条和债券。这些物品并不存在于区块链“内部”，其所有权受法律、习俗和政策的管辖，与管辖代币的共识规则相互独立。
换句话说，代币发行者和持有者可能仍需依赖现实世界的非智能合约。结果是，这些外在资产具有额外的对手方风险，因为它们由托管人持有、记录在外部登记机构中，或受区块链环境之外的法律和政策控制。

基于区块链的代币最重要的影响之一，就是能够将外在资产转化为内在资产，从而消除对手方风险。一个典型的例子是从公司股权（外在）转向 DAO（去中心化自治组织）或类似组织中的股权或投票代币（内在）。
稳定币是另一个例子，它们作为锚定法定货币的区块链代币，由国库券和现金储备等外在资产提供支持。

## 实用、股权，还是圈钱陷阱？

几乎每一个以太坊项目在启动时似乎都会发行某种代币。但所有这些项目真的都需要代币吗？“将万物代币化”的口号听起来很响亮，但现实要复杂得多。
代币可以是组织和激励社区的强大工具，但它们也已成为投机和炒作的代名词。

理论上，代币主要有两个用途。首先是实用型代币（Utility Tokens）。它们旨在提供对特定生态系统内服务或资源的访问权。例如，某种代币可能代表去中心化网络上的存储空间，
或者是 DApp（去中心化应用）中高级功能的访问权限。在这种情况下，代币的价值与其在平台内的功能挂钩。其次是股权型代币（Equity Tokens），其功能理应类似于公司的股票。这些代币可以代表对项目的所有权或控制权，例如 DAO 中的投票权或利润分成。

然而在实践中，这两类代币的界限往往非常模糊。许多实用型代币在很大程度上仍具有投机性，用户持有它们更多是将其作为资产，而非访问凭证。类似于股权的代币虽然可能赋予治理权，
但往往缺乏确保成员有效参与的机制。有些项目将代币深度整合到其经济模型中，但这只是例外，而非普遍现象。

这就引出了一个问题：代币本质上是坏的吗？并非如此。代币在创建和激励社区，或驱动 DAO 的去中心化治理方面可以非常有效。但现实情况是，大多数代币在推出时，
获利而非实用性才是主要驱动力。如果你正考虑发行或投资一种代币，值得问几个尖锐的问题：该代币在协议中是否真的起到了必不可少的作用，还是仅仅是一个筹款工具？如果没有它，项目是否也能运行得一样好？诚实地回答这些问题，可以帮助你区分真正的创新与营销驱动的炒作。

显然，代币领域仍在不断演变。代币本身并无好坏之分，其价值取决于其设计和实现方式。挑战在于如何将有意义的事物与毫无意义的事物区分开来，并抵御下一个“模因币”（Meme Coin）的诱惑。

> [!Note]
> 在撰写本章期间（2025 年 1 月），新当选的总统唐纳德·特朗普推出了他自己的模因币（Meme coin），该代币在一天之内市值就达到了 150 亿美元。此前许多人曾期待他在任期内会推出有利的加密货币政策，
> 然而，我们等来的却是一个模因币。这一事件突显了主导加密货币领域的投机狂热——在这种环境下，炒作往往战胜了实质内容。

## 它就是一只鸭子！

长期以来，代币一直是初创公司最青睐的融资工具。它们许诺创新、去中心化，有时甚至是彻底的财务自由。但问题在于：在大多数司法管辖区，向公众发行证券是一项受监管的活动，而代币很容易跨越红线进入证券领域。
多年来，各项目一直试图通过将自己的代币贴上“实用型代币”的标签来规避监管，声称它们只是对未来服务访问权的预售。其逻辑是：如果代币不是股权，它就不是证券。但正如那句老话所说：“如果它走起来像鸭子，叫起来也像鸭子，那它就是一只鸭子。” 监管机构，尤其是美国证券交易委员会（SEC），正盯紧了这些“鸭子”。

在过去的几年里，SEC 对代币发行采取了日益激进的立场，打击那些试图在实用性与股权之间“横跳”的项目。例如，2020 年，SEC 起诉了 Ripple Labs 及其发行的 XRP 代币，指控其为未经注册的证券。
Ripple 辩称 XRP 是货币而非证券，但法院的后续裁决显示了这类案件的复杂程度。

即使是以太坊本身也未能免于审查。早在 2018 年，前 SEC 官员曾宣布以太币已经“足够去中心化”，因此不属于证券。但就在 2024 年，SEC 暗示以太坊向权益证明（PoS）的转型可能会让它重新回到显微镜下。
为什么？因为质押奖励（Staking rewards）类似于红利，而红利是证券的核心特征。这些进展表明，监管环境已变得多么反复无常且难以预测。

令人着迷——同时也令人沮丧——的是，这些案件往往演变成了语义之争。项目方坚称其代币是实用工具，就像服务的门票。但如果购买者的主要动机是投机，SEC 就会将其视为证券，简单明了。
挑战在于，用于做出这些判定的法律框架早在区块链技术出现之前就已存在。诞生于 20 世纪 40 年代的“豪威测试”（Howey Test）旨在定义投资合同，并非为去中心化网络或可编程资产而设计。因此，将其应用于加密项目并不总是那么直接。创新者想要筹集资金并建立社区，而监管机构则想要保护投资者免受误导。其结果便是：一场又一场法庭大戏不断上演，数亿资金和整个生态系统的命运悬于一线。

## 以太坊上的代币

区块链代币早在以太坊之前就已存在。从某种意义上说，第一种区块链货币——比特币本身就是一种代币。在以太坊出现之前，许多代币平台也已在比特币和其他加密货币之上开发出来。然而，以太坊上第一个代币标准的引入，才真正引发了代币的爆炸式增长。

维塔利克·布特林（Vitalik Buterin）曾建议，代币是像以太坊这样通用的可编程区块链最显而易见且最有用的应用之一。事实上，在以太坊运行的第一年里，经常能看到维塔利克和其他人穿着印有以太坊 Logo 的 T 恤，
背后则是智能合约的示例代码。这些 T 恤有几个版本，但最常见的一款展示的就是代币的实现代码。

在我们深入研究如何在以太坊上创建代币的细节之前，了解代币在以太坊上是如何运行的宏观图景至关重要。代币与以太币（Ether）不同，因为以太坊协议本身对代币一无所知。 
发送以太币是以太坊平台的内在行为，但发送甚至拥有代币则不然。以太坊账户的以太币余额是在协议层处理的，而以太坊账户的代币余额则是在智能合约层处理的。

要在以太坊上创建一种新代币，你必须创建一个新的智能合约。一旦合约部署完成，它将处理包括所有权、转账和访问权限在内的一切事务。你可以按照任何你想要的方式编写智能合约来执行所有必要的操作，但遵循现有标准可能是最明智的做法。接下来我们将探讨这些标准。


## ERC-20 代币标准

第一个代币标准由 Fabian Vogelsteller 于 2015 年 11 月作为一项 ERC（以太坊意见征求稿）提出。该提案被自动分配了 GitHub 问题编号 20，因此得名“ERC-20 代币”。
目前绝大多数代币都基于 ERC-20 标准。虽然该提案最终成为了 EIP-20（以太坊改进提案），但人们大多仍沿用其最初的名称：ERC-20。

ERC-20 是同质化代币的标准，这意味着 ERC-20 代币的不同单位是可以互换的，且不具备唯一属性。ERC-20 标准为实现代币的合约定义了一个通用接口，
使得任何兼容该标准的代币都可以通过相同的方式被访问和使用。该接口由一系列必须在标准实现中存在的函数组成，同时也包含了一些开发者可以自行添加的可选函数和属性。

### ERC-20 强制性函数与事件

一个符合 ERC-20 标准的代币合约必须至少提供以下函数和事件：

**totalSupply**

返回当前存在的代币总单位数。ERC-20 代币的供应量可以是固定的，也可以是可变的。

**balanceOf**

给定一个地址，返回该地址拥有的代币余额。

**transfer**

给定地址和金额，从执行转账的地址余额中向目标地址转入相应数量的代币。

**transferFrom**

给定发送者、接收者和金额，将代币从一个账户转移到另一个账户。该函数通常与 approve 配合使用。

**approve**

给定接收地址和金额，授权该地址从授权账户中多次执行转账，总额不超过设定的金额。

**allowance**

给定所有者地址和支出者（spender）地址，返回支出者仍被授权从所有者处提取的余额。

**Transfer（事件）**

在转账成功（调用 transfer 或 transferFrom）后触发，即使转账金额为零也应触发。

**Approval（事件）**

在成功调用 approve 后记录的事件。

#### ERC-20 可选函数

除了上一节列出的强制性函数外，该标准还定义了以下可选函数：

**name**

返回代币的人类可读名称（例如：“US dollars”）。

**symbol**

返回代币的人类可读符号（例如：“USD”）。

**decimals**

返回用于划分代币金额的小数位数。例如，如果小数位数为 2，那么 1,000 的代币单位量实际上代表 10.00 的余额。

#### 用 Solidity 定义的 ERC-20 接口

以下是 ERC-20 接口规范在 Solidity 中的样子：
```Solidity
contract ERC20 {
   function totalSupply() public view returns (uint256 theTotalSupply);
   function balanceOf(address _owner) public view returns (uint256 balance);
   function transfer(address _to, uint256 _value) public returns (bool success);
   function transferFrom(address _from, address _to, uint256 _value) public returns
      (bool success);
   function approve(address _spender, uint256 _value) public returns (bool success);
   function allowance(address _owner, address _spender) public view returns
      (uint256 remaining);
   event Transfer(address indexed _from, address indexed _to, uint256 _value);
   event Approval(address indexed _owner, address indexed _spender, uint256 _value);
}
```

#### ERC-20 数据结构

如果你去研究任何一个 ERC-20 的实现方案，你会发现它包含两个核心数据结构：一个用于追踪余额（Balances），另一个用于追踪授权额度（Allowances）。
在 Solidity 中，它们是通过**数据映射**（Mapping）实现的。

第一种数据映射实现了一张按所有者分类的代币余额内部表。这使得代币合约能够记录谁拥有这些代币。每一次转账本质上都是从一个余额中扣除，并增加到另一个余额中：
```Solidity
mapping(address account => uint256) _balances;
```
第二种数据结构是授权额度的映射。正如我们在下一节将看到的，对于 ERC-20 代币，所有者可以向支出者（Spender）委派权限，
允许他们从所有者的余额中支出特定金额（即授权额度）。ERC-20 合约使用一个二维映射来记录这些授权额度，其主键是代币所有者的地址，映射到支出者地址及相应的授权金额：
```Solidity
mapping(address account => mapping(address spender => uint256)) public _allowances;
```

#### ERC-20 工作流程：“Transfer” 与 “Approve 和 TransferFrom”

ERC-20 代币标准拥有两个转账函数。你可能会纳闷这是为什么。

ERC-20 允许两种不同的工作流程。第一种是使用 transfer 函数的直观单笔交易流程。钱包向其他钱包发送代币时，使用的正是这种流程。绝大多数代币交易都是通过 transfer 流程完成的。

执行 transfer 合约非常简单：如果 Alice 想给 Bob 发送 10 个代币，她的钱包会向代币合约地址发送一笔交易，调用 transfer 函数，并将 Bob 的地址和数值 10 作为参数传入。
代币合约随后调整 Alice 的余额（-10）和 Bob 的余额（+10），并发布一个 Transfer 事件。

第二种是先后使用 approve 和 transferFrom 的两笔交易流程。这种流程允许代币所有者将控制权委派给另一个地址。它最常用于将控制权委派给某个合约以进行代币分发，但也可以被交易所使用。
例如，如果一家公司正在进行 ICO 售卖代币，他们可以向一个众筹（Crowdsale）合约地址进行 approve（授权），允许其分发特定数量的代币。随后，众筹合约就可以执行 transferFrom，
将代币合约所有者的余额转移给每一位代币买家。就像图 10-1 中展示的。
![Figure 10-1](<./images/figure 10-1.png>)
图 10-1：ERC-20 代币的“Approve”与“TransferFrom”两步工作流

对于 approve 和 transferFrom 工作流，需要执行两笔交易。假设 Alice 想要允许 AliceICO 合约将总供应量 50% 的 AliceCoin 代币出售给像 Bob 和 Charlie 这样的买家。
首先，Alice 部署 AliceCoin ERC-20 合约，并将所有代币发行到她自己的地址。接着，Alice 部署可以由以太币（Ether）换取代币的 AliceICO 合约。
下一步，Alice 启动 approve 和 transferFrom 流程。她向 AliceCoin 合约发送一笔交易，调用 approve 函数，参数为 AliceICO 合约的地址以及总供应量的 50%。
这将触发 Approval 事件。现在，AliceICO 合约便获得了出售 AliceCoin 的权限。

当 AliceICO 合约从 Bob 那里收到以太币时，它需要向 Bob 回传相应数量的 AliceCoin。在 AliceICO 合约内部设定了 AliceCoin 与以太币之间的汇率。Alice 在创建 AliceICO 合约时设定的汇率，
决定了 Bob 发送一定量以太币后能获得多少代币。当 AliceICO 合约调用 AliceCoin 的 transferFrom 函数时，它将 Alice 的地址设为发送方（sender），将 Bob 的地址设为接收方（recipient），
并根据汇率计算出要转移给 Bob 的 AliceCoin 数量填入 value 字段。随后，AliceCoin 合约将余额从 Alice 的地址转移到 Bob 的地址，并触发 Transfer 事件。
只要不超过 Alice 设定的授权限额，AliceICO 合约可以无限次调用 transferFrom。AliceICO 合约可以通过调用 allowance 函数来实时追踪它还能出售多少 AliceCoin。

#### ERC-2612：通过 “Permit” 实现的无 Gas 转账

在第 9 章中，我们详细探讨了 ERC-20 代币传统的 transfer 和 transferFrom 流程。虽然这些方法一直是代币转账的中流砥柱，但它们并非没有局限性。
两者都要求发送者直接与区块链交互，这意味着他们手头必须拥有一些原生加密货币（如 ETH）来支付 Gas 费。这造成了一个巨大的障碍，尤其是当代币被发送到一个没有任何原生资金的新地址时。这种体验令人沮丧，远非理想。

这正是 ERC-2612 大显身手的地方。它是对 ERC-20 代币标准的一个巧妙补充，允许用户在无需亲自接触区块链的情况下批准代币转移。
它的工作原理是这样的：你不再需要发送链上交易来授权（Approve）转账，而是只需使用钱包对必要的数据（如接收者地址、代币数量、过期时间和 Nonce 值）进行签名。
这会生成一个签名，而任何需要执行转账的人（无论是接收者还是第三方）都可以将该签名提交给代币合约的 permit 方法。合约读取并验证签名，随后处理授权——这一切都无需你为初始步骤支付 Gas 费。
它既高效又安全，大大减少了流程中的麻烦。

要实现 ERC-2612，代币开发者需要在其 ERC-20 合约中扩展此功能。一旦部署，它将为用户带来两大核心收益：首先，它简化了整个流程。
用户只需一个签名即可授予权限，而无需逐一授权每一笔转账。其次，由于减少了所需的交易次数，它节省了 Gas 成本。

#### ERC-20 的实现方案

虽然可以用大约 30 行 Solidity 代码实现一个兼容 ERC-20 的代币，但大多数实际方案要复杂得多。这是为了应对潜在的安全漏洞。
EIP-20 标准提到了两种实现方案，分别由 Consensys 和 OpenZeppelin 开发。Consensys 的 EIP-20 代币自 2018 年起已停止维护，
而 [OpenZeppelin 的 ERC-20 代币](https://oreil.ly/7d7RD) 则成为了开发者的事实标准，并一直保持活跃维护。
该实现方案构成了 OpenZeppelin 库的基础，用于实现更复杂的兼容 ERC-20 的代币，例如带有融资上限、代币化金库、归属计划（Vesting）等功能的代币。

### 发行我们自己的 ERC-20 代币

让我们来创建并发行属于我们自己的代币。在本例中，我们将使用 Foundry 框架。本示例假设你已经安装了 Foundry 并完成了相关配置，且熟悉其基本操作。

我们将把我们的代币命名为 “Mastering Ethereum Token”，符号为 MET。首先，让我们使用以下命令创建并初始化一个 Foundry 项目目录：
```Bash
$ mkdir METoken
$ cd METoken
$ forge init
```
完成上述初始化命令后，你的项目应该呈现如下目录结构。
```
METoken/
├── foundry.toml
├── lib
│   └── forge-std
│       └── ...
├── README.md
├── script
│   └── Counter.s.sol
├── src
│   └── Counter.sol
└── test
    └── Counter.t.sol
```
Counter 是 Foundry 的默认示例合约，自带相关的测试和部署脚本。我们将删除所有与之相关的文件，为我们的代币合约腾出空间。

在本例中，我们将导入 OpenZeppelin 库，它是基于 Solidity 开发代币的行业标准：
```Bash
$ forge install OpenZeppelin/openzeppelin-contracts
[...]
Installed openzeppelin-contracts v5.2.0
```
在 METoken/lib/openzeppelin-contracts/contracts 目录下，我们现在可以看到所有的 OpenZeppelin 合约。虽然 OpenZeppelin 库包含的内容远超 ERC-20 代币标准，但我们只会使用其中的一小部分。

接下来，让我们编写代币合约。创建一个新文件 METoken.sol，并复制示例 10-1 中的代码。我们的合约非常简单，因为它从 OpenZeppelin 库中继承了所有功能。
示例 10-1：METoken.sol —— 一个实现 ERC-20 代币的 Solidity 合约
```Solidity
pragma solidity 0.8.28;
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
contract METoken is ERC20 {
    constructor(uint256 initialSupply) ERC20("METoken", "MET") {
        _mint(msg.sender, initialSupply);
    }
}
```
在这里，我们将 "METoken" 和 "MET" 作为名称和符号传递给 ERC-20 合约的构造函数。代币的**初始供应量**（Initial Supply）在部署期间作为构造函数参数提供，
并会被发送给该代币合约的部署者（msg.sender）。我们使用了默认的 decimals（小数位数）值 18，这是 ERC-20 代币广泛采用的标准。

现在我们可以使用 Foundry 来编译 METoken 合约：
```Bash
$ forge build
[⠊] Compiling...
[⠒] Compiling 6 files with Solc 0.8.28
[⠢] Solc 0.8.28 finished in 36.18ms
Compiler run successful!
```
让我们设置一个部署脚本，将 METoken 合约发布到区块链上。在 METoken/script 文件夹中创建一个新文件 METokenDeploy.s.sol，并复制以下代码：

```Solidity
pragma solidity 0.8.28;
import {Script, console} from "forge-std/Script.sol";
import {METoken} from "../src/METoken.sol";
contract METokenDeployer is Script {
    METoken public _METoken;
    function run() public {
        vm.startBroadcast();
        _METoken = new METoken(50_000_000e18);
        vm.stopBroadcast();
    }
}
```
在这个例子中，我们传入了 5000 万作为初始供应量。你注意到我们乘以了 1e18 吗？这就是代币的小数位数（decimals）。
请记住，为了获得 X 个代币的余额，我们需要输入的代币数量为 $X \times 10^{decimals}$。

> [!Note]
> 脚本文件的后缀 .s.sol 是 Foundry 的一项命名约定，用于快速识别文件的用途。这并不是强制性的要求——只需将脚本文件存放在 script 文件夹内即可——但这是一个非常好的实践，
> 会在开发过程中带来很多便利。同样，带有 .t.sol 后缀的测试文件也适用这一约定。

在我们部署到以太坊测试网络之前，先启动一个本地区块链来测试一切。我们将使用 Foundry 工具箱中的另一个工具：Anvil，一个本地以太坊开发节点。

要使用它，只需打开一个新的终端窗口并输入 anvil。控制台将显示一系列可用账户及其私钥、链 ID（Chain ID）、RPC URL 等信息。
RPC URL 是允许 Foundry（或任何以太坊客户端）与我们的本地区块链节点通信的端点，从而实现交易、合约部署和数据查询。Anvil 默认的 RPC 地址是 `http://127.0.0.1:8545`。
为了告诉部署脚本在本地链上进行部署，我们需要通过控制台参数提供该 URL，并带上标志位 `--rpc-url "http://127.0.0.1:8545"`。

最后我们需要的是部署者账户的私钥。由于这是一个本地区块链，我们在以太坊主网或测试网使用的地址在这里不会有任何资金，而且我们也不希望非必要地暴露真实的私钥。相反，我们将使用 Anvil 启动时生成的测试账户。这些账户在本地链上预装了 10,000 ETH，非常适合开发和测试。

我们已经准备好通过运行以下命令来部署代币：

```Bash
$ forge script script/METokenDeploy.s.sol --broadcast --rpc-url "http://127.0.0.1:8545"
--private-key <DEPLOYER_PRIVATE_KEY>
[⠊] Compiling...
No files changed, compilation skipped
Script ran successfully.
```
> [!Note]
> 你可能想知道 --broadcast 标志是干什么用的。它是用来告诉 Foundry 真正将交易广播到区块链上。
> 如果没有这个标志，Foundry 只会模拟执行（Simulate）该交易，而不会产生真实的链上变更。

控制台输出告知我们部署脚本运行成功。如果我们查看运行 Anvil 的那个终端窗口，会发现大量的活动日志，其中就包括我们的合约创建信息：
```
   Transaction: 0xd01e3a90e1f2ee60112658e92f4ebf04c24df67d2ec1315cfb79d145729d15ec
    Contract created: 0x5FbDB2315678afecb367f032d93F642f64180aa3
    Gas used: 941861
    Block Number: 1
    Block Hash: 0x748b6058dea932317cacf45bb63be82f253554f359b97ace224e35979a92b00a
    Block Time: "Fri, 31 Jan 2025 19:10:42 +0000"
```
我们的 METoken 已成功部署到以下地址：
```
0x5FbDB2315678afecb367f032d93F642f64180aa3
```
或者，我们也可以直接使用 Forge 的 create 控制台命令来部署代币：
```Bash
$ forge create METoken --broadcast --rpc-url http://127.0.0.1:8545 --private-key
<DEPLOYER_PRIVATE_KEY> --constructor-args 50000000000000000000000000
```
在这里，总供应量作为构造函数参数传入，并且已经将小数位数（decimals）计算在内。

#### 与 METoken 进行交互

我们可以通过多种方式与合约进行交互。我们可以使用 Remix（如第 2 章所述）、像 Foundry 的 Chisel 那样的 Solidity REPL（交互式解析器），或者像 ethers.js 这样的 JavaScript 库。我们还可以使用 Foundry 脚本来执行交易，这正是我们在本例中要采用的方式。

以太坊地址是 40 位的十六进制字符串，这并不直观易读。为了让示例更清晰，我们将为正在使用的两个地址分配昵称：Deployer（部署者）代表部署 MET 合约的地址，Alice 代表第二个地址。我们还将使用 Anvil 生成的预注资地址之一作为 Alice 的地址。

让我们创建一个 Foundry 脚本，用它来检查部署者的 METoken 余额，并向 Alice 发送一些 METoken。复制以下代码片段的内容，并将其粘贴到一个新文件 METoken/script/METokenInteraction.s.sol 中：
```Solidity
pragma solidity 0.8.28;
import {Script, console} from "forge-std/Script.sol";
import {METoken} from "../src/METoken.sol";
contract METokenInteraction is Script {
    METoken public _METoken = METoken(0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512);
    address alice = 0x70997970C51812dc3A010C7d01b50e0d17dc79C8;
    function run() public {
        vm.startBroadcast();
        uint256 ourBalance = _METoken.balanceOf(msg.sender);
        console.log("Deployer initial balance:", ourBalance);
        uint256 aliceBalance = _METoken.balanceOf(alice);
        console.log("Alice initial balance:", aliceBalance);
        uint256 amountToTransfer = 50e18;
        bool success = _METoken.transfer(alice, amountToTransfer);
        if (success) {
            console.log("Transfer successful");
        } else {
            console.log("Transfer failed");
            revert();
        }
        ourBalance = _METoken.balanceOf(msg.sender);
        console.log("Deployer final balance:", ourBalance);
        aliceBalance = _METoken.balanceOf(alice);
        console.log("Alice final balance:", aliceBalance);
        vm.stopBroadcast();
    }
}
```
我们可以使用以下控制台命令运行该脚本：
```Bash
$ forge script script/METokenInteraction.s.sol --private-key <DEPLOYER_PRIVATE_KEY>
--rpc-url "http://127.0.0.1:8545" -vv
```
使用与部署时相同的私钥至关重要。这能确保 msg.sender 对应于部署者地址，而该地址持有代币的初始供应量。
> [!Note]
> 在运行 forge script 或 forge build 等命令时，Foundry 的 `-v` 标志用于控制输出内容的详细程度（即日志级别）。增加字母 `v` 的数量会提高输出的详细程度，从而包含更多信息。要显示控制台日志（console logs），我们至少需要 `-vv`，而 `-vvvv` 则提供最高级别的详细输出。

一旦我们运行该脚本，控制台将打印以下内容：
```
[⁘] Compiling...
[:] Compiling 1 files with Solc 0.8.28
[⁖] Solc 0.8.28 finished in 312.39ms
Compiler run successful!
Script ran successfully.
== Logs ==
  Deployer initial balance: 50000000000000000000000000
  Alice initial balance: 0
  Transfer successful
  Deployer final balance: 49999950000000000000000000
  Alice final balance: 50000000000000000000
```
在这个脚本中，我们首先记录（log）了部署者和 Alice 当前的代币余额。接着，我们从部署者向 Alice 转账了 50 枚代币，并再次记录了余额。请记住，50 枚代币被表示为 50e18，因为我们的代币有 18 位小数，所以会有大量的零。

#### 向合约地址发送 ERC-20 代币

到目前为止，我们已经创建了一个 ERC-20 代币，并完成了从一个账户到另一个账户的转账。在这些演示中，我们使用的所有账户都是 EOA（外部账户），这意味着它们受私钥控制，而不是受合约代码控制。那么，如果我们把 MET 代币发送到一个合约地址会发生什么呢？让我们一探究竟！

首先，让我们在测试环境中部署另一个合约。在本例中，我们将使用下面这个 NaiveFaucet.sol 合约：
```Solidity
pragma solidity 0.8.28;
contract NaiveFaucet {
    receive() external payable {}
    // Function to withdraw Ether from the contract
    function withdraw(uint256 amount) public {
        require(amount <= address(this).balance, "Insufficient balance in faucet");
        payable(msg.sender).transfer(amount);
    }
}
```
我们的目录结构现在应该是这样的：

```
METoken/
+---- src
|   +---- NaiveFaucet.sol
|   +---- METoken.sol
```
让我们编译并部署 NaiveFaucet 合约：

```Bash
$ forge create NaiveFaucet --broadcast --rpc-url http://localhost:8545
--private-key <DEPLOYER_PRIVATE_KEY>
[⁘] Compiling...
[:] Compiling 1 files with Solc 0.8.28
[⁖] Solc 0.8.28 finished in 8.69ms
Compiler run successful!
Deployer: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
Deployed to: 0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0
Transaction hash: 0x4d1947547e3cfec8db670f3c1b7ff309b41de8aacee42165578a3ddf8619f63f
```
太棒了，我们的 NaiveFaucet 合约已成功部署到地址： 0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0。

现在，让我们通过将以下脚本复制到 METoken/script/METokenSend.s.sol 中，向 NaiveFaucet 合约发送一些 MET 代币：
```Solidity
pragma solidity 0.8.28;
import {Script, console} from "forge-std/Script.sol";
import {METoken} from "../src/METoken.sol";
contract METokenSend is Script {
    METoken public _METoken = METoken(0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512);
    address naiveFaucet = 0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0;
    function run() public {
        vm.startBroadcast();
        uint256 amountToSend = 100e18;
        bool success = _METoken.transfer(naiveFaucet, amountToSend);
        if (success) {
            console.log("Transfer successful");
        } else {
            console.log("Transfer failed");
            revert();
        }
        uint256 faucetBalance = _METoken.balanceOf(naiveFaucet);
        console.log("Faucet balance:", faucetBalance);
        vm.stopBroadcast();
    }
}
```
我们可以运行该脚本：

```Bash
$ forge script script/METokenSend.s.sol --private-key <DEPLOYER_PRIVATE_KEY>
--rpc-url "http://127.0.0.1:8545" -vv
[⁘] Compiling...
[:] Compiling 1 files with Solc 0.8.28
[⁖] Solc 0.8.28 finished in 413.41ms
Compiler run successful!
Script ran successfully.
== Logs ==
  Transfer successful
  Faucet balance: 100000000000000000000
```
同样，我们需要使用部署者的私钥来使其运行，因为该地址是发起转账的地址。

我们已经向 NaiveFaucet 合约转移了 100 枚 MET。现在，我们该如何提取这些代币呢？

请记住，NaiveFaucet.sol 是一个非常简单的合约。它只有一个函数 withdraw，而那是用于提取 以太币（Ether） 的。它并没有用于提取 MET 或任何其他 ERC-20 代币的函数。如果我们调用 withdraw，它会尝试发送以太币，但由于 NaiveFaucet 目前还没有以太币余额，它将会失败。

METoken 合约知道 NaiveFaucet 拥有余额，但它能转移该余额的唯一方式是接收到来自该合约地址的 transfer 调用。也就是说，我们需要想办法让 NaiveFaucet 合约去调用 METoken 里的 transfer 函数。

如果你正在苦思冥想下一步该怎么做，别想了。这个问题没有解决方案。 发送到 NaiveFaucet 的 MET 被永久卡住了。只有 NaiveFaucet 合约能转移它，而 NaiveFaucet 并没有编写调用 ERC-20 代币合约 transfer 函数的代码。

也许你预见到了这个问题。但更有可能的是，你没有。事实上，数以百计的以太坊用户都曾不小心将各种代币转移到了没有任何 ERC-20 处理能力的合约中。多年来，数额惊人的数百万美元资产就这样被“卡住”并永久丢失了。

#### 演示 “Approve” 与 “TransferFrom” 工作流

我们之前的 NaiveFaucet 合约无法处理 ERC-20 代币。使用 transfer 函数向其发送代币导致了资产的丢失。现在，让我们重写合约，使其能够正确处理 ERC-20 代币。具体来说，我们将把它变成一个向任何请求者发放 MET 的水龙头。

我们的新水龙头合约 METFaucet.sol 将如示例 10-2 所示。
```Solidity
pragma solidity 0.8.28;
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
contract METFaucet {
    IERC20 public _METoken;
    address public _METOwner;
    constructor(address _metokenAddress, address metOwner) {
        _METoken = IERC20(_metokenAddress);
        _METOwner = metOwner;
    }
    // Function to withdraw METoken from the contract
    function withdraw(uint256 amount) public {
        require(amount <= 10e18, "At most 10 MET");
        require(_METoken.transferFrom(_METOwner, msg.sender, amount), "Transfer failed");
    }
}
```
我们对基础的 Faucet 示例做了相当多的改动。由于 METFaucet 将使用 METoken 中的 transferFrom 函数，它需要两个额外的变量。一个用于存储 METoken 合约的地址；另一个用于存储 MET 持有者（Owner）的地址，这位持有者将授权（Approve）水龙头进行提款。在我们的例子中，持有者就是部署者，因为他们接收了初始供应量。METFaucet 合约将调用 METoken.transferFrom，并指示其将 MET 从持有者处转移到发起提款请求的地址。

我们在这里声明这两个变量：
```
IERC20 public _METoken;
address public _METOwner;
```
由于我们的水龙头需要使用 METoken 和 METOwner 的正确地址进行初始化，我们需要声明一个自定义构造函数：
```Solidity
// METFaucet constructor - provide the address of the METoken contract and
// the owner address we will be approved to transferFrom
constructor(address _metokenAddress, address metOwner) {
    _METoken = IERC20(_metokenAddress);
    _METOwner = metOwner;
}
```
下一个改动是在 withdraw 函数上。METFaucet 不再调用 transfer，而是使用 METoken 中的 transferFrom 函数，并请求 METoken 将 MET 转移给水龙头的接收者：
```Solidity
// Use the transferFrom function of METoken
_METoken.transferFrom(metOwner, msg.sender, withdraw_amount);
```
最后，由于我们的水龙头不再发送以太币，我们应该防止任何人向 METFaucet 发送以太币，以免其被卡住。要拒绝转入的以太币，只需从我们的合约中移除 receive 函数即可。

现在 METFaucet.sol 的代码已经准备就绪，我们可以通过提供 MET 代币地址及其部署者地址作为参数来部署它：

```Bash
$ forge create METFaucet --broadcast --rpc-url http://localhost:8545 --private-key
<DEPLOYER_PRIVATE_KEY> --constructor-args
"0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512""0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266"
[⁘] Compiling...
No files changed, compilation skipped
Deployer: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
Deployed to: 0xCf7Ed3AccA5a467e9e704C703E8D87F634fB0Fc9
Transaction hash: 0xa8bfbde9489ee40d41328a80538d0d3e7778b7f3b896c1d51897bf85bb25cec2
```
METFaucet 合约已部署到 `0xCf7Ed3AccA5a467e9e704C703E8D87F634fB0Fc9`，我们几乎准备好测试它了。首先，让我们编写一个 METApprove.s.sol 脚本，授权 METFaucet 合约可以使用所有者的 MET 代币：

```Solidity
pragma solidity 0.8.28;
import {Script, console} from "forge-std/Script.sol";
import {METoken} from "../src/METoken.sol";
contract METApprove is Script {
    METoken public _METoken = METoken(0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512);
    address _METFaucet = 0xCf7Ed3AccA5a467e9e704C703E8D87F634fB0Fc9;
    function run() public {
        vm.startBroadcast();
        bool success = _METoken.approve(_METFaucet, type(uint256).max);
        if (success) {
            console.log("Approve successful");
        } else {
            console.log("Approve failed");
            revert();
        }
        vm.stopBroadcast();
    }
}
```
我们可以通过以下命令运行它：
```Bash
$ forge script script/METApprove.s.sol --broadcast --private-key <DEPLOYER_PRIVATE_KEY>
--rpc-url "http://127.0.0.1:8545" -vv
[⠊] Compiling...
[⠰] Compiling 2 files with Solc 0.8.28
[⠔] Solc 0.8.28 finished in 327.90ms
Compiler run successful!
Script ran successfully.
== Logs ==
  Approve successful
```
现在，我们可以编写一个脚本，让第二个地址与 METFaucet 合约进行交互，提取 10 枚 MET 代币，并记录操作前后的余额。让我们按照示例 10-3 创建一个 `METFaucetWithdraw.s.sol` 脚本。

示例 10-3：METFaucetWithdraw —— 一个水龙头提款脚本
```Solidity
pragma solidity 0.8.28;
import {Script, console} from "forge-std/Script.sol";
import {METoken} from "../src/METoken.sol";
import {METFaucet} from "../src/METFaucet.sol";
contract METFaucetWithdraw is Script {
    METoken public _METoken = METoken(0xe7f1725E7734CE288F8367e1Bb143E90bb3F0512);
    METFaucet public _METFaucet = METFaucet(0xCf7Ed3AccA5a467e9e704C703E8D87F634fB0Fc9);
    function run() public {
        vm.startBroadcast();
       uint256 balanceBefore = _METoken.balanceOf(msg.sender);
       console.log("Alice balance before:", balanceBefore);
       _METFaucet.withdraw(10e18);
       uint256 balanceAfter = _METoken.balanceOf(msg.sender);
       console.log("Alice balance after:", balanceAfter);
        vm.stopBroadcast();
    }
}
```
现在，我们可以在运行脚本时提供 Alice 的私钥，从而以 Alice 的地址来运行此脚本：

```Bash
$ forge script script/METFaucetWithdraw.s.sol --broadcast --private-key <ALICE_PRIVATE_KEY>
--rpc-url "http://127.0.0.1:8545" -vv
[⠊] Compiling...
[⠰] Compiling 1 files with Solc 0.8.28
[⠔] Solc 0.8.28 finished in 330.97ms
Compiler run successful!
Script ran successfully.
== Logs ==
  Alice balance before: 0
  Alice balance after: 10000000000000000000
```
正如你从结果中看到的，我们可以利用 approve 和 transferFrom 的工作流，授权一个合约去转移在另一个代币合约中定义的代币。如果使用得当，ERC-20 代币可以被 EOA（外部账户）和其他合约共同使用。
然而，正确管理 ERC-20 代币的责任被推给了用户界面（User Interface）。如果用户错误地尝试将 ERC-20 代币转移到一个合约地址，而该合约并未具备接收 ERC-20 代币的功能，那么这些代币将会丢失。

### ERC-20 代币存在的问题

ERC-20 代币标准的采用呈爆炸式增长。成千上万的代币被发布，既为了实验新功能，也为了在各种众筹拍卖和 ICO 中筹集资金。然而，正如我们看到的将代币转入合约地址会导致丢失一样，这里也存在一些潜在的陷阱。

一个较不明显的 issue 是，ERC-20 代币暴露了代币与以太币（ETH）之间的微妙差异。以太币是通过以收款地址为目的地的交易进行转移的，而代币转移则发生在特定的代币合约状态内，其交易的目的地是代币合约本身，而非收款人的地址。代币合约负责跟踪余额并发布事件。在代币转账中，实际上并没有任何交易发送给代币的接收者。相反，接收者的地址只是被添加到代币合约内部的一个映射（Mapping）中。

发送以太币到某个地址会改变该地址的状态。而将代币转移到某个地址仅改变了代币合约的状态，并未改变收款地址的状态。即便是一个支持 ERC-20 的钱包，除非用户显式添加特定的代币合约进行“监视”，否则它无法察觉到代币余额的存在。

事实上，用户不太可能想要追踪所有可能的 ERC-20 合约余额。许多 ERC-20 代币更像是“电子邮件垃圾邮件”而非可用的代币。它们会自动为有活跃记录的账户创建余额以吸引用户。如果你有一个活动历史悠久的地址，你会发现它莫名其妙地塞满了各种“垃圾”代币。当然，你的地址里并没有真的塞满代币，而是那些代币合约里记录了你的地址。

代币的行为与以太币完全不同。以太币通过 send 函数发送，并能被合约中任何 payable 函数或任何外部账户（EOA）接收。代币则使用 transfer 或 approve/transferFrom 发送，这些函数仅存在于 ERC-20 合约中，且（至少在 ERC-20 中）不会触发接收合约中的任何 payable 函数。代币本应像以太币这样的加密货币一样运作，但这些差异打破了这种错觉。

对于新用户来说，approve-transferFrom 系统尤其具有误导性。许多人认为转账应该是单步操作，因此这种两步流程让人感到困惑且反直觉。更糟的是，approve 操作看似无害，实则可能是个陷阱。当用户授权（Approve）一个合约时，他们可能会在不知情的情况下授予无限权限，让恶意行为者随后搬空他们的代币。

再考虑另一个问题：Gas 费。发送以太币需要以太币支付 Gas 费。发送代币，同样需要以太币。你不能用代币支付交易的 Gas，代币合约也无法代付。假设你用比特币换了一些代币，钱包里显示了余额，看起来和其他加密货币没区别。但当你尝试发送它时，钱包会提示你需要以太币。这会让人困惑——毕竟你接收代币时并不需要以太币。

我们可以通过 ERC-2612 的 permit 函数以及 EIP-4337 和 EIP-7702 等智能钱包提供的 Gas 费代付功能来部分解决这一问题。ERC-2612 允许你通过签署离线消息来授权额度，省去了链上授权交易。EIP-4337 则允许第三方为你支付 Gas 费，并换取 ERC-20 代币作为补偿。然而，挑战在于 ERC-2612 在代币中的采用率有限，且智能钱包尚未大规模普及。

### ERC-223：一种提议的代币合约接口标准
ERC-223 提案试图通过检测目标地址是否为合约，来解决误将代币转移到（可能支持也可能不支持代币的）合约中的问题。ERC-223 要求那些旨在接收代币的合约必须实现一个名为 tokenFallback 的函数。如果转账的目标是一个合约，而该合约不支持代币（即未实现 tokenFallback），转账则会失败。

为了检测目标地址是否为合约，ERC-223 的参考实现以一种相当有创意的方式使用了一小段内联字节码（inline bytecode）：
```Solidity
function isContract(address _addr) private view returns (bool is_contract) {
  uint256 length;
    assembly {
       // retrieve the size of the code on target address; this needs assembly
       length := extcodesize(_addr)
    }
    return (length>0);
}
```

> [!Note]
> extcodesize 会返回存储在给定地址处的字节码大小。从历史上看，这是 EOA（外部账户） 与 智能合约 之间的主要区别：EOA 没有代码，而合约有。但这一假设现在已不再成立。随着 EIP-7702 的引入，EOA 现在可以挂载（attach）代码，这彻底模糊了两者的界限。此外，还有一个重要的边缘情况：在合约的构造函数（Constructor）阶段，其代码尚未存储在链上。EVM 只有在构造函数执行完毕后，才会将合约的字节码写入该地址。因此，如果你在构造函数内部对合约自身的地址调用 extcodesize（或者对另一个尚未完成部署的合约调用），它将返回 0，尽管该地址最终会包含代码。简而言之，如果一个地址没有代码，它可能是一个 EOA，也可能是一个正在构造中的合约；而如果它确实有代码，它可能是一个已部署的合约，也可能是一个使用了自定义代码负载的 EOA。总之，这种检查已不再能可靠地告诉我们一个地址是否为合约。

ERC-223 合约接口规范如下：
```Solidity
interface ERC223Token {
  uint256 public totalSupply;
  function balanceOf(address who) public view returns (uint256);
  function name() public view returns (string _name);
  function symbol() public view returns (string _symbol);
  function decimals() public view returns (uint8 _decimals);
  function totalSupply() public view returns (uint256 _supply);
  function transfer(address to, uint256 value) public returns (bool success);
  function transfer(address to, uint256 value, bytes data) public returns (bool success);
  function transfer(address to, uint256 value, bytes data, string custom_fallback)
      public returns (bool success);
  event Transfer(address indexed from, address indexed to, uint256 value,
                 bytes indexed data);
}
```
ERC-223 并没有被广泛采用。在 ERC [讨论帖](https://oreil.ly/iguyT)中，关于“向后兼容性”以及“究竟是在合约接口层还是在用户界面（UI）层实施变更”的权衡，一直存在争议。这场辩论至今仍在继续。

### ERC-777：原本可能的未来

ERC-777 通过引入钩子（Hooks）——即在代币转账过程中触发的函数——为代币交互带来了一种全新的方式。这些钩子与 ERC-20 完全兼容，确保了现有系统可以无缝地与 ERC-777 代币进行交互。

发送者的钩子 tokensToSend 在代币离开账户前执行，允许发送者添加日志记录或条件检查等逻辑。另一方面，接收者的钩子 tokensReceived 则在代币进入账户时立即生效。ERC-777 钩子架构的核心是 ERC-1820 注册表，它负责记录哪些地址实现了必要的钩子：发送者的 tokensToSend 和接收者的 tokensReceived。这确保了只有在双方都准备好处理转账时，转账才会成功，从而防止了代币丢失等常见问题。

钩子还简化了交易流程。在 ERC-20 中，将代币转移到合约通常需要繁琐的两步操作：先 approve（授权），再 transferFrom（转账）。ERC-777 摒弃了这种做法，通过其钩子机制实现了原子交易（Atomic Transactions）。它高效、直观，坦率地说，这项改进早就该出现了。

ERC-777 的另一个有趣特性是其操作员（Operator）机制，它允许获得授权的地址（通常是智能合约，如交易所或支付处理器）代表持有者发送和销毁代币。持有者可以随时授予或撤销对操作员的授权，从而完全控制哪些第三方可以在任何给定时刻代表他们管理代币。每次授权或撤销都会发出一个事件，为授权变更提供透明度。

```Solidity
interface ERC777Token {
    function name() public view returns (string);
    function symbol() public view returns (string);
    function totalSupply() public view returns (uint256);
    function granularity() public view returns (uint256);
    function balanceOf(address owner) public view returns (uint256);
    function send(address to, uint256 amount, bytes userData) public;
    function authorizeOperator(address operator) public;
    function revokeOperator(address operator) public;
    function isOperatorFor(address operator, address tokenHolder)
        public constant returns (bool);
    function operatorSend(address from, address to, uint256 amount,
                          bytes userData,bytes operatorData) public;
    event Sent(address indexed operator, address indexed from,
               address indexed to, uint256 amount, bytes userData,
               bytes operatorData);
    event Minted(address indexed operator, address indexed to,
                 uint256 amount, bytes operatorData);
    event Burned(address indexed operator, address indexed from,
                 uint256 amount, bytes userData, bytes operatorData);
    event AuthorizedOperator(address indexed operator,
                             address indexed tokenHolder);
    event RevokedOperator(address indexed operator, address indexed tokenHolder);
}
```
#### ERC-777 代币存在的问题
虽然 ERC-777 让代币交互变得更加直观，但其钩子（Hooks）机制也引入了一些重大挑战——其中最著名的便是**重入攻击**（Reentrancy Attacks）风险。此类攻击利用了合约在完整更新其状态之前，能够重新进入自身逻辑的能力，往往导致严重后果。ERC-777 钩子的本质是在代币转账期间将执行控制权移交给发送方和接收方，这为这类漏洞利用创造了理想场景。这种设计意味着开发者必须极其谨慎地处理这些钩子，以避免安全漏洞。

因集成 ERC-777 代币而导致重入攻击的最臭名昭著的案例是 2020 年 4 月的 Uniswap v1 事件。Uniswap 作为一个去中心化交易所协议，由于 ERC-777 的钩子机制，无意中将其储备金暴露在了攻击风险之下。攻击者可以在操作中途回调（Call back）Uniswap 合约，利用代币与以太币储备金之间的差价来抽走资金。虽然这一漏洞特发于 Uniswap 与 ERC-777 的交互方式，但它有力地提醒了人们这些钩子所引入的风险。

另一个值得注意的问题是潜在的 DoS（拒绝服务）攻击。假设一个合约向多个账户分发 ERC-777 代币。如果其中一个接收者是恶意合约，且被编程为在执行 tokensReceived 钩子时触发交易回滚（Revert），那么整个分发过程都会被阻断。

这些风险凸显了为什么集成 ERC-777 并非简单地替换 ERC-20。开发者需要采用最佳实践，例如依赖**重入锁**（Reentrancy Locks）来降低风险。

#### 原本可能的未来

ERC-777 本可以成为以太坊代币标准的下一次进化。它解决了 ERC-20 的许多痛点，特别是在用户体验和代币处理方面。预防代币丢失、实现原子交易以及构建更丰富的合约交互，这些都是极具吸引力的进步。

然而，以太坊社区变得犹豫不决。潜在的 DoS（拒绝服务）和重入攻击场景留下了长久的阴影。开发者们害怕集成 ERC-777 所带来的复杂性和风险，即便这些风险在采取适当预防措施的情况下是可控的。我们没有去迎接挑战，而是选择守着熟悉但功能有限的 ERC-20。这错失了一个拥抱更符合以太坊哲学且功能更优越标准的机会。

归根结底，ERC-777 的钩子（Hooks）并不是罪魁祸首，不当的实现才是。只要稍加努力并遵循安全编码规范，ERC-777 本可以为一个更无缝的以太坊生态系统铺平道路。若想进一步研究此问题，建议阅读 OpenZeppelin 库关于[弃用 ERC-777 的讨论](https://oreil.ly/J4p9B)。

## ERC-721：NFT 标准

到目前为止，我们已经探讨了像 ERC-20 这样的同质化代币标准，在这种标准下，每个单位都是可以互换的，系统只关心账户余额。现在，让我们深入研究一些完全不同且更加独特的东西：ERC-721，即非同质化代币标准，或者正如现在人尽皆知的名字——NFT。你可能在 2021 年的狂热时期听说过 NFT，当时像 CryptoPunks、Bored Ape Yacht Club（无聊猿）和 NBA Top Shot 这样的数字收藏品占据了头条新闻，并以令人咋舌的高价成交。OpenSea 和 Rarible 等交易平台也成为了加密世界中家喻户晓的名字。

但究竟是什么让 NFT 如此特别？与 ERC-20 代币不同，NFT 的核心在于唯一性。每个代币都代表一个独特物品的所有权，而这个物品可以是任何东西：数字收藏品、游戏内资产、一件艺术品，甚至是现实世界的资产，如房产或汽车。ERC-721 的精妙之处在于它的灵活性。它不在乎代币代表什么，只要它是唯一的，并且能通过数字进行识别。在底层，这是通过为每个 NFT 分配一个 uint256 类型的标识符（ID）来实现的。你可以把它看作是区分一个代币与另一个代币的“序列号”。在 ERC-20 中，我们通过将账户映射到其持有的金额来追踪余额；但在 ERC-721 中，除了余额，我们还将每个唯一的代币 ID 映射到其所有者：
```Solidity
mapping (uint256 => address) private _owners;
```
这种结构上的细微转变改变了一切。每个 NFT 都与特定所有者绑定，其历史、来源（Provenance）和独特属性都可以被直接追踪。这使得 ERC-721 非常适合那些强调“个体差异”的应用场景，例如证明一件独一无二的艺术品、一把稀有的游戏宝剑、甚至元宇宙中一块土地的所有权。

让 ERC-721 真正强大且用途广泛的是可选的 ERC-721 元数据扩展（Metadata Extension），它允许将每个代币 ID 与一个**统一资源标识符**（URI）绑定。这个 URI 可以指向描述该 NFT 的元数据，例如它的名称、描述和图像。URI 可以是指向中心化服务器的标准 HTTP 链接，也可以是用于去中心化存储的 IPFS（星际文件系统） 链接。中心化服务器可能会宕机或彻底消失，从而使 NFT 依赖于第三方基础设施。相比之下，IPFS 有助于确保元数据保持可访问且不可篡改，因此是存储 NFT 元数据的首选。将每个代币与元数据 URI 关联的能力，使 NFT 能够承载丰富的描述性数据和多媒体内容，进一步增强了它们的唯一性和可用性。

不可否认，NFT 的投机性用途（尤其是作为数字收藏品）主导了公众舆论。这些代币成为了炒作的象征，其价值受稀缺性、名人代言和社区情绪驱动。然而，在闪耀的头条新闻背后，是一个具有重新定义行业潜力的实用应用世界。例如，NFT 可用于供应链管理以追踪商品的来源，确保消费者的真实性和透明度。它们可以代表房产证，通过自动化所有权转移和减少欺诈来简化房地产交易。NFT 还可以应用于教育领域，颁发可验证的凭证（如学位和证书），这些凭证既防篡改又具有普世可访问性。

虽然收藏品将 NFT 推向了聚光灯下，但它们真正的潜力在于这些务实的、变革性的用途。通过结合唯一性、可追溯性和可编程性，ERC-721 代币有望成为数字经济的基础基石。无论你是在铸造一件奇特的艺术品，还是将一份救命的医疗记录代币化，ERC-721 都赋予了开发者创造远超投机价值的解决方案的能力。

ERC-721 合约接口规范如下：
```Solidity
interface ERC721 /* is ERC165 */ {
    event Transfer(address indexed _from, address indexed _to, uint256 _deedId);
    event Approval(address indexed _owner, address indexed _approved,
                   uint256 _deedId);
    event ApprovalForAll(address indexed _owner, address indexed _operator,
                         bool _approved);
    function balanceOf(address _owner) external view returns (uint256 _balance);
    function ownerOf(uint256 _deedId) external view returns (address _owner);
    function transfer(address _to, uint256 _deedId) external payable;
    function transferFrom(address _from, address _to, uint256 _deedId)
        external payable;
    function approve(address _approved, uint256 _deedId) external payable;
    function setApprovalForAll(address _operator, boolean _approved) payable;
    function supportsInterface(bytes4 interfaceID) external view returns (bool);
}
```

## ERC-1155：多代币标准

自 ERC-20 和 ERC-721 为同质化与非同质化代币奠定基础以来，以太坊已经走过了很长一段路。ERC-1155，即多代币标准（Multitoken Standard），结合了两者的优势，实现了高效且通用的代币管理。

想象你是一名游戏开发者。你想创建一个系统，让玩家可以收集金币（同质化）、独特的宝剑（非同质化）以及成堆使用的药水（半同质化，Semi-fungible）。如果使用 ERC-20 或 ERC-721，你需要为每种类型的代币部署一个独立的智能合约。每个新合约都会增加 Gas 成本并提升复杂性，因为你还需要管理这些合约之间的权限和交互。ERC-1155 解决了这个问题。它允许我们通过单个合约来管理多种代币类型。每种代币类型由唯一的 ID 标识，合约可以处理任何代币组合的所有操作，包括转账、余额查询和元数据检索。这种精简的方法减少了冗余和交易成本，使 ERC-1155 成为一种被广泛采用的标准。

ERC-1155 的批量操作（Batch Operations）是一大亮点。它们允许我们在单笔交易中转账或查询多个代币，从而节省 Gas 并提高标准的扩展性。此外，ERC-1155 包含安全机制以防止代币锁死等问题。在将代币转移到智能合约时，接收合约必须实现 IERC1155Receiver 接口；否则，交易将回滚。这些钩子（Hooks）使代币转账期间的高级交互成为可能。例如，合约可以实现自定义逻辑，在收到代币时执行特定操作，如更新游戏排行榜或触发事件。然而，正如在 ERC-777 中所见，如果对这些钩子处理不当，它们可能会引入漏洞，例如重入攻击。开发者必须确保采取适当的防护措施，例如使用“检查-效果-交互”（Checks-effects-interactions）模式和重入锁，以在实现 ERC-1155 代币时确保合约安全。

```Solidity
interface IERC1155 /* is IERC165 */ {
    event TransferSingle(address indexed operator, address indexed from, address indexed to,
uint256 id, uint256 value);
    event TransferBatch(
        address indexed operator,
        address indexed from,
        address indexed to,
        uint256[] ids,
        uint256[] values
    );
    event ApprovalForAll(address indexed account, address indexed operator, bool approved);
    event URI(string value, uint256 indexed id);
    function balanceOf(address account, uint256 id) external view returns (uint256);
    function balanceOfBatch(
        address[] calldata accounts,
        uint256[] calldata ids
    ) external view returns (uint256[] memory);
    function setApprovalForAll(address operator, bool approved) external;
    function isApprovedForAll(
        address account,
        address operator
    ) external view returns (bool);
    function safeTransferFrom(
        address from,
        address to,
        uint256 id,
        uint256 value,
        bytes calldata data
    ) external;
    function safeBatchTransferFrom(
        address from,
        address to,
        uint256[] calldata ids,
        uint256[] calldata values,
        bytes calldata data
    ) external;
}
```

## 代币使用标准
在上一节中，我们回顾了几种提议中的标准以及几种已广泛部署的代币合约标准。这些标准究竟起到了什么作用？你是否应该使用这些标准？该如何使用它们？是否应该在这些标准之外增加功能？你应该选择哪种标准？接下来我们将探讨其中的一些问题。

### 什么是代币标准？它们的目的是什么？
代币标准是实现的最低规范。这意味着，为了符合（例如）ERC-20 标准，你至少需要实现该标准所指定的函数和行为。同时，你也可以通过实现标准之外的函数来增加功能。

这些标准的主要目的是促进合约之间的互操作性（Interoperability）。这样，所有的钱包、交易所、用户界面和其他基础设施组件都能以可预测的方式，与任何遵循该规范的合约进行交互（Interface）。换句话说，如果你部署了一个遵循 ERC-20 标准的合约，所有现有的钱包用户都可以无缝地开始交易你的代币，而无需你进行任何钱包升级或付出额外努力。

标准通常是**描述性**（Descriptive）的，而非**指令性**（Prescriptive）的。你如何选择实现这些函数完全取决于你自己；合约的内部运作方式与标准无关。标准会有一些功能性要求，规定了在特定情况下的行为，但它们并不规定具体的代码实现。一个例子是当转账金额（value）设为零时 transfer 函数的行为：ERC-20 标准并未规定在这种情况下交易是否应该回滚（Revert）。

### 你是否应该使用这些标准？
面对这些标准，每个开发者都面临一个抉择：是使用现有标准，还是突破它们施加的限制进行创新？

这个抉择并不容易。标准通过创建一条你必须遵循的狭窄“路径”，必然会限制你的创新能力。但另一方面，基础标准是从成百上千个应用的经验中总结出来的，通常能很好地适配绝大多数用例。

在这一考量中，一个更重大的问题是：互操作性和广泛采用的价值。如果你选择使用现有标准，你就获得了所有为该标准设计的系统的价值。如果你选择背离标准，你就必须考虑自行构建所有支持基础设施的成本，或者说服他人支持你的实现并将其作为新标准。这种倾向于自辟蹊径而忽视现有标准的做法被称为**非我所创**（Not Invented Here, NIH）”综合征，这与开源文化背道而驰。然而，进步和创新有时确实取决于对传统的背离。这是一个微妙的选择，请务必仔细权衡！

> [!Note]
> 根据维基百科，“非我所创”是社会、企业或机构文化中采取的一种立场，由于产品的外部来源及版税等成本，它们会避免使用或购买已有的产品、研究、标准或知识。

### 检测标准：EIP-165

正如我们所见，像 ERC-20 这样的标准简化了代币与钱包之间的交互。但我们该如何识别一个智能合约究竟支持哪些接口呢？这正是 EIP-165 的用武之地——它为合约提供了一种标准化的方式来声明和检测接口。

EIP-165 定义了一种方法，让合约能够宣布它们所实现的接口。合约使用 supportsInterface 函数针对给定的接口 ID 返回 true 或 false（接口 ID 是通过对接口中所有函数选择器进行 异或（XOR） 运算得出的唯一标识符）。例如，如果一个接口包含 foo() 和 bar(int256)，其 ID 计算如下：
```
foo.selector ^ bar.selector
```
这种方法使其他合约和工具能够在交互前验证兼容性。例如，一个交易市场可以在列出某个 NFT 的代币之前，先确认该 NFT 合约确实支持 ERC-721 接口。

要实现 EIP-165，合约需要继承自一个基类（例如 OpenZeppelin 的 ERC165 实现），并重写 supportsInterface 方法，以包含该合约所支持的接口：
```Solidity
contract MyContract is IMyContract, ERC165 {
    function supportsInterface(bytes4 interfaceId) public view override returns (bool) {
        return interfaceId == type(IMyContract).interfaceId ||
super.supportsInterface(interfaceId);
    }
}
```
为了更直观地理解 EIP-165 在实际开发中是如何运作的，让我们以 ERC-1155 为例。ERC-1155 是一个功能丰富且应用广泛的多代币（Multitoken）合约标准：
```Solidity
abstract contract ERC1155 is Context, ERC165, IERC1155, IERC1155MetadataURI,
    IERC1155Errors {
[...]
    /**
     * @dev See {IERC165-supportsInterface}.
     */
    function supportsInterface(bytes4 interfaceId) public view virtual override(ERC165,
IERC165) returns (bool) {
        return
            interfaceId == type(IERC1155).interfaceId ||
            interfaceId == type(IERC1155MetadataURI).interfaceId ||
            super.supportsInterface(interfaceId);
    }
[...]
}
```
> [!Warning]
> EIP-165 依赖于合约如实报告其功能。一个恶意或实现不当的合约可能会虚假声明支持某个接口，从而导致潜在问题。虽然 EIP-165 提升了开发者体验并减少了交互摩擦，但不应将其视为一种安全保证。

对于更复杂的场景，例如当合约代表其他账户实现接口时，开发者可以探索 ERC-1820。该标准使用一个全局注册表来追踪接口支持情况。虽然 ERC-1820 比 EIP-165 更复杂，但它为去中心化系统提供了更大的灵活性。

### 成熟度带来的安全性

除了选择标准之外，还面临着实现方式的选择。当你决定使用某种标准（如 ERC-20）时，你必须继而决定如何实现一个与之兼容的设计。在以太坊生态系统中，有许多被广泛使用的“参考实现”，或者你也可以从零开始编写自己的代码。同样地，这一选择也代表了一个可能产生严重安全后果的抉择。

现有的实现是经过“实战测试（Battle-tested）”的。虽然无法证明它们是绝对安全的，但其中许多实现支撑着价值数百万美元——在某些情况下甚至是数十亿美元——的代币。它们遭受过反复且猛烈的攻击。到目前为止，尚未发现重大的漏洞。而自行编写代码并非易事；合约被攻破的方式多种多样且极其隐蔽。使用经过充分测试、广泛应用的实现要安全得多。在我们的示例中，我们使用了 OpenZeppelin 对 ERC-20 标准的实现，因为该实现从底层设计开始就专注于安全性。

如果你使用现有的实现，你也可以对其进行扩展。但同样要警惕这种冲动。复杂性是安全的敌人。 你增加的每一行代码都会扩大合约的攻击面（Attack surface），并可能代表着一个潜伏待发的漏洞。在你在合约中存入大量价值并被他人攻破之前，你可能根本察觉不到问题的存在。

> [!Tip]
> 标准的选择和实现方式的选择是智能合约整体安全设计的重要组成部分，但它们并非唯一的考量因素（详见第 9 章）。

## 代币接口标准的扩展

到目前为止，我们讨论的代币标准提供了创建和管理代币的核心功能。然而，它们被刻意设计得非常精简，为项目方留下了扩展和适配特定需求的空间。随着时间的推移，许多项目在这些标准的基础上引入了增强易用性、安全性和灵活性的功能。作为以太坊智能合约的领先代码库，OpenZeppelin 已成为此类扩展的首选来源。让我们探讨一下针对 ERC-20、ERC-721 和 ERC-1155 代币的一些最显著的扩展。

以 ERC-20 为例，我们看到了销毁（Burning）机制的加入。销毁允许代币从流通中永久移除，从而减少供应量，这对于通缩模型或需要刻意控制供应量的代币经济学非常有用。另一方面，一些项目通过引入**总量上限**（Caps）来设定总供应量的硬限制，确保铸造的代币永远不会超过预设的阈值。

另一个有趣的 ERC-20 扩展是投票（Voting）功能。这允许代币持有者直接通过其持有的代币参与治理决策。实现这一功能的项目创建了去中心化的决策流程，使利益相关者能够在协议演进中拥有发言权。

ERC-721 在扩展方面也表现出了类似的创意。**版税支付**（Royalty payments）等功能让创作者在每次 NFT 交易时都能获得一定比例的销售分成。URI 存储是另一种常见的补充，它使元数据能够被动态存储和检索，这对于属性会发生变化的 NFT 特别有用。枚举（Enumerable）扩展则允许开发者高效地列出某个地址持有的所有代币，从而更容易构建交易市场或钱包。

作为多代币标准的 ERC-1155 也没有落后。其扩展包括可销毁代币和可暂停（Pausable）合约，在游戏或代币化供应链等用例中增加了灵活性。一些实现还增强了元数据处理，确保代币详情易于访问且方便更新。

除此之外，还存在无数其他扩展，涵盖了众筹、黑名单（Blocklisting）、白名单（Allowlisting）以及转账手续费等需求。开发者通常将这些功能与 OpenZeppelin 的 **Ownable**（所有权控制）或 **Access Control*（访问控制）等标准库结合使用，利用这些经过实战测试的资源。

然而，灵活性伴随着责任。扩展代币标准涉及创新与互操作性之间的平衡。编写自定义功能可能看起来很有吸引力，但往往会引入不必要的复杂性和风险。相反，利用像 OpenZeppelin 这样成熟的库和扩展，既能确保安全和代码质量，又能显著降低开发成本。当强大且经过测试的解决方案已经存在时，完全没有必要重新造轮子。

## 结语

代币不仅仅是数字货币；它们还可以代表治理权、访问凭证、身份以及现实世界的资产。这种多样性的实现，完全归功于 ERC-20、ERC-721 和 ERC-1155 等标准。
这些标准确保了钱包、交易所和去中心化应用（DApps）之间的无缝互操作性，从而构建了一个更高效、更互联的区块链生态系统。
在本章中，我们研究了不同类型的代币及代币标准，并且你已经动手构建了你的第一个代币及其相关应用。








[^1]: 这个词有纯正的日耳曼语血统，既不来自拉丁语，也不来自希腊语。
