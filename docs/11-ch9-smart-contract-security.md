# 第九章：智能合约安全
在编写智能合约时，安全性是最重要的考量因素之一。在智能合约编程领域，错误不仅代价高昂，而且极易被利用。在本章中，我们将研究安全最佳实践和设计模式，
以及安全反模式（Security Antipatterns）——即那些可能向智能合约引入漏洞的实践和模式。

与其他程序一样，智能合约将严格执行所编写的代码，而这并不总是程序员的本意。此外，所有智能合约都是公开的，
任何用户只需发起一笔交易即可与之交互。任何漏洞都可能被利用，且损失几乎总是无法追回。因此，遵循最佳实践并使用经过充分测试的设计模式至关重要。

可以将稳健的开发过程视为安全“瑞士奶牛干酪模型”的第一层。每一层保护都像一片瑞士干酪：每一片本身都不是完美的（都有孔洞），
但它们叠加在一起就能构建出强大的防御。最底层也是第一层保护是遵循扎实的开发实践：使用可靠的设计模式，编写清晰且具意图性的代码，
并主动规避已知陷阱。这一基础层为我们确保合约不受漏洞侵害提供了最佳起点。除此之外，测试、代码审查和漏洞赏金（Bug Bounties）等其他层级提供了额外的保护，但一切都始于我们的开发实践。

## 安全最佳实践
*防御性编程*是一种非常适合智能合约的编程风格。它强调以下几点，这些也都是行业最佳实践：

**极简主义/简单化 (Minimalism/Simplicity)**

在动笔写代码之前，值得退一步思考每一个组件是否真的必要。设计能否简化？某些数据结构是否引入了不必要的风险面？架构确定后，仍应以挑剔的眼光审视代码，寻找减少行数、消除边缘情况或删减非核心特性的机会。简单的合约更容易进行逻辑推理、测试和审计。虽然某些 DeFi 协议的代码量确实会增长到数千行，但当有人吹嘘其代码库规模时，仍应保持怀疑。更多的代码通常意味着更多的 Bug，而不是更多的价值。

**代码复用 (Code Reuse)**

不要重新发明轮子。如果现有的库或合约已经能实现大部分需求，请直接复用。例如，OpenZeppelin 提供了一套经过广泛采用、严密测试且社区持续审查的合约库。在编写代码时，遵循 DRY 原则（Don't Repeat Yourself）：不要重复你自己。如果一段代码重复出现超过一次，思考是否可以将其重写为函数或库。经过实战检验（Battle-tested）的代码几乎总是比你刚写出来的代码更安全，无论你多么自信。警惕“非我所创”综合症（Not Invented Here syndrome）——即为了“改进”某个组件而从零开始构建。其带来的安全风险往往远超改进的价值。

**代码质量 (Code Quality)**

智能合约代码是不容错错的。每一个 Bug 都可能导致资金损失。你不应像对待通用编程那样对待合约编程。编写 Solidity 并不是在写 JavaScript 网页插件，相反，你应该应用航空航天工程或类似严苛学科中的严谨工程方法。代码一旦“发射”，几乎无法修复。即便代码可升级，当问题发生时，你的响应时间也极其有限。如果别人先发现了 Bug，攻击通常在单笔或几笔交易内就会完成，这意味着损失在几秒钟内就已造成，你根本来不及干预。

**可读性/可审计性 (Readability/Auditability)**

代码应当清晰易懂。越容易阅读，就越容易审计。智能合约是公开的：任何人都能读取字节码，高手甚至能进行逆向工程。因此，利用协作和开源方法在阳光下开发是有益的，这能吸纳社区智慧。你应该编写文档齐全、易于阅读的代码，并遵循以太坊社区的风格和命名规范。

**测试覆盖率 (Test Coverage)**

尽可能测试一切。合约运行在公共执行环境中，任何人都能输入任何数据。**绝不要假设输入数据（如函数参数）是格式正确的、边界合理的或带有善意的**。在允许执行继续之前，务必测试所有参数以确保其在预期范围内且格式正确。

## 安全风险与反模式
作为一名智能合约程序员，你应该熟悉最常见的安全风险，以便能够检测并规避那些会让你的合约暴露于这些风险中的编程模式。在接下来的几节中，我们将研究不同的安全风险、漏洞产生的示例，以及可用于解决这些问题的对策或预防性解决方案。

与 Web2 安全类似，以下反模式通常会被组合起来执行攻击。现实世界中的漏洞利用往往比本章中的示例更加复杂。

### 重入攻击（Reentrancy）
以太坊智能合约的一个特性是能够调用并利用其他外部合约的代码。同时，合约通常会处理以太币，因此经常向各种外部用户地址发送以太币。这些操作要求合约发起“外部调用”。而这些外部调用可以被攻击者劫持，迫使合约执行进一步的代码（通过回调：要么是 fallback 回退函数，要么是某些钩子函数，通常是 transfer 操作），其中包括回调该合约自身。这类攻击曾被用于 2016 年那场臭名昭著且至今仍被铭记的 The DAO 劫持事件。即便过去了这么多年，[我们仍然看到大量利用该漏洞的攻击](https://oreil.ly/87VUR)，尽管它其实很容易被发现且修复成本极低。

#### 漏洞原理（The vulnerability）
当攻击者在另一个合约完成状态更新之前，成功夺取了该合约执行过程中的控制权时，这类攻击就会发生。由于合约仍处于处理过程中，它尚未更新其状态（例如关键变量）。攻击者可以在这个脆弱时刻“重新进入”（Reenter）合约，利用不一致的状态来触发非预期或未曾预料的行为。这种重入允许攻击者绕过保护机制、操纵数据或抽干资金，这一切都是因为合约尚未完全进入安全、一致的状态。

如果没有实际案例，重入攻击可能很难理解。让我们看看示例 9-1 中这个简单的漏洞合约，它充当一个以太坊金库，允许存款人每周仅能提取 1 个以太币。

示例 9-1. EtherStore：一个存在重入漏洞的合约
```Solidity
1 contract EtherStore {
2      uint256 public withdrawalLimit = 1 ether;
3      mapping(address => uint256) public lastWithdrawTime;
4    mapping(address => uint256) balances;
5
6    function depositFunds() public payable{
7      balances[msg.sender] += msg.value;
8    }
9
10    function withdrawFunds() public {
11        require(block.timestamp >= lastWithdrawTime[msg.sender] + 1 weeks);
12        uint256 _amt = balances[msg.sender];
13        if(_amt > withdrawalLimit){
14            _amt = withdrawalLimit;
15        }
16      (bool res, ) = address(msg.sender).call{value: _amt}("");
17        require(res, "Transfer failed");
18      balances[msg.sender] = 0;
19        lastWithdrawTime[msg.sender] = block.timestamp;
20    }
21 }
```
该合约有两个公共函数：depositFunds（存款）和 withdrawFunds（取款）。depositFunds 函数只是简单地增加发送者的余额。withdrawFunds 函数允许发送者提取其余额，且该函数旨在仅在过去一周内未发生过取款时才能执行成功。

漏洞位于第 17 行，即合约向用户发送其请求的以太币金额的地方。现在考虑一个创建了示例 9-2 中合约的攻击者。

示例 9-2. Attack.sol：用于利用 EtherStore 合约重入漏洞的攻击合约
```Solidity
1 contract Attack {
2  EtherStore public etherStore;
3
4  // initialize the etherStore variable with the contract address
5  constructor(address _etherStoreAddress) {
6      etherStore = EtherStore(_etherStoreAddress);
7  }
8
9  function attackEtherStore() public payable {
10      // attack to the nearest ether
11      require(msg.value >= 1 ether, "no bal");
12      // send eth to the depositFunds() function
13      etherStore.depositFunds{value: 1 ether}();
14      // start the magic
15      etherStore.withdrawFunds();
16  }
17
18  function collectEther() public {
19      payable(msg.sender).transfer(address(this).balance);
20  }
21
22  // receive function - the fallback() function would have worked out too
23  receive() external payable {
24      if (address(etherStore).balance >= 1 ether) {
25          // reentrant call to victim contract
26          etherStore.withdrawFunds();
27      }
28  }
29 }
```
攻击是如何发生的？首先，攻击者会创建恶意合约（假设地址为 0x0...123），并将 EtherStore 的合约地址作为唯一的构造函数参数。这会初始化公共变量 etherStore 并指向要攻击的目标合约。

随后，攻击者调用 pwnEtherStore 函数，并附带大于或等于 1 的以太币（假设此时发送 1 ETH）。在本例中，我们假设已有其他用户向该合约存入了以太币，因此其当前余额为 10 ETH。接下来将发生以下过程：

* Attack.sol, 第 13 行：调用 EtherStore 的 depositFunds 函数，并发送 1 ether（伴随充足的 Gas）。此时，恶意合约（地址 0x0...123）成为发送者，EtherStore 账本记录：balances[0x0..123] = 1 ether。

* Attack.sol, 第 15 行：恶意合约调用 withdrawFunds 函数。由于之前未进行过取款，通过了 EtherStore 第 11 行的权限要求。
* EtherStore.sol, 第 16 行：合约向恶意合约发送 1 ether。
* Attack.sol, 第 23 行：发往恶意合约的款项触发了其 receive（或 fallback）函数。
* Attack.sol, 第 24 行：此时 EtherStore 余额从 10 ETH 降至 9 ETH，if 语句（检查余额是否足够）判断为真。
* Attack.sol, 第 26 行：receive 函数再次调用 withdrawFunds，“重入”了 EtherStore 合约。
* EtherStore.sol, 第 10 行：在第二次调用 withdrawFunds 时，攻击合约的余额依然显示为 1 ether，因为扣款的第 18 行代码尚未被执行。同理，lastWithdrawTime 变量也未更新。要求再次通过。
* EtherStore.sol, 第 16 行：攻击合约再次提取 1 ether。
* 循环往复：重复重入 EtherStore，直到其余额不再满足 Attack.sol 第 24 行设定的条件（即余额 < 1 ETH）。
* Attack.sol, 第 24 行：一旦 EtherStore 余额不足 1 ETH，if 语句失效。此时，递归开始逐层返回，终于允许 EtherStore 的第 17-19 行代码（此前被挂起的扣款逻辑）在每次调用中依次执行。
* EtherStore.sol, 第 18 和 19 行：更新 balances 和 lastWithdrawTime 映射，执行结束。

最终的结果是，攻击者在单笔交易中就从 EtherStore 合约里提取了所有的以太币。

虽然由回退函数（Fallback functions）拦截的原生以太币转账是重入攻击的常见矢量，但它们并非引入此类风险的唯一机制。一些代币标准，如 ERC-721（NFT）和 ERC-777，也包含可能引发重入攻击的回调机制。例如，ERC-721 的 safeTransfer 函数会确保在向合约转账时调用接收者的 onERC721Received 函数。同样，ERC-777 代币允许在转账期间通过 tokensReceived 函数触发钩子（Hooks）。

#### 超越经典的重入模式
重入攻击并不局限于单个函数或合约。虽然经典的重入涉及在同一个函数结束前再次进入它，但还有一些更难发现的变体，例如跨函数重入、跨合约重入，以及其中最棘手的——只读重入。

只读重入利用了那些依赖于其他合约 view（只读）函数的合约。这些函数本身不修改状态，但会返回其他合约所依赖的数据，且通常没有重入保护。当重入调用允许攻击者让目标合约暂时处于不一致的状态时，问题就出现了：攻击者可以利用另一个合约（受害者）通过只读函数查询这个不稳定的状态。让我们看看它是如何演变的：
1. 攻击者的合约与一个存在漏洞的合约（我们称之为 合约 A）交互，该合约可以被重入。该合约持有其他协议所依赖的数据。
2. 合约 A 触发了一个指向攻击者合约的回调，允许执行攻击者的合约逻辑。
3. 在回调过程中（此时合约 A 的状态尚未更新完成），攻击者的合约调用了另一个协议（合约 B）。合约 B 与合约 A 相关联，并依赖于合约 A 提供的数据。
4. 合约 B 在不知情的情况下从合约 A 读取数据。然而，由于合约 A 尚未完成更新，其状态是过时的。
在整个循环结束前，攻击者已经利用合约 A 的过时数据完成了对合约 B 的攻击，随后才让合约 A 的回调和原始调用正常结束。这一过程如图 9-1 所示。

![Figure 9-1](<./images/figure 9-1.png>)

图 9-1. 只读重入

这里的关键在于：合约 B 信任来自合约 A 的数据，但合约 A 的状态尚未完成同步，从而让攻击者利用了这一“延迟”进行牟利。此类攻击更难防御，因为开发者通常认为 view 函数不修改状态，因而是安全的，所以往往不会为其添加重入锁。只读重入带给我们的教训是：当只读函数被外部合约所依赖时，它们同样可能变得非常危险。

#### 预防技术
为了防止重入问题，首要遵循的最佳实践是在编写智能合约时坚持检查-效果-交互（Check-Effect-Interaction, CEI）模式。该模式的核心是确保在与外部合约交互之前，完成所有对状态变量的修改。例如，在 EtherStore.sol 合约中，修改状态变量的代码行应该出现在任何外部调用之前。其目标是确保任何与外部地址交互的代码都是函数中最后执行的部分。这可以防止外部合约在重入时干扰内部状态，因为必要的更新已经完成。

另一种有用的技术是应用重入锁（Reentrancy Lock）。重入锁是一个简单的状态变量，它在合约执行某个函数时将其“锁定”，防止其他外部调用中断执行。这可以通过如下所示的修饰器（Modifier）来实现：
```Solidity
contract EtherStore {
    bool lock;
      uint256 public withdrawalLimit = 1 ether;
      mapping(address => uint256) public lastWithdrawTime;
    mapping(address => uint256) balances;

      modifier nonReentrant {
      require(lock, "Can't reenter");
      lock = true;
      _;
      lock = false;
    }

    function withdrawFunds() public nonReentrant{
        [...]
    }
}
```
在这个例子中，nonReentrant 修饰器利用锁变量防止 withdrawFunds 函数在运行期间被重入。该修饰器在函数开始时锁定合约，并在结束后解锁。然而，我们不应重新发明轮子。与其自己编写重入锁，不如依赖像 OpenZeppelin 的 ReentrancyGuard 这样经过充分测试的库。这些库提供了安全且经过 Gas 优化的解决方案。

> [Note]
> 随着 Solidity 0.8.24 引入了以太坊的瞬时存储（Transient Storage），OpenZeppelin 推出了 ReentrancyGuardTransient。这是 ReentrancyGuard 的一个新变体，它利用瞬时存储显著降低了 Gas 成本。由 [EIP-1153](https://oreil.ly/fRvKl) 启用的瞬时存储提供了一种更廉价的方式来存储仅在单笔交易期间需要的数据，这使其成为重入锁及类似临时逻辑的理想选择。然而，ReentrancyGuardTransient 只能在支持 EIP-1153 的链上使用，因此在实施前请确保你的目标链支持此功能。

另一种方法是使用 Solidity 内置的 transfer 和 send 函数来发送以太币。这些函数仅转发极少量的 Gas（2,300 单位），这通常不足以让接收合约执行重入调用，因此它们是防御重入的一种简单方法。然而，它们也有显著的缺点。如果接收者是一个在其 fallback 或 receive 函数中带有正当逻辑的智能合约，转账可能会失败，从而导致资金锁死。随着 EIP-7702 的引入（该提案允许普通账户 EOA 附加代码，包括回退逻辑），这种风险正变得越来越大。随着更多 EOA 采用此功能，使用 transfer 或 send 的交易更有可能因为 Gas 不足而在常规执行中触发回退（Revert）。从安全角度来看，这种方法并非“面向未来”：如果未来的硬分叉降低了某些操作的 Gas 成本，2,300 单位可能变得足以执行重入，从而打破先前安全的假设。因此，虽然 transfer 和 send 在极少数情况下仍有帮助，但我们需要谨慎使用，不能将其作为主要的防御手段。

只读重入和跨合约重入值得特别关注：这些漏洞可能涉及两个独立的协议，这使得协调防御变得极具挑战。当我们的项目依赖外部协议提供的数据时，我们需要深入研究组合后的逻辑是如何运作的。即使每个项目自身是安全的，在集成过程中也可能出现漏洞。

#### 真实案例：The DAO 攻击
重入漏洞在 2016 年发生的 The DAO 攻击中扮演了核心角色，那是以太坊早期发展过程中最重大的黑客事件之一。当时，该合约持有超过 1.5 亿美元，占以太坊流通总量的 15%。为了逆转黑客攻击的影响，以太坊社区最终选择进行硬分叉（Hard Fork），导致以太坊区块链一分为二。结果，以太坊经典（ETC）作为原始链继续存在，而通过更新规则以撤销攻击的镜像链则成为了我们今天所熟知的以太坊（ETH）。

#### 真实案例：Libertify
2023 年 7 月发生的 Libertify 协议遭袭案是一个重入漏洞作为唯一攻击矢量的近距离案例，该 DeFi 协议因此损失了 40 万美元。让我们检查一下被攻击函数的代码，看看它是如何发生的：
```Solidity
function _deposit(
    uint256 assets,
    address receiver,
    bytes calldata data,
    uint256 nav
) private returns (uint256 shares) {
        /*
        validations
    */
    uint256 returnAmount = 0;
    uint256 swapAmount = 0;
    if (BASIS_POINT_MAX > invariant) {
        swapAmount = assetsToToken1(assets);
        returnAmount = userSwap( // External call
            data,
            address(this),
            swapAmount,
            address(asset),
            address(other)
        );
    }
    uint256 supply = totalSupply(); // State update
    if (0 < supply) {
        uint256 valueToken0 = getValueInNumeraire(
            asset,
            assets - swapAmount,
            MathUpgradeable.Rounding.Down
        );
        uint256 valueToken1 = getValueInNumeraire(
            other,
            returnAmount,
            MathUpgradeable.Rounding.Down
        );
        shares = supply.mulDiv(
            valueToken0 + valueToken1,
            nav,
            MathUpgradeable.Rounding.Down
        );
    } else {
        shares = INITIAL_SHARE;
    }
    uint256 feeAmount = shares.mulDiv(
        entryFee, BASIS_POINT_MAX, MathUpgradeable.Rounding.Down
    );
    _mint(receiver, shares - feeAmount);
    _mint(owner(), feeAmount);
}
```
这是一个经典的重入问题案例——如今你已经很难见到如此直观的漏洞了！这里的核心问题在于缺乏重入保护。userSwap() 函数允许攻击者在原始调用更新 totalSupply（总供应量）之前，重新进入 deposit()（存款）函数。这意味着攻击者可以铸造比他们实际应得更多的份额（Shares），从而利用该合约牟利。

### 委派调用（DELEGATECALL）
CALL 和 DELEGATECALL 操作码对于以太坊开发者实现代码模块化非常有用。对合约的标准外部消息调用由 CALL 操作码处理，它在被调用合约的上下文中执行代码。相比之下，DELEGATECALL 虽然运行的是另一个合约的代码，但其执行环境却处于调用方合约的上下文中。这意味着存储空间（Storage）、发送者（msg.sender）和发送金额（msg.value）都保持不变。
理解 DELEGATECALL 的一个有效方式是：调用合约临时“借用”了被调用合约的字节码，并像执行自己的代码一样运行它。这使得像代理合约（Proxy Contracts）和库（Libraries）这样强大的设计模式成为可能——你可以一次性部署可重用的逻辑，并在多个合约中复用。尽管这两个操作码之间的区别简单且直观，但使用 DELEGATECALL 可能会导致细微且难以预料的行为，特别是在存储布局（Storage Layout）方面。如需进一步阅读，请参阅 Loi Luu 在 Ethereum Stack Exchange 上关于此话题的[内容](https://oreil.ly/GJOUl)以及 [Solidity 官方文档](https://oreil.ly/gA7vg)。

#### 漏洞原理（The vulnerability）
由于 DELEGATECALL 具有保留上下文（Context-preserving）的特性，构建无漏洞的自定义库并非像你想象的那么简单。库本身的代码可能是安全且无漏洞的；然而，当它在另一个应用程序的上下文中运行时，就可能产生新的漏洞。让我们看一个使用斐波那契数列（Fibonacci numbers）的相当复杂的例子。考虑示例 9-3 中的库，它可以生成斐波那契序列及类似形式的序列。（注：此代码修改自 [*https://oreil.ly/EHjOb*](https://oreil.ly/EHjOb)）

示例 9-3. FibonacciLib：一个存在缺陷的自定义库实现
```Solidity
1 // library contract - calculates Fibonacci-like numbers
2 contract FibonacciLib {
3     // initializing the standard Fibonacci sequence
4     uint256 public start;
5     uint256 public calculatedFibNumber;
6
7     // modify the zeroth number in the sequence
8     function setStart(uint256 _start) public {
9         start = _start;
10     }
11
12     function setFibonacci(uint256 n) public {
13         calculatedFibNumber = fibonacci(n);
14     }
15
16     function fibonacci(uint256 n) internal view returns (uint) {
17         if (n == 0) return start;
18         else if (n == 1) return start + 1;
19         else return fibonacci(n - 1) + fibonacci(n - 2);
20     }
21 }
```
该库提供了一个可以生成序列中第 $n$ 个斐波那契数的函数。它允许用户修改序列的起始数字（start），并计算这一新序列中的第 $n$ 个类斐波那契数。现在，让我们考虑一个利用该库的合约：

```Solidity
contract FibonacciBalance {
    address public fibonacciLibrary;
    // the current Fibonacci number to withdraw
    uint256 public calculatedFibNumber;
    // the starting Fibonacci sequence number
    uint256 public start = 3;
    uint256 public withdrawalCounter;
    // the Fibonacci function selector
    bytes4 constant fibSig = bytes4(keccak256("setFibonacci(uint256)"));
    // constructor - loads the contract with ether
    constructor(address _fibonacciLibrary) payable {
        fibonacciLibrary = _fibonacciLibrary;
    }
    function withdraw() public {
        withdrawalCounter += 1;
        // calculate the Fibonacci number for the current withdrawal user-
        // this sets calculatedFibNumber
        (bool success, ) = fibonacciLibrary.delegatecall(
            abi.encodeWithSelector(fibSig, withdrawalCounter)
        );
        require(success, "Delegatecall failed");
        payable(msg.sender).transfer(calculatedFibNumber * 1 ether);
    }
    // allow users to call Fibonacci library functions
    fallback() external {
        (bool success, ) = fibonacciLibrary.delegatecall(msg.data);
        require(success, "Delegatecall failed");
    }
}
```
该合约允许参与者从合约中提取以太币，提取金额等于该参与者取款顺序对应的斐波那契数——也就是说，第一位参与者得到 1 ETH，第二位同样得到 1 ETH，第三位得到 2 ETH，第四位得到 3 ETH，第五位得到 5 ETH，依此类推（直到合约余额少于要提取的斐波那契数）。

合约中有几个元素可能需要解释。首先是一个看起来很有趣的变量：fibSig。它保存了字符串 "setFibonacci(uint256)" 的 Keccak-256 哈希的前 4 个字节。这被称为函数选择器（[Function Selector](https://oreil.ly/u9uaH)），被放入 calldata 中以指定调用智能合约的哪个函数。它在第 21 行的 delegatecall 函数中被使用，用以指定我们希望运行 setFibonacci(uint256) 函数。delegatecall 的第二个参数是我们传递给该函数的参数。其次，我们假设 FibonacciLib 库的地址在构造函数中已被正确引用。

你能发现这个合约中的错误吗？如果你部署这个合约，填入以太币并调用 withdraw，它很可能会执行失败（Revert）。

你可能已经注意到，状态变量 start 同时出现在库合约和主调用合约中。在库合约中，start 用于指定斐波那契数列的起始值并被设为 0；而在调用合约中，它被设为 3。你可能还注意到，FibonacciBalance 合约的回退函数允许将所有调用转发给库合约，从而允许调用库合约的 setStart 函数。回想一下，由于我们保留了合约的状态（上下文），看起来这个函数似乎允许你修改本地 FibonacciBalance 合约中 start 变量的状态。如果真是这样，这将允许你提取更多的以太币，因为最终生成的 calculatedFibNumber 取决于 start 变量。但实际上，setStart 函数并不能（也无法）修改 FibonacciBalance 合约中的 start 变量。该合约底层的漏洞远比修改 start 变量严重得多。

在讨论实际问题之前，我们先绕道了解一下状态变量在合约中是如何存储的。状态变量（或称存储变量，即在不同交易中持久存在的变量）会按照在合约中引入的顺序依次放入**槽位**（Slots）中（这里存在一些复杂性，请参考 [Solidity 文档](https://oreil.ly/LzV7L)以获得更透彻的理解）。

以库合约为例，它有两个状态变量：start 和 calculatedFibNumber。第一个变量 start 存储在合约存储的 slot[0]（即第一个槽位）。第二个变量 calculatedFibNumber 被放在下一个可用的存储槽位 slot[1]。函数 setStart 接收输入并将 start 设置为该输入值。因此，该函数的作用是将 slot[0] 设置为我们在 setStart 函数中提供的任何输入。同样，setFibonacci 函数将 calculatedFibNumber 设置为 fibonacci(n) 的结果。这同样只是简单地将存储 slot[1] 设置为 fibonacci(n) 的值。

现在，让我们看看 FibonacciBalance 合约。此时的存储 slot[0] 对应的是 fibonacciLibrary 地址，而 slot[1] 对应的是 calculatedFibNumber。漏洞正是在这种错误的映射中产生的：delegatecall 会保留合约上下文。这意味着通过 delegatecall 执行的代码将作用于调用方合约的状态（即存储）。

请注意，在第 21 行的 withdraw 函数中，我们执行了 fibonacciLibrary.delegatecall(fibSig, withdrawalCounter)。这调用了 setFibonacci 函数，如前所述，该函数会修改存储 slot[1]，在当前的上下文中即 calculatedFibNumber。这符合预期（即执行后 calculatedFibNumber 被修改了）。然而，回想一下 FibonacciLib 库合约中的 start 变量位于存储 slot[0]，而这在当前合约中正是 fibonacciLibrary 的地址。这意味着 fibonacci 函数将给出意想不到的结果。这是因为它引用了 start (slot[0])，而在当前调用上下文中，该槽位是 fibonacciLibrary 的地址（当被解释为 uint 时，通常是一个巨大的数值）。因此，withdraw 函数很可能会执行失败（Revert），因为合约中不太可能有 uint(fibonacciLibrary) 这么大数量的以太币——而这正是 calculatedFibNumber 将返回的值。

更糟糕的是，FibonacciBalance 合约允许用户通过第 27 行的回退函数调用 fibonacciLibrary 的所有函数。正如我们之前讨论的，这包括 setStart 函数。我们讨论过，该函数允许任何人修改或设置存储 slot[0]。在这种情况下，存储 slot[0] 就是 fibonacciLibrary 的地址。因此，攻击者可以创建一个恶意合约，将该地址转换为 uint256（这在 Python 中可以很容易地使用 `int('<address>', 16)` 完成），然后调用 setStart(<攻击合约地址的整数形式>)。这将把 fibonacciLibrary 更改为攻击合约的地址。此后，每当用户调用 withdraw 或回退函数时，恶意合约就会运行（它可以窃取合约的全部余额），因为我们已经修改了 fibonacciLibrary 的实际地址。

这样一个攻击合约的示例如下：
```Solidity
contract Attack {
    uint256 private storageSlot0; // corresponds to fibonacciLibrary
    uint256 private storageSlot1; // corresponds to calculatedFibNumber
    // fallback - this will run if a specified function is not found
    fallback() external {
        storageSlot1 = 0; // we set calculatedFibNumber to 0, so if withdraw
        // is called we don’t send out any ether
        payable(<attacker_address>).transfer(this.balance); // we take all the ether
    }
}
```
请注意，这个攻击合约通过更改存储 slot[1] 来修改 calculatedFibNumber。原则上，攻击者可以根据自己的选择修改任何其他存储槽位，从而对该合约发起各种攻击。我们鼓励你将这些合约放入 Remix 中，通过这些 delegatecall 函数尝试不同的攻击合约和状态更改。

同样重要的是，当我们说 delegatecall 是保留状态（State Preserving）时，我们指的并不是合约中的变量名，而是这些变量名所指向的实际存储槽位。正如你从这个例子中所看到的，一个简单的错误就可能导致攻击者劫持整个合约及其中的以太币。

#### 预防技术
Solidity 提供了 library 关键字用于实现库合约（详见官方文档）。这确保了库合约是**无状态（Stateless）且不可自毁（Non-self-destructible）**的。强制库合约保持无状态，可以减轻本节所展示的存储上下文带来的复杂性。无状态库还能防止攻击者直接修改库的状态，进而影响那些依赖该库代码的合约。作为一条通用准则，当你使用 DELEGATECALL 时，请务必仔细关注库合约和调用合约之间可能存在的调用上下文，并尽可能构建无状态库。

#### 真实案例：Parity 多签钱包（第二次攻击）
第二次 Parity 多签钱包黑客事件是一个典型的案例，展示了即便编写良好的库代码，如果在其预期的上下文之外运行，也可能被恶意利用。关于这次攻击有很多精彩的解释，例如《Parity 多签再次被黑》。作为参考补充，让我们来探究一下那些被利用的合约。

由于该漏洞发生在七年前，以下代码片段已根据最新的 Solidity 语法进行了更新，以便于阅读和理解。

库合约如下：
```Solidity
1 contract WalletLibrary is WalletEvents {
2
3   ...
4
5   // throw unless the contract is not yet initialized.
6   modifier only_uninitialized { if (m_numOwners > 0) revert(); _; }
7
8   // constructor - just pass on the owner array to multiowned and
9   // the limit to daylimit
10   function initWallet(address[] memory _owners, uint256 _required, uint256
11        _daylimit) public only_uninitialized {
12     initDaylimit(_daylimit);
13     initMultiowned(_owners, _required);
14   }
15
16   // kills the contract sending everything to `_to`.
17   function kill(address _to) onlymanyowners(keccak256(msg.data)) external {
18     selfdestruct(_to);
19   }
20
21   ...
22
23 }
```
以下是钱包合约：
```Solidity
1 contract Wallet is WalletEvents {
2
3   ...
4
5   // METHODS
6
7   // gets called when no other function matches
8   fallback() external payable {
9     // just being sent some cash?
10     if (msg.value > 0)
11       Deposit(msg.sender, msg.value);
12     else if (msg.data.length > 0)
13       _walletLibrary.delegatecall(msg.data);
14   }
15
16   ...
17
18   // FIELDS
19   address constant _walletLibrary =
20     0xcafecafecafecafecafecafecafecafecafecafe;
21 }
```
请注意，Wallet 合约本质上通过 delegatecall 将所有调用转发给 WalletLibrary 合约。代码片段中的常量 _walletLibrary 地址充当了实际部署的 WalletLibrary 合约的占位符（该合约当时的地址为 0x863DF6BFa4469f3ead0bE8f9F2AAE51c91A907b4）。

这些合约的设计初衷是拥有一个简单、低成本且易于部署的 Wallet 合约，其代码库和主要功能都集中在 WalletLibrary 合约中。不幸的是，WalletLibrary 合约本身也是一个合约，并维护着自己的状态。你能看出为什么这会成为一个问题吗？

向 WalletLibrary 合约本身发送调用是可行的。具体来说，WalletLibrary 合约可以被初始化并被拥有。事实上，当时一名用户确实这么做了：他调用了 WalletLibrary 合约上的 initWallet 函数，从而成为了该库合约的所有者（Owner）。随后，该用户又调用了 kill 函数。由于该用户已是库合约的所有者，权限检查（Modifier）通过，库合约执行了自毁（Self-destruct）。由于当时所有存在的 Wallet 合约都指向这个库合约，且没有更改该引用的方法，它们所有的功能（包括提取以太币的能力）都随着 WalletLibrary 合约的消失而丧失了。结果，所有此类 Parity 多签钱包中的以太币瞬间变得不可访问或永久无法找回。

> [!Note]
> 这位攻击者随后在 GitHub 上[现身](https://oreil.ly/VVgnm)，留下了一条令人难忘的评论：“我一不小心杀掉了它。”（I accidentally killed it.）他声称自己是以太坊的新手，当时只是在尝试研究智能合约。

### 熵之错觉（Entropy Illusion）
以太坊区块链上的所有交易都是确定性的状态转换操作。这意味着每一笔交易都以一种可计算且无不确定性的方式修改以太坊生态系统的全局状态。这产生了一个根本性的影响：在以太坊内部不存在熵或随机性的来源。在早期，寻找一种去中心化的方式来创造随机性是一个巨大的挑战。但经过多年的发展，我们已经开发出了一些可靠的方案来解决这个问题。

#### 漏洞原理
当开发者在以太坊上构建智能合约时，往往需要一个随机性来源，无论是用于游戏、抽奖，还是其他需要不可预测性的功能。挑战在于，以太坊作为一个区块链，本质上是确定性的：每个节点必须达到完全相同的结果才能维持共识。因此，引入真正的随机性需要一点“创意”。

许多开发者采取的一种方法是使用区块变量（如区块哈希、时间戳或区块编号）作为生成随机数的种子。这些值看起来似乎是随机的，但实际上它们受控于提议当前区块的验证者（Validator）。例如，假设一个 DApp 根据下一个区块哈希的结尾是否为偶数来决定游戏胜负。验证者可以操纵这一过程：如果他们即将提议一个区块，但发现哈希不符合他们预期的结果，他们可以（例如）通过更改交易顺序来以有利的方式改变区块哈希。

利用区块变量派生随机数时，验证者的操纵并非唯一风险。其他智能合约也能感知到这些区块变量的值，这使得它们能够仅在结果有利时才与存在漏洞的合约进行交互。

#### 预防技术
与过去相比，以太坊开发者现在拥有可靠的随机数生成方法：PREVRANDAO 和可验证随机函数（VRFs）。

VRF 是一种加密证明，旨在确保生成的随机数公平且无偏见。目前已有多个服务商（如 Chainlink）支持 VRF。VRF 会生成一个随机数以及一个验证其公平性的证明。任何人都可以验证这一证明，从而确保随机性的安全性。VRF 已成为智能合约中安全获取随机数的标准去中心化解决方案。

另一个可靠的选择是 PREVRANDAO 操作码，它是随着以太坊转向权益证明（PoS）机制而引入的。该操作码用于获取 PREVRANDAO 值，该值源自 Randao 流程——这是 PoS 区块生产中不可或缺的组成部分。本质上，Randao 是验证者们通过各自贡献一部分数据来集体生成随机数的过程。PREVRANDAO 是上一个区块这一过程的结果，它是一个可靠的随机性来源。它的可信度在于，操纵 PREVRANDAO 值需要攻破大量的验证者，这在实践中是不可行且在经济上不划算的。开发者可以在合约中使用此值，但应记住 PREVRANDAO 代表的是上一个区块的值，而该值已经是公开已知的。为了防止该值在承诺时被预测，智能合约应改为承诺使用未来某个区块的 PREVRANDAO 值。这样，在做出承诺时，该随机值尚未产生。

> [!Warning]
> 如果攻击者控制了被分配到每个 Epoch（纪元）最后几个 Slot（槽位）的提案者（Proposers），Randao 就可能被操纵。要决定 PREVRANDAO 是否是智能合约生成随机数的可靠选择，你应该仔细权衡其被操纵的成本与收益。尽管干预 Randao 的成本非常高昂，但如果你的合约涉及价值巨大的资产，使用去中心化预言机解决方案（如 VRF）会更加安全。

由于 PREVRANDAO 和 VRF 等解决方案已有详尽的文档说明且易于获取，如今已很难见到开发者再使用不安全的区块变量作为随机性来源。然而，当开发者为了走捷径或者不了解这些工具时，错误仍然会发生。

#### 真实案例：Fomo3D

Fomo3D 是一款曾风靡一时的以太坊彩票游戏。玩家购买“钥匙”（Keys）来延长倒计时器，目标是成为计时器归零前的最后一位购买者，从而赢取巨额奖池。该游戏包含一个“空投”（Airdrop）功能，但其随机性实现非常糟糕，如下面的代码所示：

```Solidity
function airdrop()
        private
        view
        returns(bool)
    {
        uint256 seed = uint256(keccak256(abi.encodePacked(

            (block.timestamp).add
            (block.difficulty).add
            ((uint256(keccak256(abi.encodePacked
            (block.coinbase)))) / (block.timestamp)).add
            (block.gaslimit).add
            ((uint256(keccak256(abi.encodePacked
            (msg.sender)))) / (block.timestamp)).add
            (block.number)
        )));
        if((seed - ((seed / 1000) * 1000)) < airDropTracker_) {
            return(true);
        } else {
            return(false);
        }
    }
```
恶意合约可以预先知道用于计算种子的各个数值，从而使其能够仅在结果中奖的情况下才触发 airdrop 函数。因此，该合约被攻击者利用也就不足为奇了。


### 未检查的 CALL 返回值

在 Solidity 中有多种执行外部调用的方法。向外部账户发送以太币通常通过 transfer 方法完成。此外，也可以使用 send 函数；而对于更通用的外部调用，可以直接在 Solidity 中使用 CALL 操作码。call 和 send 函数会返回一个布尔值（Boolean），指示调用成功与否。因此，这两个函数存在一个简单的限制：如果外部调用（由 call 或 send 发起）失败，执行这些函数的交易不会撤销（Revert）；相反，函数只会简单地返回 false。一个常见的错误是：开发者期望在外部调用失败时发生撤销，但却没有检查其返回值。

#### 漏洞原理
请看示例 9-4。

示例 9-4. 存在漏洞的 Lotto（彩票）合约

```Solidity
1 contract Lotto {
2
3     bool public payedOut;
4     address public winner;
5     uint256 public winAmount;
6
7     // ... extra functionality here
8
9     function sendToWinner() public {
10         require(!payedOut);
11         payable(winner).send(winAmount);
12         payedOut = true;
13     }
14
15     function withdrawLeftOver() public {
16         require(payedOut);
17         payable(msg.sender).send(address(this).balance);
18     }
19 }
```
这代表了一个类似彩票的合约，其中**中奖者（winner）**会获得 winAmount 数量的以太币，通常还会剩下一小部分供任何人提取。漏洞存在于第 11 行，那里使用了 send 但没有检查响应结果。在这个简单的例子中，如果中奖者的交易失败（无论是由于 Gas 耗尽，还是因为中奖者是一个在回退函数中故意抛出异常的合约），无论以太币是否成功发送，payedOut 都会被设置为 true。在这种情况下，任何人都可以通过 withdrawLeftOver 函数提取走原本属于中奖者的奖金。

#### 预防技术

第一道防线始终是检查 send 函数和底层调用（low-level calls）的返回值，绝无例外。现如今，任何静态分析工具都会标记出这一问题，使其很难被忽视。

在发送以太币时，我们需要仔细考虑使用哪种方法。如果我们希望交易在失败时自动撤销，transfer 似乎很有吸引力，因为它默认处理了失败情况。但由于 send 和 transfer 都仅转发 2300 个 Gas 单位，一旦接收方（无论是合约，还是在 EIP-7702 之后甚至可能包含逻辑的外部账户 EOA）拥有任何回退逻辑，这些方法都很容易失败。鉴于这种不断变化的上下文，更安全且更灵活的方法是改用 call，显式检查其返回值，并据此管理错误。这让我们能够完全控制 Gas 的转发，并保持合约与更多样化的接收方兼容。

#### 真实案例：Etherpot 与 King of the Ether

[Etherpot](https://oreil.ly/_-iC-) 是一个智能合约彩票，与示例 9-4 中的合约非常相似。该合约的失败主要归因于对区块哈希（Block Hashes）的错误使用（只有最近的 256 个区块哈希是可用的；参见“预定义全局变量和函数”）。然而，该合约同样遭受了未检查调用返回值的影响。

请看示例 9-5 中的 cash 函数：同样，以下代码片段已更新，以反映最新 Solidity 的语法。

示例 9-5. Lotto.sol 代码片段
```Solidity
1 function cash(uint256 roundIndex, uint256 subpotIndex) public {
2    uint256 subpotsCount = getSubpotsCount(roundIndex);
3    if(subpotIndex>=subpotsCount)
4        return;
5    uint256 decisionBlockNumber = getDecisionBlockNumber(roundIndex,subpotIndex);
6    if(decisionBlockNumber>block.number)
7        return;
8    if(rounds[roundIndex].isCashed[subpotIndex])
9        return;
10    //Subpots can only be cashed once. This is to prevent double payouts
11    address winner = calculateWinner(roundIndex,subpotIndex);
12    uint256 subpot = getSubpot(roundIndex);
13    payable(winner).send(subpot);
14    rounds[roundIndex].isCashed[subpotIndex] = true;
15    //Mark the round as cashed
16 }
```
请注意第 13 行，send 函数的返回值没有被检查，而紧接着的下一行就设置了一个布尔值，指示中奖者已经获得了资金。这个漏洞可能导致这样一种状态：中奖者并未收到以太币，但合约状态却显示中奖者已被支付。

一个更严重的此类漏洞发生在 [King of the Ether](https://oreil.ly/4zLMH)（以太之王）合约中。关于该合约有一篇极其出色的事后分析（[Postmortem](https://oreil.ly/U8ckx)），详细描述了如何利用一个未检查且失败的 send 来攻击合约。

#### ERC-20 案例

在 Solidity 中处理 ERC-20 代币时，仅仅检查代币转账的返回值并不足以确保交互安全。这是因为并非所有 ERC-20 代币都严格遵守 ERC-20 标准，尤其是较早期的代币。某些代币在转账完成后会返回一个布尔值，而不是在操作失败时直接撤销（Revert）或抛出异常。还有一些代币可能根本不返回任何值，这导致使用标准方法与之交互时会出现歧义行为。**泰达币（USDT）**就是一个广泛使用但未完全符合 ERC-20 标准的典型例子。

为了缓解这一问题，我们使用像 OpenZeppelin 的 SafeERC20 这样的库。该库对标准的 ERC-20 操作（如 transfer、transferFrom 和 approve）进行了封装，以优雅地处理这些差异。如果某个代币返回 false，该库会确保交易撤销；如果代币没有返回值，只要没有发生撤销，该库就会认为操作已成功。

### 竞争条件与抢跑（Front-Running）
为了真正掌握这一漏洞，让我们简要回顾一下以太坊交易的工作原理。当我们发送一笔交易时，它会被广播到节点网络并放置在 Mempool（内存池）中，这相当于待处理交易的“候补室”。随后，验证者从内存池中挑选这些交易来构建区块。区块内的交易按特定顺序顺序执行，由于每笔交易都会改变区块链的全局状态，一笔交易的结果可能会因其在区块中的位置而异。这种交易排序至关重要，因为它会显著影响交易执行的结果。

> [!Note]
> 在实践中，控制交易在区块中的位置主要取决于支付（付费）。最初，你只需提供更高的 Gas 价格（Gas Price）就能影响排序。而今天，得益于实现了“构建者-提议者分离”（PBS，目前尚未成为以太坊原生部分）的 Flashbots 基础设施，用户可以提交特定顺序的交易捆绑包（Bundles），并通过链下中继系统竞价以换取被包含在区块内的机会。这些过程——包括传统的基于内存池的系统和新的基于构建者的系统——都在第 6 章中有更详细的介绍。

#### 漏洞原理
抢跑（Front-running）是指利用这种顺序执行机制，通过在区块中插入其他交易来使抢跑者获益的行为。本质上，有人会盯梢那些可能影响市场或特定合约的待处理交易，然后提交自己的交易，力求在原交易之前被处理。通过这种方式，他们可以利用待处理交易中的信息获利，而这通常会损害原发送者的利益。我们的代码必须考虑到这种动态变化，并设计成能够抵御区块内交易顺序变更的影响，这一点至关重要。

让我们通过一个简单的例子来看看这是如何运作的。请参考示例 9-6 中的合约。
示例 9-6. FindThisHash：一个易受抢跑攻击的合约
```Solidity
contract FindThisHash {
    bytes32 constant public hash =
      0xb5b5b97fafd9855eec9b41f74dfb6c38f5951141f9a3ecd7f44d5479b630ee0a;
    constructor() payable {} // load with ether
    function solve(string memory solution) public {
        // If you can find the pre-image of the hash, receive 1000 ether
        require(hash == keccak256(abi.encodePacked(solution)));
        payable(msg.sender).transfer(1000 ether);
    }
}
```
假设这个合约里有 1,000 个以太币。谁能找到 SHA-3 哈希值 0xb5b5...ee0a 的原像（Preimage），谁就可以提交答案并取走这 1,000 个以太币。假设一名用户发现答案是 Ethereum!。他调用 solve 函数并传入 Ethereum! 作为参数。不幸的是，一名攻击者在内存池中发现了这笔交易，验证了其有效性，随后提交了一笔在区块中优先级更高的相同交易。由于攻击者的交易会被优先处理，原用户的交易将会失败（Revert）。

#### 预防技术

抢跑漏洞可能以各种形式出现，通常取决于智能合约或协议的具体逻辑。每当一个操作可以通过交易排序被恶意利用时，就存在抢跑漏洞。因此，解决方案通常是针对具体问题量身定制的。例如，自动做市商（AMM）协议通过允许用户设置在兑换过程中必须收到的最小代币数量（即滑点设置）来解决此问题。虽然这不能完全阻止抢跑，但它严重限制了攻击者可以榨取的潜在利润，减轻了损失并保护用户免受极端滑点的影响。

另一种通用技术是使用承诺-揭晓方案（Commit-reveal scheme）。在这种方法中，用户首先提交包含隐藏信息的交易，通常以哈希值的形式表示（承诺阶段）。一旦该交易被包含进区块，用户再提交第二笔交易来揭晓实际数据（揭晓阶段）。这种方法能有效防止抢跑，因为在攻击者能够采取行动之前，他们无法看到初始交易的具体细节。然而，其代价是需要两笔独立的交易，这意味着更高的成本和额外的延迟。除了用户体验较差之外，两笔交易之间所需的延迟在对时间敏感的应用中也可能是一个实际限制。

#### 真实案例：AMM 与 minAmountOut
让我们探究一个常见的现实世界抢跑漏洞。当集成自动做市商（AMM）协议的智能合约在执行代币兑换（Swap）时，如果未设置收到的最小代币数量，就会导致这些兑换极易受到抢跑攻击。如果最小接收数量设置不当（或设置得太低），该兑换交易就会变得脆弱，容易遭受三明治攻击（Sandwich Attack）——这是抢跑攻击的一种特定类型。

三明治攻击的演变过程如下：抢跑者监控内存池中的待处理交易，并发现我们那笔未强制执行最小输出金额的兑换交易。攻击者会在我们的兑换之前提交一笔买入交易，人为地推高代币价格。接着，我们的交易在这一虚高的价格下成交，导致我们获得的代币数量低于预期。最关键的是，我们的交易会进一步推高价格。紧接着，攻击者以这个更高的价格卖出他们的代币，使价格回落，并从我们交易创造的价格差中获利。这种策略将我们的交易“夹”在他们的两笔交易之间，因此得名“三明治攻击”。

为了修复这个问题，集成 AMM 的智能合约需要从可信来源（如预言机，甚至是基于时间加权平均价格 TWAP 的预言机）获取真实的资产价格，然后在执行兑换时计算并强制执行精确的最小输出金额（minAmountOut）。

### 拒绝服务攻击 (DoS)

这一类别非常广泛，但从根本上说，它包含了那些用户可以使合约或其部分功能在一段时间内、甚至在某些情况下永久性失效的攻击。这可能会将资金永远困在这些合约中，正如“真实案例：Parity 多签钱包（第二次黑客攻击）”中所描述的那样。

#### 漏洞原理

合约失效的方式有多种。这里我们重点介绍几种不太明显、但会导致 DoS 漏洞的 Solidity 编码模式。

#### 遍历受外部操纵的映射或数组

这种模式通常出现在合约所有者希望通过类似 distribute 的函数向投资者分发代币时，如示例 9-7 中的合约所示。

示例 9-7. DistributeTokens 合约
```Solidity
1 contract DistributeTokens {
2     address public owner; // gets set somewhere
3     address[] investors; // array of investors
4     uint[] investorTokens; // the amount of tokens each investor gets
5
6     // ... extra functionality, including transfertoken()
7
8     function invest() public payable {
9         investors.push(msg.sender);
10         investorTokens.push(msg.value * 5); // 5 times the wei sent
11         }
12
13     function distribute() public {
14         require(msg.sender == owner); // only owner
15         for(uint256 i = 0; i < investors.length; i++) {
16             // here transferToken(to,amount) transfers "amount" of
17             // tokens to the address "to"
18             transferToken(investors[i],investorTokens[i]);
19         }
20     }
21 }
```
请注意，该合约中的循环是在一个可以被人为膨胀的数组上运行的。攻击者可以创建大量的用户账户，使得 investors 数组变得异常庞大。风险不仅在于循环本身，还在于循环内部操作（如 transferToken 或其他逻辑）所产生的累计 Gas 成本。每一个额外的迭代都会增加总 Gas 消耗，如果数组变得足够大，完成该循环所需的 Gas 可能会超过区块 Gas 限制（Block Gas Limit）。到那时，distribute 函数实际上将无法使用。

> [!Note]
> 这种类型的 DoS 不仅限于改变状态的函数。如果只读（view）函数在大型数组上进行循环，也可能变得无法访问。虽然调用它们在链上不消耗 Gas，但 RPC 节点会对 eth_call 的执行实施各自的 Gas 上限限制。因此，如果一个 view 函数运行的逻辑足以超过这些限制，RPC 调用将会失败。

#### 基于外部调用推进状态

有些合约在编写时，要求必须向某个地址发送以太币或等待外部来源的输入才能推进到新状态。当外部调用失败或因外部原因被阻止时，这些模式会导致 DoS。以发送以太币为例，用户可以创建一个不接收以太币的合约。如果一个合约要求必须成功发送以太币才能进入新状态，那么由于无法向该不接收资产的合约发送以太币，原合约将永远无法达到新状态。

#### 意外问题

DoS 问题可能以意想不到的方式出现，且并不总是涉及恶意攻击。有时，合约功能仅因不可预见事件就会中断。例如，如果智能合约依赖所有者的私钥来调用特定的特权函数，而该私钥丢失或泄露，麻烦就大了。没有该私钥，这些关键函数将永久无法访问，可能导致整个合约运行停滞。想象一个 ICO 合约，所有者必须调用某个函数来结束销售；如果私钥丢失，无人能调用它，代币将永远被锁定。

另一个意外中断的例子是，以太币在合约不知情或非自愿的情况下被发送至合约。可以使用 selfdestruct（现已弃用）方法，甚至在合约部署到其预定地址之前发送以太币，从而将以太币“强制”打入合约。如果一个合约假设它通过自身函数完全控制了所有接收以太币的账目，它可能不知道如何处理这些不请自来的资金，从而导致非预期行为。这就像银行账户里突然多了一笔你没预料到的钱——有时这很不错，但也可能意味着你的账户余额对不上了，任何依赖该精确数字的系统都会开始出问题。

#### 预防技术

由于 DoS 问题表现形式多样，解决方案通常也因情况而异。

针对容易触发区块 Gas 限制的长列表问题（这是一种非常普遍的情况），我们可以提供一些处理建议。在第一个例子中，合约不应遍历那些可被外部用户人为操纵的数据结构。推荐使用提款模式（Withdrawal pattern），即由每位投资者分别调用 withdraw 函数来独立领取代币（即 Pull-over-Push 模式）。对于必须遍历长列表的函数，一个好的解决方案是实现**分页**（Pagination）功能。

应对 DoS 的通用方案是尽可能深入地研究可能出错的环节，并实施相应的保护措施。

#### 真实案例：ZKsync Era Gemholic 资金锁定

正如我们刚才所说，最令人意想不到的错误也可能导致智能合约的 DoS 问题。一个近期的案例涉及 Gemholic，该项目在以太坊 L2 方案 ZKsync Era 上部署了智能合约。Gemholic 面临的一个重大问题是，它无法提取在代币销售中筹集的 921 ETH（约 170 万美元）。根源何在？该智能合约依赖于 transfer() 函数，而 ZKsync Era 并不支持该函数。尽管 ZKsync Era 与大部分 EVM 功能兼容，但它并非完全的 EVM 等效（EVM equivalent），这意味着某些功能（如 transfer()）在 ZKsync Era 上的表现与在以太坊主网上不同。这种不兼容性导致 Gemholic 的资金被封死，因为合约无法按预期提取以太币。幸运的是，ZKsync 团队介入并开发了他们所称的“优雅解决方案”来解锁资金，使 Gemholic 能够重新获得这些资金。遗憾的是，这一“优雅解决方案”的具体细节尚未公开。

### 浮点数与精度

截至目前，Solidity 的 v0.8.29 版本尚未完全支持定点数（Fixed-point）和浮点数（Floating-point）。这一设计选择源于区块链对**确定性**（Determinism）的核心需求：网络中的每个节点必须从相同的输入中得出完全一致的结果，以维护共识。遗憾的是，浮点运算在不同的硬件架构之间本质上是非确定性的，相同的计算可能会产生极其微小的差异。

由于区块链应用需要绝对的确定性来防止网络分叉并确保安全，Solidity 强制开发者使用整型（Integer types）来实现浮点数的表示。虽然这种方法操作起来更为繁琐，且如果实现不当容易出错，但它确保了金融计算和智能合约逻辑在网络中的所有节点上都能产生完全相同的结果。

#### 漏洞原理

Solidity 尚未完全支持定点数。虽然可以声明定点数，但无法对其进行赋值，这意味着开发者必须使用标准的整型数据类型来实现自己的定点数逻辑。在这一过程中，开发者可能会遇到许多陷阱。我们将在本节中重点介绍其中一部分。让我们从一个代码示例开始（示例 9-8）。

示例 9-8. FunWithNumbers 合约

```Solidity
1 contract FunWithNumbers {
2    uint256 constant public tokensPerEth = 10;
3    uint256 constant public weiPerEth = 1e18;
4    mapping(address => uint) public balances;
5
6    function buyTokens() public payable {
7        // convert wei to eth, then multiply by token rate
8        uint256 tokens = msg.value/weiPerEth*tokensPerEth;
9        balances[msg.sender] += tokens;
10    }
11
12    function sellTokens(uint256 tokens) public {
13        require(balances[msg.sender] >= tokens);
14        uint256 eth = tokens/tokensPerEth;
15        balances[msg.sender] -= tokens;
16        payable(msg.sender).transfer(eth*weiPerEth);
17    }
18 }
```
这个简单的代币买卖合约存在一些显而易见的问题。虽然买卖代币的数学公式在逻辑上是正确的，但由于缺乏浮点数支持，会导致错误的结果。例如，在第 8 行购买代币时，如果输入的价值小于 1 ether，初始的除法运算结果将为 0，从而导致最终乘法的结果也为 0（例如：200 wei 除以 $10^{18}$ 的 weiPerEth 等于 0）。同样，在卖出代币时，任何少于 10 个的代币数量也会导致结果为 0 ether。事实上，这里的舍入总是向下取整的，因此卖出 29 个代币将得到 2 ether（29 tokens / 10 tokensPerEth = 2.9，向下取整后为 2）。

该合约的问题在于其精度仅精确到最近的 ether（即 $10^{18}$ wei）。在处理需要更高精度的 [ERC-20](https://oreil.ly/YzzyU) 代币小数位时，这会变得非常棘手。在实际情况中，精度损失看似微不足道，但很容易被放大和利用。例如，**闪电贷**（Flash loans）允许攻击者在无需前期成本的情况下借入巨额资金，从而使利用微小的计算不一致性成为可能。

#### 预防技术

在智能合约中保持正确的精度至关重要，尤其是在处理反映经济决策的比率和汇率时。你应该确保所使用的任何比率或速率都允许分数中存在巨大的分子。例如，在我们的示例中使用了 tokensPerEth 这一速率，更好的做法是使用 weiPerTokens，这会是一个很大的数值。要计算对应的代币数量，我们可以执行 msg.value / weiPerTokens，这样会得到更精确的结果。

另一种策略是留心运算顺序。在我们的示例中，购买代币的计算公式是 msg.value / weiPerEth * tokenPerEth。请注意，除法发生在乘法之前。与某些语言不同，Solidity 保证按照代码书写的顺序执行运算。如果该示例先执行乘法再执行除法，即 msg.value * tokenPerEth / weiPerEth，则会获得更高的精度。

最后，在定义数字的任意精度时，一种好的做法是将数值转换为更高精度，完成所有数学运算，然后再转换回输出所需的精度。通常使用 uint256 类型，因为它们在 Gas 使用上是最优的；其范围提供了约 60 个数量级，其中一部分可以专门用于提升数学运算的精度。在 Solidity 内部保持所有变量的高精度，并在外部应用中转换回低精度是更好的做法。这本质上就是 ERC-20 代币合约中 decimals 变量的工作原理：当我们在 MetaMask 上发送 1,000 USDT 时，实际上发送的是 1,000,000,000 单位的 USDT，即 1,000 乘以 USDT 的精度倍数（1e6）。

为了进一步了解如何处理高精度数学运算，让我们引入 Wad 和 Ray 数学。Wad 代表具有 18 位小数精度的数字，这与以太币等常见 ERC-20 代币的 18 位精度完美对齐，使其成为表示代币余额的理想选择，确保了计算过程中的准确性。另一方面，Ray 拥有更高的 27 位小数精度，适用于计算非常接近于零的比率。第一个 Solidity 定点数数学库 DS-Math 为处理这些高精度数字提供了框架。

MakerDAO 的开发者最初是专门为项目需求创建了 Wad 和 Ray。鉴于以太币 18 位小数的标准（且大多数 ERC-20 代币也遵循此约定，尽管存在不少例外），Wad 非常适合主要金融单位，而 Ray 则专门用于需要精确微调比例的情况。虽然 DS-Math 开创了这种方法，但如今已有更多库可用于精确的 Solidity 数学运算，例如 Aave 的 WadRayMath、Solmate 的 FixedPointMathLib 以及 OpenZeppelin 的 Math 库。

#### 真实案例：ERC-4626 通胀攻击

我们现在将通过 OpenZeppelin ERC-4626 实现的一个简化版本，来观察一种在现实世界中经常被利用的精度损失漏洞。ERC-4626 是一种代币化金库标准，允许用户将资产（如 USDT）存入金库，并获得代表其在金库资产中所占份额的“份额代币”（Shares）。示例 9-9 是我们正在研究的合约的简化版本。

示例 9-9. OpenZeppelin 原始 ERC-4626 实现的简化版

```Solidity
1 abstract contract ERC4626 is ERC20, IERC4626 {
2    using Math for uint256;
3    IERC20 private immutable _asset;
4
5    constructor(IERC20 asset_) {
6        _asset = asset_;
7    }
8
9    function totalAssets() public view returns (uint256) {
10        return _asset.balanceOf(address(this));
11    }
12    function deposit(address receiver, uint256 assets) public {
13        SafeERC20.safeTransferFrom(_asset, msg.sender, address(this), assets);
14        uint256 shares = _convertToShares(assets, Math.Rounding.Down);
15        _mint(receiver, shares);
16        emit Deposit(msg.sender, receiver, assets, shares);
17    }
18    function _withdraw(address receiver, uint256 assets) public {
19        uint256 shares = _convertToShares(assets, Math.Rounding.Up);
20        _burn(msg.sender, shares);
21        SafeERC20.safeTransfer(_asset, receiver, assets);
22        emit Withdraw(msg.sender, receiver, msg.sender, assets, shares);
23    }
24    function _convertToShares(uint256 assets, Math.Rounding rounding) internal view
      returns (uint256) {
25        uint256 supply = totalSupply();
26        return
27            (assets == 0 || supply == 0)
28                ? assets
29                : assets.mulDiv(supply, totalAssets(), rounding); // (assets * supply) /
                  totalAssets()
30    }
31    function _convertToAssets(uint256 shares, Math.Rounding rounding) public view returns
      (uint256) {
32        uint256 supply = totalSupply();
33        return
34            (supply == 0)
35                ? shares
36                : shares.mulDiv(totalAssets(), supply, rounding); // (shares * totalAssets())
                  / supply
37    }
38 }
```
现在，让我们看看攻击是如何展开的：一名密切关注新创建 ERC-4626 金库的攻击者发现了一个新金库。他们立即行动，存入极小额资产（仅 1 单位），为自己铸造 1 份份额（Share）。此时，金库总资产为 1，份额总供应量也为 1。

接下来是关键的隐蔽操作。攻击者等待另一名用户存入大额资金——假设是 1,000 USDT。但在该合法交易执行前，攻击者通过抢跑（Front-run），直接向金库合约转账 1,000 USDT。重要的是，攻击者不使用金库的 deposit 函数，而是直接调用 USDT.transfer()。这笔 1,000 USDT 的“捐赠”将金库的 totalAssets() 推高至 1000e6 + 1，而份额的 totalSupply() 仍保持为 1。请记住，1,000 USDT 实际上记作 1,000e6，即 1,000 乘以 USDT 的精度单位（1e6）。

当受害者的存款最终被处理时，智能合约尝试计算应为该用户铸造多少份额。记住计算份额的公式是：

$$(assets \times supply) / totalAssets()$$

在本例中，受害者存入 1,000e6 USDT，公式变为：

$$1,000e6 \times 1 / (1,000e6 + 1) = 0.999...$$

由于**向下取整**（Rounding-down）机制，结果变为 0 份份额。受害者存入了 1,000 USDT，却什么也没得到。

与此同时，依然持有那 1 份份额的攻击者现在可以销毁该份额，并提取金库的全部余额（即 2,000 USDT）。攻击者卷走了所有资金，而受害者空手而归。

> [!Note]
> OpenZeppelin 随后更新了其 ERC-4626 实现，通过引入虚拟偏移量（Virtual Offset）和小数位偏移量（Decimal Offset）来防止此类攻击。小数位偏移量增加了金库份额的小数位数，有助于最小化舍入误差，降低精度丢失攻击的获利空间。虚拟偏移量在汇率计算中加入了虚拟资产和份额，限制了攻击者操纵初始兑换率的能力，并保护金库免受“死份额”创建的影响。

### 价格操纵

资产定价的准确性对于 DeFi 协议的顺畅运行至关重要。这些系统依赖**价格预言机（Price Oracles）**来提供资产的即时价值。你可以将预言机想象成一种向智能合约提供现实世界信息的数据流。价格操纵攻击的目标正是这些预言机——也就是说，攻击的重点不在于智能合约的代码本身，而在于合约所依赖的数据。这种操纵可以显著改变 DeFi 协议的行为，创造出常态下并不存在的套利机会。其结果如何？攻击者可以利用该系统赚取巨额利润。

#### 漏洞原理

想象这样一个简单的场景：攻击者发现一个借贷协议依赖于一个不安全的预言机进行定价。通过操纵资产价格使其看起来低于实际价值，攻击者可以借入超出其应得额度的资产。随后，他们按真实市场价格卖出借来的资产，从而获利。该漏洞的根源在于协议依赖了可被操纵的链上价格指标来确定资产价格。这种操纵通常会通过**闪电贷（Flash loans）**被放大——这是一种必须在同一个交易区块内偿还的、即时的、无抵押贷款。

#### 预防技术

当我们需要确定价格时，最佳选择是使用 Chainlink、RedStone 或 Pyth 等去中心化预言机。由于这些预言机是去中心化的，攻击者需要控制网络中超过 50% 的节点才能实施破坏，这使得它们极难被攻破。然而，它们也有局限性，例如并非所有资产都有对应的喂价。在这种情况下，我们可以转而使用时间加权平均价格（TWAP）预言机。

TWAP 预言机基于链上数据并增加了一定的安全性。它们的工作原理是计算资产在特定时间段内（例如过去 5 分钟）的平均价格。通过在计算中排除当前区块，TWAP 预言机有效地抵御了闪电贷攻击。然而，对于资金极其雄厚的攻击者来说，TWAP 预言机并非完全免疫操纵。这里的关键在于调整周期长度：周期越长，攻击者操纵价格所需的资金就越多。但较长的周期也意味着 TWAP 价格与实际市场价格之间的偏差可能更大。因此，根据项目的具体需求和风险状况对 TWAP 进行精细调优至关重要。

无论我们使用哪种预言机，都不应盲目信任它提供的数据。定期将预言机数据与其他来源进行核实是良好的实践。例如，我们可以编写脚本对比预言机价格与其他来源的价格，并对任何显著差异进行预警。如果发现此类差异，可以暂停协议以防止进一步的问题。

#### 典型案例：依赖 AMM 链上数据

通常情况下，存在漏洞的预言机模块本身就是协议的一部分，正如我们在这个例子中看到的。一种常见的漏洞场景发生在智能合约直接从 Uniswap 等链上 AMM 协议获取资产价格时。想象一个拥有 4,000 USDC 和 1 ETH 储备的 Uniswap V2 资金池。某个智能合约可能会据此假设 1 ETH 价值 4,000 USDC。然而，如果推导出的价格被用于进一步的改变状态的操作，这种假设将变得非常危险。在这种情况下，攻击者可以利用闪电贷执行一笔大额兑换，改变资金池的平衡，从而改变 ETH 的推导价格。依赖此操纵价格的漏洞协议随后就会被攻击者榨取。

幸运的是，这种特定的攻击路径已经众所周知。虽然它不像以前那样频繁被利用，但仍出现在一些备受关注的事件中。例如，在 2025 年 5 月，Mobius Token 遭受攻击，损失达 210 万美元。尽管直接诱因是 mint 函数中一个错误的 $10^{18}$ 乘法逻辑，但该合约还包含另一个同样关键的漏洞：它依赖链上指标来计算 BNB/USDT 价格，从而暴露在操纵风险之下。即使没有那个数学 Bug，该合约也会在短时间内被攻破。你可能会纳闷，这样的代码是如何进入生产环境并承载如此高的总锁仓量（TVL）的。团队当时选择不公开合约源码，认为保持隐蔽能提供安全性——这再次提醒我们，“通过隐蔽实现安全”（Security through obscurity）是行不通的，尤其是在赌注如此之高的情况下。


> [!Note]
> 仅当推导出的价格被应用于改变状态（State-changing）的操作时，使用链上数据才具有风险。如果价格仅用于展示目的（例如前端用于显示数据的一个 view 函数），那么此类攻击是不可行的。然而，如果一个外部合约通过调用这个 view 函数获取价格，并随后将其用于改变状态的操作，那么该操作就会变得易受操纵。

#### 真实案例：Mango Markets

在 Mango Markets 攻击事件中，一名交易者利用该平台的价格操纵漏洞提取了超过 1.16 亿美元。通过在两个钱包中使用 1000 万美元，攻击者以每个 3.8 美分的价格开仓了 4.83 亿份 Mango 永续期货（MNGO-PERPs）。随后，他们在三个不同的交易所购买了价值 400 万美元的 MNGO 代币，将预言机报告的价格推高了 2,300%。利用这一虚增的永续合约仓位作为抵押品，攻击者从 Mango Markets 借走了 1.16 亿美元，留下了巨额坏账并携带资金逃离。正如价格操纵攻击中常见的那样，这并不是传统意义上的“黑客行为”，而是对系统机制的操纵，在不破坏任何底层代码的情况下利用了 Mango 的流动性。

#### 与攻击者谈判

攻击者与协议方经常直接在链上进行谈判，以决定攻击者应归还多少被盗资金，从而换取协议方同意撤销指控。虽然这类交易很常见，但它们在法庭上可能几乎没有法律效力。通常，协议方会向攻击者提供约 10% 的赏金，意味着如果攻击者归还 90% 的资金，协议方将同意停止追究。尽管此类谈判屡见不鲜，但本案的情况尤为引人注目。攻击发生后，攻击者向 Mango Markets 的 DAO 提出了一项协议：如果社区同意承担此前为拯救另一个 Solana 项目 Solend 而产生的部分坏账，他将归还大部分被盗资金。作为回应，Mango 团队提出了第二项方案，要求攻击者归还至多 6700 万美元，同时保留 4700 万美元作为某种“漏洞赏金”。该协议包括放弃任何与坏账相关的索赔，并承诺在代币归还后不进行刑事追究或冻结攻击者的资金。第一项提案被拒绝，而第二项获得了通过。这导致 Mango Markets 在 10 月 15 日发布推文称，确实已归还了 6700 万美元的资产。

当其中一名攻击者在 Twitter 上公开身份，称该行为是“一种高利润的交易策略”并声称这完全是在协议预设的设计范围内进行时，事情发生了法律转折。但美国当局持有不同看法，并以市场操纵罪将其逮捕。随后，Mango Markets 提起民事诉讼，辩称该协议应无效，因为它是“在胁迫下达成的”，并寻求 4700 万美元的损害赔偿。由于 DAO 在法律上是一个相对较新的概念，此案引起了极大关注，并可能为去中心化组织如何处理法律纠纷开辟先例。在一个完美捕捉了加密货币领域疯狂法律现状的转折中，攻击者竟然在 2025 年 5 月赢得了他的欺诈案诉讼：法官裁定，你无法欺诈一个没有服务条款（ToS）的无许可协议。但最戏剧化的是：当局在最初调查 Mango Markets 期间搜查他的设备时，发现了 1200 多张儿童性虐待材料的图像和视频，他现在正因此服刑四年多。这证明了即使是天才的 DeFi 操纵也无法拯救那些违反基本人类道德底线的人。



### 输入验证不当

一个经常被忽视的重大漏洞是输入验证不当。当来自用户或外部来源的输入未经过妥善验证时，智能合约面临的后果可能千差万别，从细微的问题到重大的资金损失均有可能发生。妥善的输入验证有助于防御恶意攻击者（他们可能试图操纵合约行为）以及用户或管理员的无心之失（这些失误原本可能导致资金损失）。如果我们不采取正确的预防措施，一个看似无伤大雅的疏忽也可能导致合约执行中出现重大问题。

#### 漏洞原理

从本质上讲，输入验证不当发生在智能合约在处理数据或参数之前，未对其进行彻底检查的情况下。如果我们不确保某些数值满足特定条件，就会为用户的无心之失和潜在的攻击行为打开大门。用户可能会不小心输入错误的数据，而攻击者则可能故意向合约输入非预期的数据。这可以绕过预设逻辑并导致非预期的状态变更，使得我们的合约表现得不可预测。

该漏洞的一个简单实例出现在 setter（设置）函数中：在将某个地址设置为资金接收者之前，未校验该地址是否为零地址（Zero address）。如果我们错误地将零地址设为接收者，发送到该地址的资金将被永久锁定，无法挽回。

在智能合约开发中，一个普遍且危险的误解是：认为保持源码私有能在某种程度上防止被攻击。我们在本章之前讨论过为什么**“通过隐蔽实现安全”（Security through obscurity）行不通，这一原则同样适用于输入验证。开发者有时会不为某些函数设置保护措施，假设只要不发布代码，这些函数就不会被发现。然而，攻击者能够且确实会对合约字节码（Bytecode）进行逆向工程**，以识别那些敏感且未受保护的函数。例如，由于未受保护的闪电贷回调函数（Flash-loan callbacks），多个闭源的 MEV 机器人已遭到攻击，导致数百万美元的损失。隐藏代码并不能隐藏风险。

#### 预防技术

那么，我们该如何防范输入验证不当呢？第一步非常简单：永远不要假设收到的输入是有效的。无论输入是来自外部账户（EOA）、另一个合约，有时甚至是来自同一个合约内部，都应当进行严格检查。我们不仅需要验证输入的长度，还需要验证边缘情况（Edge cases）和边界条件，例如最小值和最大值。一个不容忽视的经典边缘情况就是零值（Zero value）。

编写安全且易于维护的智能合约的关键在于使用可复用的验证逻辑。我们可以根据实际需求，使用 Modifier（函数修改器） 或 Internal Function（内部函数） 来实现这些验证模块。

* Modifier 特别适用于以一致且声明式的方式为多个函数附加前置或后置条件。例如，我们可以使用 Modifier 来确保函数的输入不是零地址，或者在执行敏感操作前检查调用者是否拥有正确权限。

* Internal Function 同样可以实现这些目标，且有时能提供更高的灵活性，尤其是当验证逻辑较为复杂或需要返回数值时。


#### 访问控制

谈到权限，实施稳健的访问控制至关重要：msg.sender 是一个参数，应当被视为参数来处理。虽然可以编写自定义逻辑，但使用像 OpenZeppelin 这样值得信赖的库，可以帮助我们在最小化复杂性的同时，安全地管理访问权限。

对于需要单一实体拥有完全控制权的简单项目，开发者可以使用 OpenZeppelin 的 Ownable 合约，它指定了一个对关键函数拥有权限的单一“所有者（Owner）”。为了进一步提升安全性，我们建议使用 Ownable2Step。该版本包含一个两步所有权转移流程，有助于防止因误操作导致的所有权丢失。

对于更复杂的需求，OpenZeppelin 的 AccessControl 允许我们创建多个角色，每个角色拥有不同的权限。这种**基于角色的访问控制（RBAC）**让我们可以将特定任务分配给不同的用户，非常适合大型项目。

在实施妥善的访问控制之前，我们需要验证所有关于“谁可能调用外部（external）和公共（public）函数”的假设。智能合约运行在一个公开且去中心化的环境中，因此我们不能假设只有预期的实体才会与合约交互。事实上，我们应当始终假设攻击者会尝试调用这些函数，以触发非预期行为。

#### 典型案例：任意调用

最常见的漏洞利用往往涉及任意调用。在这种情况下，存在漏洞的智能合约允许攻击者提供一个待调用的地址。在这种环境下，合约实际上会执行攻击者想要的任何调用。利用此类漏洞的一种可能方式是：通过返回经过操纵的数值，诱骗合约转出它本不应转出的代币。

请看下面这个存在漏洞的收益聚合器（Yield Aggregator）协议示例代码：

```Solidity
contract Aggregator {
    function stake( ... ) external {
        ...
    }
    function claimMultipleStakingRewards(address[] calldata _claimContracts) external {
        uint256 totalRewards;
        for (uint256 i = 0; i < _claimContracts.length; i++) {
            totalRewards += IClaimContract(_claimContracts[i]).claimStakingRewards(
            msg.sender
            );
        }
        IERC20(stakingToken).transfer(msg.sender, totalRewards);
    }
}
```
其目标很简单，`claimMultipleStakingRewards` 函数循环遍历用户提供的质押合约地址数组，调用每个合约上的 `claimStakingRewards` 函数，并累计奖励总额。最后，它将累计的奖励发送到用户的地址。问题在于，该合约并未检查 `_claimContracts` 数组中的地址是否真正指向受信任的质押合约。这为任意外部调用打开了大门。

例如，攻击者可以部署这样一个恶意合约：
```Solidity
contract Attack {
    function claimStakingRewards(address ) external pure returns (uint256) {
        return 1_000_000 ether; // fabricated reward
    }
}
```
这个恶意合约伪造了一个质押合约，并简单地返回一个虚高的奖励值。当 Aggregator（聚合器）合约对其调用 claimStakingRewards 时，会被诱骗并认为调用者（攻击者）拥有一笔巨额代币。在缺乏额外检查的情况下，Aggregator 会盲目地将该数值计入总额，并将真实的代币转移给攻击者。通过引入一个基础的**允许列表**（Allowlist）来确保 claimMultipleStakingRewards 函数仅允许调用受信任的合约，原本可以避免这一问题。


### 签名重放攻击

以太坊上的签名非常有用，因为它们允许我们在链下授权操作，从而减少了昂贵的链上交易需求。例如，如果你要授权某人代表你执行特定操作（如转移代币或访问智能合约中的某项功能），你可以签署一条链下消息来给予他们许可。随后，合约会验证该签名并执行操作，而无需你直接在链上进行交互。这还实现了无 Gas 交易（Gasless Transactions），即你在链下签署，由中继者（Relayer）提交到链上并支付 Gas 费用。智能合约可以验证这些签名，以确保操作在无需持续链上交互的情况下获得安全授权。

然而，一段数据一旦被签署，逻辑上应该只能使用一次。如果一个已签署的交易可以被重复使用，就会引发重放攻击——攻击者通过重放该签名来多次执行同一操作，例如在未经许可的情况下重复转移资金或修改合约状态。智能合约必须经过精心设计，通过确保每条签署的消息都是唯一的且无法被重放，来防止此类漏洞。

#### 漏洞原理

让我们看一个易受重放攻击的示例合约（示例 9-10）。

示例 9-10. Token：一个易受签名重放攻击的合约

```Solidity
1 contract Token {
2    mapping(address => uint256) public balances;
3    struct Signature {
4        bytes32 r;
5        bytes32 s;
6        uint8 v;
7    }
8    event Transfer(address indexed from, address indexed to, uint256 amount);
9    function transfer(uint256[] memory _amount, address[] memory _from, address[]
     memory _to,  Signature memory _signature) public {
10        bytes32 messageHash = keccak256(abi.encodePacked(_from, _to, _amount));
11        address signer = ecrecover(messageHash, _signature.v, _signature.r, _signature.s);
12        for(uint256 i = 0; i < _from.length; i++){
13            address __from = _from[i];
14            address __to = _to[i];
15            uint256 __amount = _amount[i];
16            require(balances[__from] >= _amount[i], "Insufficient balance");
17            require(signer == _from[i], "Invalid signature");
18            balances[__from] -= __amount;
19            balances[__to] += __amount;
20            emit Transfer(__from, __to, __amount);
21        }
22    }
23 }
```
乍一看，这是一个非常实用的合约函数。它允许任何持有有效签名的人进行多次转账，而无需签名者支付 Gas 费。管理员可以在链下签署数据，然后由其他人（比如某个服务平台）代其在链上提交交易。然而，处理签名绝非易事，这段极短的代码包含了一系列重大问题。

最明显的问题是：缺乏防止重复使用同一签名的机制。由于没有追踪签名是否已被使用的手段，攻击者可以简单地重复该交易，直到受害者的余额被抽干。修复方法很简单：我们需要在签名数据中加入一个 Nonce（仅使用一次的数值，通常是随交易递增的计数器）。负责验证签名的合约有义务检查所提供的 Nonce 此前是否被使用过。这能确保每个签名都是唯一的，从而防止重放。出于同样的原因，以太坊原生交易也使用了 Nonce 机制。

另一个关键问题是签名可延展性（Signature Malleability）。当一个加密签名可以被修改，并在底层消息不变的情况下产生另一个同样有效的签名时，就会发生这种情况。合约中使用的内置 ecrecover 函数就存在这个问题。攻击者可以微调一个有效签名，创造出另一个依然有效的签名，即使底层签署的消息完全没变。为了避免这一点，开发者应使用更安全的签名验证方法，例如 OpenZeppelin 的 ECDSA 库。签名具有可延展性正是你不应将签名本身作为唯一标识符（例如用来防重放）的原因——请坚持使用 Nonce。

我们已经解决了潜在的签名篡改问题，但如果被签署的数据本身也可以被操纵呢？在这个合约中，确实可以。问题出在 `abi.encodePacked` 的使用上。开发者通常为了紧凑编码（节省内存/Gas）而选择它，但这种效率是有代价的。具体来说，`abi.encodePacked` 只是简单地连接原始字节，而不添加长度信息或边界，这意味着不同的输入组合最终可能产生相同的输出。
为了简单起见，假设金额占 8 位（两个十六进制数字），地址占 12 位（三个十六进制数字）。假设参数如下：
```
_amount = [0x64, 0x64]
_from = [0x001, 0x002]
_to = [0x003, 0x003]
```
当我们使用 `abi.encodePacked` 时，它将这些值组合成 `0x6464001002003003`。但棘手的地方在于：如果我们尝试将 `0x002` 从 `_from` 移动到 `_to`，`abi.encodePacked` 生成的输出竟然与之前完全相同。对于这一组新的数值，`abi.encodePacked` 仍会返回：`0x6464001002003003`。
```
_amount = [0x64, 0x64]
_from = [0x001]
_to = [0x002, 0x003, 0x003]
```
当我们使用 `abi.encodePacked` 时，它将这些值组合成 `0x6464001002003003`。这意味着用户 0x002 可以利用那个有效的签名，但通过修改输入参数 `_from` 和 `_to`，诱骗合约认为唯一要执行的转账是从 `0x001` 到 `0x002`。示例中的代码在输入验证方面做得非常糟糕，才导致了这种问题的发生。无论如何，这都表明在对数组等动态数据类型生成签名时，应避免使用 encodePacked。在这些情况下，我们应当使用 abi.encode，它即使在连接动态数据时也能产生无歧义的输出，从而有效防止此类攻击。

然而，还有一个问题。如果这个合约被部署在多条链上会发生什么？同一个签名在所有链上都是有效的，这为跨链重放攻击创造了机会。攻击者可以监控用户在一条链上的活动，然后在其他链上重复使用其签名。为了防止这种情况，我们需要在签署的消息中包含上下文数据——至少要包含 chainId。根据具体用例，你可能还需要包含合约地址或其版本号。

幸运的是，我们不必从头开始设计方案：EIP-712 标准解决了这个问题。它允许生成具备上下文感知能力的签名，并通过向用户展示可读的签署信息（而非令人困惑的字节串）来提升用户体验。

#### 预防技术

为了防止重放攻击和其他漏洞，我们需要确保每个签名都是唯一的、安全的，并且只能使用一次。通过使用 Nonce、安全签名处理以及上下文感知签名，我们可以轻松实现这一目标。

* Nonce（随机数/计数器） 用于确保唯一性。通过在每条签署的消息中加入 Nonce，我们可以防止攻击者重复使用签名。验证签名的合约需确保所使用的 Nonce 是唯一的。一旦某个签名被使用，其对应的 Nonce 即失效，从而阻止重放尝试。

* 安全签名处理：在验证签名时，我们应避免使用原生内置的 ecrecover 函数，转而使用 OpenZeppelin 的 ECDSA 库，该库对“签名可延展性”具有免疫力。此外，我们绝不应将签名本身作为唯一标识符，因为签名是可以被操纵的。

* 规范化编码：对于动态数据的签名，使用 abi.encode 代替 abi.encodePacked。这通过对输入进行妥善分离来防止操纵，确保它们不会被篡改或误读。

* 上下文感知（EIP-712）：最后，每当我们处理签名时，都应实施 EIP-712 标准。除了通过添加 chainId 等上下文信息来防止跨链重放外，EIP-712 还改善了用户体验——它让用户能看到所签署数据的清晰、有意义的可视化展示，而不是晦涩难懂的字节串。这不仅让交易更易于理解，还通过让用户更容易识别可疑请求，增强了用户安全性，从而提高了防御钓鱼攻击的能力。

#### 真实案例：TCH 代币

2024 年 5 月，TCH 代币因一个常见的签名可延展性漏洞而遭到攻击。问题出在合约的 burnToken 函数中，该函数通过验证签名来授权销毁代币。为了防止签名重放攻击，合约将已使用的签名存储在一个映射（Mapping）中。然而，如果签名被篡改，这种防御措施就可以被绕过。攻击者通过收集先前提交的签名并修改其中的 $v$ 和 $s$ 值（签名的一部分）来实施攻击。尽管签名被更改了，但它仍然能通过 ecrecover 的验证。由于修改后的签名与原始签名不同，它未被识别为已使用，随后新版本被存储在映射中。利用这个技巧，攻击者能够反复销毁 PancakeSwap 流动性对所持有的海量 TCH 代币。这使得攻击者能够操纵资金池中的代币价格，并从他们制造的价格波动中获利。

### 智能合约配置错误

配置错误属于那种隐蔽的问题——从技术上讲，它并不算代码层面的“漏洞”，但仍会给智能合约带来严重后果。在你编写完合约并完成审计后，工作并未结束；你仍需对其进行部署、维护，有时还需要进行升级。正是在这些阶段，配置错误经常发生。

以 DeFi 协议为例，它们通常包含海量的参数。如果其中任何一个配置错误，都可能导致重大损失。不幸的是，这类问题很难被发现，甚至在审计中也是如此，因为审计师往往会忽略部署和升级脚本。因此，虽然配置错误本身不是漏洞，但它们可以为漏洞创造切入点。这使得在部署和管理阶段保持格外谨慎变得至关重要。

配置错误问题很难分类，因为它们的形式千差万别。与其尝试列举所有类型，不如让我们直接通过一些真实案例来感受一下到底哪里会出问题。

#### 真实案例：yUSDT

让我们看一个最简单的配置错误案例：Yearn Finance 的 yUSDT 代币中一个配置错误的存储变量，该错误导致了 2023 年 4 月的一次攻击。yUSDT 代币本应通过投资基于 USDT 的衍生品来产生收益，但由于配置失误，它实际上将另一种代币（iUSDC）设为了其底层资产。令人疯狂的是，这一错误竟然在长达一千多天的时间里未被察觉。这一配置错误允许攻击者操纵系统，抽干资金池的价值，并几乎以零成本铸造 yUSDT。结果，yUSDT 的价值归零，攻击者带着 1160 万美元的利润扬长而去。



#### 真实案例：Ronin Bridge

2024 年 8 月，Ronin 跨链桥在一次合约升级仅一小时后即遭到攻击。其根源在于升级过程中的一个失误：一个关键变量 _totalOperatorWeight 未被初始化。这个变量本应在 initializeV3 函数中设置，但在升级期间，团队仅调用了 initializeV4，跳过了前一个版本所需的设置步骤。这一疏忽使合约暴露在风险之中。在这种情况下，一个白帽 MEV 机器人成功抢跑（Front-run）了攻击并归还了被盗的 4,000 枚 ETH，但这再次凸显了对升级流程进行彻底审查和测试的重要性。

> [!Note]
> 如果你认为在合约升级后不久发生的黑客攻击仅仅是巧合，那你就大错特错了。无论黑帽还是白帽黑客，都会密切监控合约的升级动态：黑帽寻找可利用的弱点，而白帽则试图阻止攻击。开发团队经常低估哪怕是微小的代码改动所带来的安全风险，从而跳过审计流程。不幸的是，破坏一个合约并不需要大费周章。正如本案例所展示的，漏洞并不总是在智能合约代码本身——有时，它隐藏在升级的执行方式中。


#### 真实案例：Sonne Finance

2024 年 5 月 Sonne Finance 遭黑客攻击的根本原因并非典型的协议漏洞，而是其市场激活流程中的缺陷。与许多协议一样，Sonne 深知 Compound v2 中存在的“空市场（Empty Market）”漏洞——即一个已开放但尚未注资的市场可能被利用，进而抽干整个协议的资金。针对此漏洞的标准修复方案是：在市场激活时，确保资金**原子性（Atomically）**地存入市场，从而防止市场在任何时刻处于真空状态。

Sonne 原本制定了处理计划：先添加市场，存入资金，然后开放市场供使用——这三个动作都将通过时间锁（Timelocks）完成。如果按正确顺序执行，这个流程本应行之有效。问题在于，Sonne 在治理时间锁控制器中将这些步骤调度为独立的交易，这意味着它们的执行顺序并没有被强制约束。

Sonne 团队将治理的 EXECUTOR_ROLE（执行者角色）设为对所有人开放，允许任何用户在时间锁到期后执行治理交易。虽然这种设置并不常见，但其本身并非致命问题；然而，在此特定情况下，它却是毁灭性的——它允许任何人在时间锁到期后不按顺序执行这些操作。

攻击者只需在资金注入之前，直接执行所有已排队的市场开放操作，从而留下了一个未注资且易受攻击的空市场。通过利用这个无资金市场，攻击者从协议中抽走了 2,000 万美元。

这里的核心教训是：当为了确保安全而需要按特定顺序执行治理操作时，应当使这些操作具备原子性。 例如，如果使用 OpenZeppelin 的 Timelock，应当使用 scheduleBatch() 而不是 schedule()。Sonne 的错误在于允许这些操作分开排队，从而导致了风险暴露。

#### 预防技术

为了避免配置错误问题以及我们讨论过的那类代价高昂的失误，我们需要在智能合约的整个生命周期中采取主动防御措施，尤其是在涉及部署、升级和任何关键治理操作时。

我们应始终确保部署和升级脚本经过了彻底的测试和审计。我们的目光必须超越代码审计本身，密切关注那些直接操作主网的脚本，确保整个流程的每一个环节都在类生产环境（Live-like environment）中经过了验证。


## 合约库

目前有大量的现有代码可供复用，既有作为可调用库部署在链上的，也有作为代码模板库存在于链下的。在以太坊中，使用最广泛的资源是 [OpenZeppelin](https://oreil.ly/OSSoV) 系列，这是一套内容丰富的合约库，涵盖了从各种代币实现到不同的代理架构（Proxy），再到合约中常见的简单行为模式，如 `Ownable`（所有权控制）、`Pausable`（可暂停性）或 `ReentrancyGuard`（重入保护）。该代码库中的合约经过了广泛的测试，在某些情况下甚至被视为事实上的标准实现。它们可以免费使用，并由 [OpenZeppelin](https://www.openzeppelin.com/) 以及不断增加的外部贡献者共同构建和维护。

其他值得关注的合约库还包括 Paradigm 的 [Solmate](https://oreil.ly/yTuU9) 和 Vectorized 的 [Solady](https://oreil.ly/zqpib)。Solmate 在设计上更具主见（Opinionated），而 Solady 则主要专注于 Gas 优化。


## 补充资源

由于智能合约安全涉及如此多的深度和细微差别，我们为感兴趣的读者列出了以下资源，以便深入学习这一进阶课题：

**Cyfrin Updraft 智能合约安全与审计**

一门全面的 24 小时课程（包含 270 多节课）。

**Secureum Bootcamp**

为期三个月的密集训练营，专注于以太坊智能合约安全审计。

**Ethernaut**

由 OpenZeppelin 开发的基于 Solidity 的闯关游戏，玩家通过破解智能合约关卡来学习常见的漏洞。

**Damn Vulnerable DeFi**

一个夺旗赛（CTF）平台，包含 18 个挑战，涵盖闪电贷、价格预言机、治理、NFT 等内容。

**Capture the Ether**

经典的 CTF 风格以太坊安全游戏。

**QuillCTF**

由 QuillAudits 收集的以太坊安全谜题集。

**Paradigm CTF**

由 Paradigm 组织的年度在线 CTF 竞赛，面向经验丰富的智能合约黑客；挑战题目非常先进且反映了最前沿的攻击手段。赛后通常会发布官方解决方案和解析（Write-ups），使其成为极具价值的学习资源。


## 结语

得益于版本的不断迭代，现在的 Solidity 编译器已经能够自动缓解如整数溢出（Integer Overflows）和默认可见性（Default Visibility）等风险。这使我们能够移除本书第一版中提到的一些陈旧陷阱，转而利用篇幅关注更具当下性且更相关的漏洞。

即便如此，任何深耕于智能合约领域的开发者仍有大量知识需要学习和理解。通过在智能合约设计和代码编写中遵循最佳实践，你将能够规避许多严重的隐患和陷阱。

或许，最基础的软件安全原则就是最大限度地复用受信任的代码。在密码学领域，这一原则至关重要，甚至演变成了一句名言：“永远不要搞私有的加密算法（Don’t roll your own crypto）”。在智能合约领域，这意味着应尽可能利用那些经过社区彻底审查的免费开源库，并从中获益。
