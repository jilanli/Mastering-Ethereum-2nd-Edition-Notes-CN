# 第 7 章：智能合约与 Solidity

正如我们在第 2 章中讨论过的，以太坊中有两种不同类型的账户：外部账户（EOA）和合约账户。EOA 由用户控制，通常是通过以太坊平台之外的软件（如钱包应用）来操作。相比之下，合约账户由程序代码（通常也被称为智能合约）控制，这些代码由 EVM（以太坊虚拟机）执行。

简而言之，EOA 是不关联任何代码或数据存储的简单账户，而合约账户则既有关联代码又有数据存储。EOA 由交易控制，这些交易是在独立于协议之外的“现实世界”中创建，并使用私钥进行加密签名的；而合约账户没有私钥，因此它们以其智能合约代码所规定的预置方式进行“自我控制”。这两类账户都由一个以太坊地址来标识。在本章中，我们将讨论合约账户以及控制它们的程序代码。

## 什么是智能合约？
“智能合约”（Smart Contract）这一术语多年来被用来描述各种不同的事物。20 世纪 90 年代，密码学家**尼克·萨博（Nick Szabo）**创造了这个术语，并将其定义为“一套以数字形式指定的承诺，包括各方履行这些承诺的协议”。自那时起，智能合约的概念不断演变，特别是在 2009 年比特币发布以及随后的去中心化区块链平台出现之后。

在以太坊的语境下，这个术语其实有点名不副实。因为以太坊智能合约既不“智能”，也不是法律意义上的“合约”，但这个术语还是流传了下来。在本书中，我们使用“智能合约”指代那些作为以太坊网络协议的一部分、在 EVM（以太坊虚拟机）上下文中确定性运行的不可篡改的计算机程序——也就是说，它们运行在去中心化的“以太坊世界计算机”上。

让我们拆解一下这个定义：

**计算机程序**（Computer programs） 

智能合约仅仅是计算机程序。在此语境下，“合约”一词没有任何法律含义。

**不可篡改**（Immutable） 

一旦部署，智能合约的代码就无法更改。与传统软件不同，修改智能合约的唯一方法是部署一个新实例。

**确定性**（Deterministic） 

在发起执行的交易背景以及执行时的以太坊区块链状态确定的情况下，智能合约的执行结果对每个运行它的人来说都是相同的。

**EVM 上下文**（EVM context） 

智能合约在一个非常受限的执行上下文中运行。它们只能访问自身的状态、调用它们的交易上下文，以及有关最近区块的一些信息。

**去中心化世界计算机**（Decentralized world computer） 

EVM 作为本地实例运行在每个以太坊节点上。但由于所有 EVM 实例都从相同的初始状态出发，并产生相同的最终状态，因此整个系统作为一个单一的“世界计算机”运行。

## 智能合约的生命周期
智能合约通常使用诸如 Solidity 之类的高级语言编写。但为了运行，它们必须被编译成在 EVM 中运行的低级字节码。一旦编译完成，它们就会使用一种特殊的合约创建交易部署到以太坊平台上，这类交易的特征是具有空的 to 字段（参见第 6 章中的“特殊交易：合约创建”）。每个合约都由一个以太坊地址标识，该地址是合约创建交易根据发起账户和 nonce 计算得出的函数。合约的以太坊地址可以在交易中作为接收者，用于向合约发送资金或调用合约的其中一个函数。请注意，与 EOA 不同，为新智能合约创建的账户没有关联的密钥。作为合约创建者，你在协议层面并不会获得任何特殊特权（尽管你可以显式地在智能合约中编写特权代码）。你当然也不会收到合约账户的私钥，因为私钥实际上并不存在——我们可以说智能合约账户拥有它们自己。

重要的是，合约只有在被交易调用时才会运行。以太坊中的所有智能合约最终都是由于 EOA 发起的交易而执行的。一个合约可以调用另一个合约，后者又可以调用另一个合约，依此类推，但此类执行链中的第一个合约始终是由来自 EOA 的交易调用的。合约从不“自行运行”或“在后台运行”。在交易直接或间接地触发执行（作为合约调用链的一部分）之前，合约实际上处于休眠状态。还值得注意的是，智能合约在任何意义上都不是“并行”执行的——以太坊世界计算机可以被认为是一台单线程机器。

无论交易调用了多少个合约，或者这些合约在被调用时执行了什么操作，交易都是原子性的。交易会完整地执行，只有当所有执行都成功终止时，全局状态（合约、账户等）的更改才会被记录。成功终止意味着程序在没有错误的情况下运行并到达执行终点。如果执行由于错误而失败，其所有影响（状态更改）都会被“回滚”，就像交易从未运行过一样。失败的交易仍会被记录为已尝试，并且用于执行的 gas 会从发起账户中扣除，但除此之外对合约或账户状态没有影响。

如前所述，合约的代码一旦部署就无法更改。从历史上看，合约可以被删除，从而从其地址中移除代码和内部状态（存储），留下一个空白账户。在这样的删除之后，发送到该地址的任何交易都不会导致代码执行，因为没有代码留存。这种删除是通过名为 `SELFDESTRUCT` 的 EVM 操作码完成的，它提供了 gas 返还，通过删除存储状态来激励释放网络资源。然而，`SELFDESTRUCT` 操作在 2023 年通过 [EIP-6780](https://oreil.ly/5LcZo) 被弃用，原因是它对账户状态要求的重大变动，特别是移除所有代码和存储。随着以太坊路线图中的后续升级，这种操作将不再可行。

## 以太坊高级语言简介
EVM 是一个运行名为 EVM 字节码的特殊代码形式的虚拟机，类似于你计算机的 CPU 运行诸如 x86_64 之类的机器码。我们将在第 14 章更详细地研究 EVM 的运行和语言。在本节中，我们将了解如何编写在 EVM 上运行的智能合约。

虽然可以直接用字节码编写智能合约，但 EVM 字节码相当笨拙，程序员很难阅读和理解。相反，大多数以太坊开发人员使用高级语言编写程序，并使用编译器将其转换为字节码。

尽管任何高级语言都可以适配用来编写智能合约，但将任意语言适配为可编译为 EVM 字节码是一项相当繁琐的任务，并且通常会导致一定程度的混乱。智能合约在一个高度受限且极简的执行环境（EVM）中运行。此外，还需要提供一组特定的 EVM 系统变量和函数。因此，从头开始构建一种智能合约语言比使通用语言适用于编写智能合约更容易。结果，出现了许多用于编写智能合约的专用语言。以太坊拥有几种这样的语言，以及生成 EVM 可执行字节码所需的编译器。

通常，编程语言可以分为两大编程范式：声明式和命令式，分别也称为函数式和过程式。在声明式编程中，我们编写表达程序逻辑而非其流程的函数。声明式编程用于创建没有副作用的程序，这意味着在函数之外没有状态变化。声明式编程语言包括 Haskell 和 SQL。相比之下，命令式编程是程序员编写一组结合了程序逻辑和流程的过程。命令式编程语言包括 C++ 和 Java。有些语言是“混合型”的，意味着它们鼓励声明式编程，但也可以用来表达命令式编程范式。这类混合型语言包括 Lisp、JavaScript 和 Python。通常，任何命令式语言都可以用来以声明式范式编写，但往往会导致代码不够优雅。相比之下，纯声明式语言不能用来编写命令式范式。在纯声明式语言中，没有“变量”。

虽然命令式编程更常被程序员使用，但编写完全按预期执行的程序可能非常困难。程序的任何部分改变任何其他部分状态的能力使得推导程序的执行变得困难，并引入了许多产生漏洞的机会。相比之下，声明式编程更容易理解程序的行为：由于它没有副作用，程序的任何部分都可以被隔离理解。

在智能合约中，漏洞字面意义上意味着金钱损失。因此，编写没有非预期效果的智能合约至关重要。为此，你必须能够清晰地推导程序的预期行为。因此，声明式语言在智能合约中发挥的作用比在通用软件中大得多。然而，正如你将看到的，最广泛使用的智能合约语言（Solidity）是命令式的。程序员和大多数人类一样，抵制改变！

目前支持的智能合约高级编程语言包括以下几种（按流行程度排序）：

**Solidity**

一种过程式（命令式）编程语言，语法类似于 JavaScript、C++ 或 Java。它是以太坊智能合约中最流行且最常用的语言。

**Yul**

一种中间语言，可以以独立模式使用或在 Solidity 中内联使用，非常适合跨平台的高级优化。初学者在探索 Yul 之前应先从 Solidity 或 Vyper 开始，因为它需要具备关于智能合约安全和 EVM 的高级知识。

**Vyper**

一种面向合约的编程语言，具有类 Python 语法，它优先考虑用户安全，并通过语言设计和高效执行来鼓励清晰的编码实践。

**Huff**

一种底层编程语言，主要由需要高度高效且极简合约代码的开发人员使用，允许进行超出 Solidity 等高级语言所能提供的高级优化。与 Yul 一样，不建议初学者使用。

**Fe**

一种用于 EVM 的静态类型智能合约语言，受 Python 和 Rust 启发。它的目标是易于学习，即使对于刚接触以太坊的开发人员也是如此。自 2021 年 1 月发布 Alpha 版本以来，其开发仍处于早期阶段。

过去还开发过其他语言，但现在已不再维护，例如 LLL、Serpent 和 Bamboo。

如你所见，有许多语言可供选择。然而，在所有这些语言中，Solidity 是迄今为止最流行的，甚至已经成为以太坊乃至其他类 EVM 区块链上事实上的高级语言。

## 使用 Solidity 构建智能合约
Solidity 由 Gavin Wood（本书第一版的合著者）创造，是一种专门用于编写智能合约的语言，其特性直接支持在以太坊世界计算机的去中心化环境中运行。由于其属性具有相当的通用性，它最终被用于在其他多个区块链平台上编写智能合约。该语言最初由 Christian Reitwiessner 开发，随后由 Alex Beregszaszi、Liana Husikyan、Yoichi Hirai 以及几位前以太坊核心贡献者接手。Solidity 目前作为一个独立项目在 [GitHub](https://oreil.ly/ik9kH)上进行开发和维护。

Solidity 项目的核心“产品”是 Solidity 编译器 solc，它将使用 Solidity 语言编写的程序转换为 EVM 字节码。该项目还管理着以太坊智能合约中重要的 ABI（应用程序二进制接口）标准，我们将在本章详细探讨这一标准。Solidity 编译器的每个版本都对应并编译特定版本的 Solidity 语言。

为了开始学习，我们将下载 Solidity 编译器的二进制可执行文件。然后，我们将延续第 2 章中的示例，开发并编译一个简单的合约。

### 选择 Solidity 版本
Solidity 遵循一种名为**语义化版本控制**（[Semantic Versioning](https://semver.org/)）的版本模型，其版本号结构由点分隔的三个数字组成：主版本号.次版本号.修订号（MAJOR.MINOR.PATCH）。“主版本号”在发生重大且不向后兼容的更改时增加；“次版本号”在主版本发布之间添加向后兼容的功能时增加；“修订号”则用于向后兼容的漏洞修复。

在撰写本文时，Solidity 的版本为 0.8.26。主版本号为 0 的规则（用于项目的初始开发阶段）有所不同：任何内容都可能随时发生变化。在实践中，Solidity 将“次版本号”视为主版本号，将“修订号”视为次版本号。因此，在 0.8.26 中，8 被视为主要版本，而 26 被视为次要版本。正如你在第 2 章中看到的，你的 Solidity 程序可以包含一条 pragma 指令，该指令指定了与之兼容并可用于编译合约的 Solidity 最低和最高版本。由于 Solidity 正在迅速演变，通常最好安装最新版本。

> [!Note]
> Solidity 快速演变的另一个后果是文档过时的速度很快。目前，我们使用的是 Solidity 0.8.26 版本，本书中的所有内容都基于该版本。虽然本书将始终为你学习 Solidity 打下坚实的基础，但未来的版本可能会更改某些语法和功能。因此，每当你产生疑问或遇到新事物时，查看 Solidity [官方文档](https://oreil.ly/LzV7L)以保持更新是一个好主意。

### 下载与安装 Solidity
根据你的操作系统和具体需求，你可以通过多种方法下载并安装 Solidity：既可以使用二进制发布版本，也可以通过源代码进行编译。你可以在 Solidity [官方文档](https://oreil.ly/JW--z)中找到详细且更新及时的说明。

以下是在 Ubuntu/Debian 操作系统上，使用 apt 软件包管理器安装最新 Solidity 二进制发布版本的方法：
```Bash
$ sudo add-apt-repository ppa:ethereum/ethereum
$ sudo apt update
$ sudo apt install solc
```
安装好 solc 之后，可以通过运行以下命令检查版本：
```Bash
$ solc --version
solc, the solidity compiler commandline interface
Version: 0.8.26+commit.8a97fa7a.Linux.g++
```
### 开发环境
虽然完全可以使用简单的文本编辑器来开发 Solidity 智能合约，但利用像 [Hardhat](https://hardhat.org/) 或 [Foundry](https://oreil.ly/-7Qvy) 这样的开发框架可以显著提高开发效率。这些框架提供了一套全面的工具，能够简化并改进开发流程。例如，它们提供了强大的测试环境，允许你编写并运行单元测试来验证合约行为；它们还提供**分叉（Forking）**功能，可以创建主网的本地实例，用于真实的测试场景。凭借先进的调试和追踪功能，你可以轻松地单步执行代码，快速识别并解决问题，从而节省时间并减少错误。此外，这些框架还支持脚本编写和部署自动化、扩展功能的插件生态系统，以及跨不同环境的无缝网络管理。将这些功能整合到你的工作流中，可以确保更高水平的代码质量和安全性，而这仅凭简单的文本编辑器是很难实现的。

除了框架之外，采用像 VS Code 这样的现代 IDE 能进一步提高生产力。VS Code 为 Solidity 提供了广泛的扩展插件，包括让代码更易读的语法高亮；帮助组织和导航复杂项目的先进注释和书签工具；以及提供代码结构洞察和潜在问题分析的视觉工具。此外，还有像 Remix IDE 这样基于 Web 的开发环境。

这些工具结合在一起，不仅提高了代码质量，还加速了开发过程，使我们能够更快速、更安全地构建和部署智能合约。

### 编写一个简单的 Solidity 程序
在第 2 章中，我们编写了第一个 Solidity 程序。当初次构建 Faucet（水龙头）合约时，我们使用 Remix IDE 来编译和部署该合约。在本节中，我们将重新审视、改进并修饰这个 Faucet 合约。

我们的第一次尝试如示例 7-1 所示。

示例 7-1. Faucet.sol：一个实现水龙头功能的 Solidity 合约
```Solidity
// SPDX-License-Identifier: GPL-3.0
// Our first contract is a faucet!
contract Faucet {
    // Give out ether to anyone who asks
    function withdraw(uint _withdrawAmount, address payable _to) public {
        // Limit withdrawal amount
        require(_withdrawAmount <= 100000000000000000);
        // Send the amount to the address that requested it
        _to.transfer(_withdrawAmount);
    }
    // Accept any incoming amount
    receive() external payable {}
}
```
正如我们在第 2 章中看到的，注释中的 SPDX 许可证标识符表明该智能合约是在 GPL-3.0 协议下授权的。这旨在告知用户和开发者在使用及分发该代码时所享有的法律权利和承担的义务。

### 使用 Solidity 编译器 (solc) 进行编译
现在，我们将使用命令行形式的 Solidity 编译器来直接编译我们的合约。Solidity 编译器 solc 提供了多种选项，你可以通过传递 `--help` 参数来查看这些选项。

我们使用 solc 的 `--bin` 和 `--optimize` 参数来为我们的示例合约生成优化后的二进制文件：
```Bash
$ solc --optimize --bin Faucet.sol
======= Faucet.sol:Faucet =======
Binary:
6080604052348015600e575f5ffd5b5060fa8061001b5f395ff3fe608060405260043610601d575f3560e01c806
2f714ce146027575f5ffd5b36602357005b5f5ffd5b3480156031575f5ffd5b506041603d366004608d565b6043
565b005b67016345785d8a00008211156056575f5ffd5b6040516001600160a01b0382169083156108fc0290849
05f818181858888f193505050501580156088573d5f5f3e3d5ffd5b505050565b5f5f60408385031215609d575f
5ffd5b8235915060208301356001600160a01b038116811460b9575f5ffd5b80915050925092905056fea264697
06673582212208935b6cf5d9070b7609ad59ac4b727e512522c674cacf09a2eff88dafa3242ee64736f6c634300
081b0033
```
solc 产生的结果是一个十六进制序列化的二进制文件，它可以被提交到以太坊区块链上。

## 以太坊合约 ABI
在计算机软件中，应用程序二进制接口（ABI）是两个程序模块之间的接口——通常存在于操作系统与用户程序之间。ABI 定义了在机器码级别如何访问数据结构和函数；这不应与 API 混淆，API 是在高级的、通常是人类可读的源代码格式中定义这种访问。因此，ABI 是将数据编码进机器码以及从机器码中解码数据的主要方式。

在以太坊中，ABI 用于为 EVM 编码合约调用，并从交易中读取数据。ABI 的目的是定义合约中可以被调用的函数，并描述每个函数如何接收参数以及如何返回结果。

合约的 ABI 被指定为一个由函数描述（参见“函数”）和事件（参见“事件”）组成的 JSON 数组。函数描述是一个包含 type（类型）、name（名称）、inputs（输入）、outputs（输出）、constant（是否为常量）和 payable（是否可支付）等字段的 JSON 对象。事件描述对象则包含 type、name、inputs 和 anonymous（是否匿名）字段。

我们使用 solc 命令行编译器来为我们的 Faucet.sol 示例合约生成 ABI：
```Bash
$ solc --abi Faucet.sol
======= Faucet.sol:Faucet =======
Contract JSON ABI
[{"inputs":[{"internalType":"uint256","name":"withdrawAmount","type":"uint256"}],
"name":"withdraw","outputs":[],"stateMutability":"nonpayable","type":"function"},
{"stateMutability":"payable","type":"receive"}]
```

如你所见，编译器生成了一个 JSON 数组，描述了 Faucet.sol 中定义的两个函数。一旦合约部署完成，任何想要访问 Faucet 合约的应用程序都可以使用这个 JSON。利用 ABI，诸如钱包或 DApp 浏览器之类的应用程序可以构造交易，以正确的参数和参数类型调用 Faucet 中的函数。例如，钱包会知道要调用 withdraw 函数，必须提供一个名为 withdrawAmount 的 uint256 类型参数。钱包可以提示用户提供该数值，然后创建一个对该数值进行编码并执行 withdraw 函数的交易。

应用程序与合约进行交互所需的全部信息就是 ABI 以及合约部署的地址。

## 选择 Solidity 编译器和语言版本
正如我们在之前的代码中看到的，我们的 Faucet 合约在 Solidity 0.8.26 版本下可以成功编译。但如果我们使用的是另一个版本的 Solidity 编译器会怎样呢？这种语言仍处于不断的变动之中，事物可能会以意想不到的方式发生变化。我们的合约相当简单，但如果我们的程序使用了一个仅在 Solidity 0.8.26 版本中新增的功能，而我们尝试用 0.8.25 版本去编译它，结果会如何？

为了解决此类问题，Solidity 提供了一种名为 版本 Pragma (version pragma) 的编译器指令，用于告知编译器该程序需要特定的编译器（及语言）版本。让我们看一个例子：
```Solidity
pragma solidity 0.8.26;
```
Solidity 编译器会读取版本 Pragma，如果编译器版本与该指令不兼容，则会产生错误。在这种情况下，我们的版本 Pragma 表明该程序可以由版本为 0.8.26 的 Solidity 编译器编译。Pragma 指令不会被编译进 EVM 字节码；它们是编译时指令，仅供编译器用于检查兼容性。

> [!Note]
> 在 pragma 指令中，符号 ^ 表示我们允许使用等于或高于指定版本的任何**次版本**（minor revision）进行编译。例如，指令 pragma solidity ^0.7.1; 意味着该合约可以使用版本号为 0.7.1、0.7.2 和 0.7.3 的 solc 进行编译，但不能使用 0.8.0（因为 0.8.0 是主版本，而不是次版本）。

让我们为 Faucet 合约添加一个 pragma 指令。我们将新文件命名为 Faucet2.sol，以便在接下来的示例中记录我们的改动，如示例 7-2 所示。

示例 7-2. Faucet2.sol：为 Faucet 添加版本 pragma
```Solidity
pragma solidity 0.8.26;
// SPDX-License-Identifier: GPL-3.0
// Our first contract is a faucet!
contract Faucet {
    // Give out ether to anyone who asks
    function withdraw(uint _withdrawAmount, address payable _to) public {
        // Limit withdrawal amount
        require(_withdrawAmount <= 100000000000000000);
        // Send the amount to the address that requested it
        _to.transfer(_withdrawAmount);
    }
    // Accept any incoming amount
    receive() external payable {}
}
```
添加版本 pragma 是一项最佳实践，因为它能避免编译器与语言版本不匹配的问题。在本章中，我们将探索其他的最佳实践，并继续改进 Faucet 合约。

## 使用 Solidity 编程
在本节中，我们将了解 Solidity 语言的一些功能。正如我们在第 2 章中提到的，我们的第一个合约示例非常简单，而且在多方面都存在缺陷。在探索如何使用 Solidity 的过程中，我们将在这里逐步改进它。不过，这不会是一本详尽无遗的 Solidity 教程，因为 Solidity 相当复杂且演变迅速。我们将介绍基础知识，为你打下坚实的基础，以便你能够自行探索其余内容。

### 数据类型
首先，让我们来看看 Solidity 提供的一些基础数据类型：

**布尔型 (bool)**

布尔值，true 或 false，配合逻辑运算符使用：!（非）、&&（与）、||（或）、==（等于）以及 !=（不等于）。

**整型 (int, uint)**

有符号整型（int）和无符号整型（uint），以 8 位为增量进行声明，范围从 int8 到 uint256。如果不带尺寸后缀，则默认使用 256 位，以匹配 EVM 的字长（Word Size）。

**定点数 (fixed, ufixed)**

定点数，声明格式为 (u)fixedMxN，其中 M 表示总位数（以 8 为增量，最高 256 位），N 表示小数点后的位数（最多 18 位）——例如 ufixed32x2。

> [!Note]
> 注意 Solidity 目前尚未完全支持定点数。它们可以被声明，但无法进行赋值（无论是赋值给他人还是被他人赋值）。

**地址（Address）**

一个 20 字节的以太坊地址。address 对象拥有许多有用的成员函数，最主要的是 balance（返回账户余额）和 transfer（向该账户转账以太币）。

**字节数组（定长）**

固定大小的字节数组，声明范围从 bytes1 到 bytes32。

**字节数组（变长）**

大小可变的字节数组，声明为 bytes 或 string。

**枚举（Enum）**

用于列举离散值的用户自定义类型：enum NAME {LABEL1, LABEL2, ...}。枚举的底层类型是 uint8；因此，它最多只能有 256 个成员，并且可以显式转换为所有整型。

**数组（Arrays）**

任意类型的数组，可以是定长的或动态的：uint32[][5] 是一个包含 5 个动态无符号整型数组的定长数组。

**结构体（Struct）**

用于将变量分组的用户自定义数据容器：struct NAME {TYPE1 VARIABLE1; TYPE2 VARIABLE2; ...}。

**映射（Mapping）**

用于 键 ⇒ 值 对的哈希查找表：`mapping(KEY_TYPE => VALUE_TYPE) NAME`。

除了这些数据类型，Solidity 还提供了多种值字面量，可用于计算不同的单位。

**时间单位（Time units）**

全局变量 block.timestamp 表示区块发布并添加到区块链时的时间（以秒为单位），从 Unix 纪元（1970 年 1 月 1 日）起算。seconds、minutes、hours 和 days 这些单位可以作为后缀使用，并转换为基础单位“秒”的倍数。

> [!Note]
> 由于一个区块可以包含多笔交易，该区块内的所有交易都共享同一个 block.timestamp。这个时间戳反映的是区块发布（被挖出/确认）的时间，而不是每笔交易发起的准确时刻。

**以太币单位（Ether units）**

单位 wei 和 ether 可以作为后缀使用，它们会被转换为基础单位 wei 的倍数。此前，finney 和 szabo 等面值单位也是可用的，但在 Solidity 0.7.0 版本中它们已被移除。

在我们的 Faucet 合约示例中，我们为 withdrawAmount 变量使用了 uint 类型（它是 uint256 的别名）。我们也间接地使用了 address 变量，并通过 msg.sender 对其进行赋值。在本章剩余的示例中，我们将使用更多这些数据类型。

让我们使用单位乘法器来提高示例合约的可读性。在 withdraw 函数中，我们限制了最大提取金额，并以以太币的基础单位 wei 来表示该限制：
```Solidity
require(withdrawAmount <= 100000000000000000);
```
这非常不利于阅读。我们可以通过使用单位乘法器 ether 来改进代码，从而用 ether 而不是 wei 来表达该数值：
```Solidity
require(withdrawAmount <= 0.1 ether);
```

### 变量：定义与作用域
在 Solidity 中，定义变量和函数的语法与其他静态类型语言类似：我们为每个变量分配一个类型、一个名称以及一个可选的初始值。对于状态变量，我们还可以指定它们的可见性（visibility）。默认的可见性是 internal，这意味着该变量只能在当前合约及其派生合约中访问。若要让其他智能合约也能访问它们，我们需要使用 public 可见性。

Solidity 智能合约具有三种类型的变量作用域：

**状态变量** (State variables) 

通过在区块链上记录数值，这些变量在智能合约中存储永久数据，即所谓的持久状态。状态变量定义在智能合约内部，但在任何函数之外。例如：`uint public count;`

**局部变量** (Local variables) 

这些是在计算过程中使用的临时数据，仅在短时间内保存信息。局部变量不会存储在区块链上；它们存在于函数内部，在定义的范围之外无法访问。例如：`uint count = 1;`

**全局变量** (Global variables) 

这些变量由 Solidity 自动提供，无需显式声明或导入即可使用。它们提供了有关区块链环境的信息，并包含程序中使用的实用函数。预定义的全局变量在下一节中会详尽列出。

正如我们简要提到的，声明状态变量时可以指定其可见性。Solidity 提供了三种不同的可见性级别：*public*（公开）会自动生成读取器函数（getter functions），允许外部合约读取其数值，但外部合约无法直接修改它们。*internal*（内部）仅在当前合约及其派生合约（继承自该合约的合约）中可访问。*private*（私有）与 internal 类似，但即使是派生合约也无法访问。

### 预定义全局变量与函数
当合约在 EVM 中执行时，它可以访问一小组全局对象。这些对象包括 block（区块）、msg（消息）和 tx（交易）。此外，Solidity 还将许多 EVM 操作码（opcodes）公开为预定义函数。在本节中，我们将检查你可以从 Solidity 智能合约内部访问的变量和函数。

**交易/消息调用上下文（Transaction/message call context）**

msg 对象是指发起本次合约执行的交易调用（由 EOA 发起）或消息调用（由合约发起）。它包含了一系列有用的属性：

**msg.sender**

我们已经使用过这个属性了。它代表发起本次合约调用的地址，但不一定是指发送交易的原始 EOA（外部持有账户）。如果我们的合约是由一个 EOA 交易直接调用的，那么 msg.sender 就是签署该交易的地址；但在其他情况下（如通过合约调用合约），它将是一个合约地址。

**msg.value**

本次调用所发送的 Ether 数量（以 wei 为单位）。

**msg.data**

进入我们合约本次调用的数据负载（Data Payload）。

**msg.sig** 

数据负载的前四个字节，即函数选择器（Function Selector）。

> [!Note]
> 每当一个合约调用另一个合约时，msg 对象所有属性的值都会发生变化，以反映新调用者的信息。唯一的例外是 delegatecall 函数，它会在原始的 msg 上下文（Context）中运行另一个合约或库（Library）的代码。

**交易上下文 (Transaction context)**

tx 对象提供了一种访问交易相关信息的方法：

**tx.gasprice**

发起调用的交易中的 Gas 价格。

**tx.origin**

该交易的原始 EOA（外部持有账户）地址。

**区块上下文 (Block context)**

block 对象包含当前区块的以下信息：

**block.basefee**

当前区块的基础费用（Base Fee）。这是一个动态调整的值，代表交易被纳入区块所需的最低 Gas 费用。

**block.blobbasefee**

Blob 交易的动态调整基础费用。这是作为以太坊 EIP-4844 扩容改进方案的一部分引入的，旨在高效处理大数据。

**block.chainid**

当前正在构建的区块链的唯一标识符（链 ID）。

**block.prevrandao**

由信标链（Beacon Chain）提供的随机数。该值源自前一个区块的随机数信标（Randomness Beacon）。它可用于需要随机数的智能合约，但仅限非敏感操作，因为它在一定程度上仍可被操纵。

**block.coinbase**

当前区块费用和区块奖励的接收者地址（通常指构建者或验证者地址）。

**block.difficulty**

在 Paris 升级（合并/The Merge）之前的 EVM 版本中，指当前区块的 PoW 难度。对于之后采用 PoS 共识模型的版本，它表现为 block.prevrandao 的弃用别名。

**block.gaslimit**

当前区块内所有交易所允许消耗的 Gas 总量上限（区块 Gas 限制）。

**block.number**

当前区块高度（区块号）。

**block.timestamp**

区块中的时间戳（自 Unix 纪元以来的秒数）。

**地址对象 (Address object)**

任何地址（无论是作为输入传入还是从合约对象强制转换而来）都拥有一系列属性和方法：

**address.balance**

该地址的余额（以 wei 为单位）。例如，当前合约的余额为 address(this).balance。

**address.code**

存储在该地址的合约字节码（Bytecode）。对于 EOA 地址，返回空字节数组。

**address.codehash**

该地址合约字节码的 Keccak-256 哈希值。

**address.transfer(amount)**

向该地址转账指定金额（以 wei 为单位），如果出错则抛出异常。我们在 Faucet 示例中曾将此函数作为 msg.sender 地址的方法使用，即 msg.sender.transfer。

**address.send(amount)**

类似于 transfer，但出错时不会抛出异常，而是返回 false。务必记得检查 send 的返回值。

**address.call(payload)**

底层 CALL 函数。可以使用数据负载（Payload）构建任意的消息调用。出错时返回 false。注意：接收者可能（无意或恶意地）耗尽你所有的 Gas，导致你的合约因 **OOG（Gas 耗尽）**异常而停滞；务必检查 call 的返回值。

**address.delegatecall(payload)**

底层 DELEGATECALL 函数。类似于 call，但上下文保留在当前合约中（逻辑由目标地址提供）。对于实现代理模式（Proxy Pattern）特别有用。出错时返回 false。警告：仅限高级用途！

**address.staticcall(payload)**

底层 STATICCALL 函数。类似于 call 但处于只读模式，意味着被调用的函数不能修改任何状态或发送 Ether。出错时返回 false。

> [!Note]
> address.send() 和 address.transfer() 都会固定转发 2,300 Gas，这可能不足以执行回退逻辑（Fallback Logic）。随着 EIP-7702 的正式实施，社区已不再推荐使用这两个函数，转而提倡使用更具灵活性的 address.call()。更多相关内容将在第 9 章详细讨论。

### 内置函数
其他值得注意的函数包括：

**addmod, mulmod**

用于模加法和模乘法。例如，addmod(x, y, k) 计算 $(x + y) \% k$。

**keccak256, sha256, ripemd160**

使用各种标准哈希算法计算哈希值的函数。

**ecrecover**

从签名中恢复用于签署消息的地址。

**selfdestruct(recipient_address)**

已弃用。用于删除当前合约并将账户中剩余的 Ether 发送到接收者地址。在 EIP-6780 实施后，只有当 selfdestruct 指令与合约创建在同一笔交易中被调用时，才会执行删除操作。在其他所有情况下，资金会被转移，但合约及其状态不会被清除。

**this**

指代当前合约，可显式转换为 Address 类型，以获取当前执行合约账户的地址：`address(this)`。

**super**

在继承体系中高一级别的合约。

**gasleft**

当前执行上下文中剩余的 Gas 数量。

**blockhash(block_number)**

由区块号指定的区块哈希值；仅可获取最近的 256 个区块。blobhash(index)：与当前交易关联的第 index 个 Blob 的哈希值。


### 合约定义
Solidity 的主要数据类型是合约（contract）；我们的 Faucet 示例简单地定义了一个合约对象。与面向对象语言中的任何对象类似，合约是一个包含数据和方法的容器。

除了合约之外，Solidity 还提供了另外两种与之相似的对象类型：

**接口 (interface)**

接口定义的结构与合约完全一致，不同之处在于其中不定义任何函数体——仅对函数进行声明。这类声明通常被称为存根（stub）；它会告知你函数的参数和返回类型，而不包含任何实现逻辑。接口规定了合约的“形状（shape）”；当接口被继承时，子合约必须实现接口中声明的每一个函数。

**库 (library)**

库合约旨在仅部署一次，并由其他合约通过 `delegatecall` 方法调用（参见“地址对象”部分）。

### 函数
在合约内部，我们定义的函数可以由 EOA 交易或其他合约调用。在我们的 Faucet 示例中，有两个函数：withdraw（提款）和 receive（接收）。

Solidity 中声明函数的语法如下：
```Solidity
function FunctionName([parameters]) {public|private|internal|external} [virtual|override]
[pure|view|payable] [modifiers] [returns (return types)]
```
让我们逐一查看这些组成部分：

**FunctionName（函数名）**

函数的名称，用于在来自 EOA、其他合约、甚至同一合约内部的交易中调用该函数。

**parameters（参数）**

紧跟在名称之后，我们需要指定必须传递给函数的参数及其名称和类型。在 Faucet 示例中，我们定义了 uint withdrawAmount 作为 withdraw 函数的唯一参数。

下一组关键字（public, private, internal, external）指定了函数的可见性：

**public** 

默认选项（但在现代 Solidity 中必须显式指定）。这类函数可以由其他合约、EOA 交易或在合约内部调用。

**private**

私有函数。类似于内部函数，但不能被派生合约（Derived Contracts）调用。

**internal**

内部函数。仅能从合约内部访问——不能被其他合约或 EOA 交易调用。但它们可以被派生合约（继承自该合约的合约）调用。

**external**

外部函数。类似于公共函数，但它们不能从合约内部直接调用，除非显式加上 this. 前缀。

请记住，internal 和 private 这两个术语有时会产生误导。合约内部的任何函数或数据在公共区块链上始终是可见的（Visible），这意味着任何人都可以查看其代码或数据。这里描述的关键字仅影响函数何时以及如何被调用（Called）。

> [!Note]
> 提示：函数可见性不应与状态变量可见性混淆！尽管它们共享关键字和语义，但却是两回事。状态变量的可见性是可选的，而函数的可见性必须显式定义。

第二组关键字（pure, view, payable）会影响函数的行为：

**pure**

纯函数。既不读取也不写入存储（Storage）中的任何变量。它只能操作参数并返回数据，而不引用任何存储数据或区块链状态。纯函数旨在鼓励无侧重影响或状态的声明式编程。

**view**

视图函数。承诺不修改任何状态。编译器不会强制执行 view 修饰符（译者注：现代版本编译器已强制检查），如果未遵守则会发出警告。

**payable**

可支付函数。指可以接收付款的函数。未声明为 payable 的函数将拒绝转入的资金。由于 EVM 的设计决定，有两个例外：Coinbase 付款和 SELFDESTRUCT 继承，即使回退函数未声明为 payable，这些款项也会被支付。

现在让我们探索两个特殊函数的行为：

**receive()**

接收函数。它是合约接收 Ether 的入口。当合约收到一个 calldata 为空的调用时触发，通常见于普通的 Ether 转账（如使用 .send() 或 .transfer()）。其声明方式为 receive() external payable { ... }。它不能有参数或返回值，但可以是 virtual 的，可以 override（重写），并带有修饰符。

**fallback()**

回退函数。当调用的函数签名与合约中任何其他函数都不匹配时执行。我们可以使用 `fallback() external [payable]` 或 `fallback(bytes calldata input) external [payable] returns (bytes memory output)` 进行声明。如果包含输入参数，它将包含发送给合约的完整数据（等同于 msg.data）。目前，它主要用于实现代理模式（Proxy Pattern）：一种支持智能合约可升级的设计模式。

> [!Note]
> 如果合约缺少 receive 函数，但定义了声明为 payable 的 fallback 函数，那么在收到转账时将执行 fallback 函数。如果合约既没有 receive 函数，也没有 payable 的 fallback 函数，则该合约无法接收以太币，交易将因异常而回滚（revert）。

### 合约构造函数
有一个特殊的函数仅会被使用一次。当合约被创建时，如果存在构造函数（constructor），它会运行该函数以初始化合约的状态。构造函数与合约创建处于同一笔交易中。构造函数是可选的；你会注意到我们的 Faucet 示例中并没有定义它。

构造函数通过 constructor 关键字进行指定。其形式如下：
```Solidity
pragma 0.8.26
// SPDX-License-Identifier: GPL-3.0
contract MEContract {
 address owner;
 constructor () { // This is the constructor
  owner = msg.sender;
 }
}
```
合约的生命周期始于由 EOA（外部持有账户）或合约账户发起的创建交易（Creation transaction）。如果定义了构造函数，它将作为合约创建过程的一部分被执行，以便在创建时初始化合约状态；执行完毕后，构造函数代码会被丢弃。

> [!Note]
> 构造函数也可以标记为 payable。如果你希望在合约创建交易中同时发送 ETH，这一点是必需的。如果构造函数未声明为 payable，那么在部署期间发送任何 ETH 都会导致交易回滚（revert）。

### 函数修饰器 (Function Modifiers)
Solidity 提供了一种特殊类型的函数，称为函数修饰器（function modifier）。你只需在函数声明中添加修饰器名称，即可将其应用于该函数。修饰器最常用于创建适用于合约内多个函数的先决条件（Conditions）。在我们的 destroy 函数中已经使用了一条访问控制语句。现在，让我们创建一个函数修饰器来表达这一条件：
```Solidity
modifier onlyOwner {
 require(msg.sender == owner);
 _;
}
```
这个名为 onlyOwner 的函数修饰器为其所修饰的任何函数设置了一个条件：要求合约中存储的 owner（所有者）地址必须与交易的 msg.sender 地址一致。这是访问控制（Access Control）的基本设计模式，它确保只有合约的所有者才能执行带有 onlyOwner 修饰器的函数。

你可能已经注意到，我们的函数修饰器中有一个独特的语法“占位符”：下划线后跟一个分号 (_;)。这个占位符会被所修饰函数的实际代码所替换。从本质上讲，修饰器是“包装（Wrapped around）”在被修饰函数之外的，并将其代码放置在下划线字符所在的位置。

要应用修饰器，只需将其名称添加到函数声明中即可。一个函数可以应用多个修饰器；它们会按照声明的顺序执行，中间以逗号分隔。

让我们定义一个 changeOwner 函数来演示如何使用 onlyOwner 修饰器：
```Solidity
function changeOwner(address newOwner) public onlyOwner {
 require(newOwner != address(0), "New owner address not set");
 owner = newOwner;
}
```
函数修饰器的名称（onlyOwner）紧随关键字 public 之后，这表明 changeOwner 函数受到了 onlyOwner 修饰器的约束。本质上，你可以将其理解为“只有所有者（Owner）才能设置新的所有者地址”。在实际执行中，生成的代码等同于将 onlyOwner 的逻辑“包装”在 changeOwner 的代码之外。

函数修饰器是一种极其有用的工具，因为它们允许我们为函数编写先决条件（Preconditions）并一致地应用它们，从而使代码更易于阅读，并因此更易于进行安全性审计（Audit）。虽然它们最常用于访问控制，但其功能非常多样，还可以用于许多其他用途。

### 合约继承 (Contract Inheritance)
Solidity 的合约对象支持继承（inheritance），这是一种通过额外功能扩展基类合约（Base contract）的机制。要使用继承，请使用关键字 `is` 来指定父合约：
```Solidity
contract Child is Parent {
 ...
}
```
通过这种构造，子合约（Child contract）将继承父合约（Parent）的所有方法、功能和变量。Solidity 还支持多重继承（Multiple inheritance），可以通过在关键字 is 之后以逗号分隔合约名称来指定：
```Solidity
contract Child is Parent1, Parent2 {
 ...
}
```
我们可以通过显式指定合约（如 Parent1.functionName()）来调用继承链中更高级别的函数；如果我们想调用扁平化继承层级（Flattened inheritance hierarchy）中上一层的函数，也可以使用 super.functionName()。

合约继承使我们能够以模块化、可扩展性和复用性为目标编写合约。我们可以从实现最通用功能的简单合约开始，然后通过在更具体的合约中继承这些功能来进行扩展。

在我们的 Faucet 合约中，我们引入了在构造时分配所有者的访问控制功能。这种功能非常通用，许多合约都会用到。我们可以将其定义为一个通用合约，然后通过继承将其扩展到 Faucet 合约中。为了丰富这个示例，让我们将可暂停（Pausable）功能与访问控制功能结合在一起。

我们首先定义一个基类合约 Owned，它拥有一个 owner 变量，并在合约的构造函数中对其进行设置：
```Solidity
contract Owned {
    address owner;
    // Contract constructor: set owner
    constructor() {
        owner = msg.sender;
    }
    // Access control modifier
    modifier onlyOwner {
        require(msg.sender == owner);
        _;
    }
}
```
接下来，我们定义一个继承自 Owned 的基类合约 Pausable：
```Solidity
contract Pausable is Owned {
    bool paused;
    // Status check modifier
    modifier whenNotPaused {
        require(paused == false);
        _;
    }
    // Functions to pause/unpause user operations
    function pause() public onlyOwner {
        paused = true;
    }
    function unpause() public onlyOwner {
        paused = false;
    }
}
```
正如你所看到的，Pausable 合约可以使用在 Owned 中定义的 onlyOwner 函数修饰器。它也间接地使用了 Owned 中定义的 owner 地址变量及其构造函数。继承让每个合约都变得更加简洁，并专注于其特定的功能，使我们能够以模块化的方式管理细节。

现在我们可以进一步扩展 Owned 合约（译注：此处根据上下文应为扩展 Pausable 合约），在 Faucet 中继承其功能：
```Solidity
contract Faucet is Pausable {
    // Give out ether to anyone who asks
    function withdraw(uint _withdrawAmount, address payable _to) public whenNotPaused {
        // Limit withdrawal amount
        require(_withdrawAmount <= 0.1 ether);
        // Send the amount to the address that requested it
        _to.transfer(_withdrawAmount);
    }
    // Accept any incoming amount
    receive() external payable {}
}
```
通过继承 Pausable（进而继承 Owned），Faucet 合约现在可以使用 whenNotPaused 修饰器，而该修饰器的开关状态可由 Owned 合约构造函数所定义的所有者（Owner）来控制。其功能效果与这些函数直接写在 Faucet 内部完全一致，但得益于这种模块化架构（Modular architecture），我们可以在其他合约中复用这些函数和修饰器，而无需重复编写。代码复用和模块化使我们的代码更简洁、更易读，也更易于审计。

有时我们可能需要修改继承合约中的某些功能。幸运的是，Solidity 为我们提供了相应的功能：函数重写（Function Overriding）。声明为 virtual 的函数可以被继承链中更高级别的合约所重写（Override），这使得继承方案保持了极高的灵活性。

让我们看一个例子。假设我们想让“可暂停”功能变成单向的：一旦暂停，合约就再也无法取消暂停。为了实现这一点，我们需要将 Pausable 合约中的 unpause 函数标记为 virtual，并在 Faucet 合约中重新声明 unpause 函数，并加上 override 属性，定义我们想要的新行为——即回滚（revert）：
```Solidity
contract Pausable is Owned {
    bool paused;
    // Status check modifier
    modifier whenNotPaused {
        require(paused == false);
        _;
    }
    // Functions to pause/unpause user operations
    function pause() public virtual onlyOwner {
        paused = true;
    }
    function unpause() public virtual onlyOwner {
        paused = false;
    }
}
contract Faucet is Pausable {
    // Give out ether to anyone who asks
    function withdraw(uint _withdrawAmount, address payable _to) public whenNotPaused {
        // Limit withdrawal amount
        require(_withdrawAmount <= 0.1 ether);
        // Send the amount to the address that requested it
        _to.transfer(_withdrawAmount);
    }
    function unpause() public view override onlyOwner {
        revert("Disabled feature”);
    }
    // Accept any incoming amount
    receive() external payable {}
}
```

如你所见，Faucet 中的 unpause() 函数必须使用 override 关键字进行声明。在 Pausable 合约中，为了保持一致性，我们将 pause 和 unpause 函数都标记为了 virtual，尽管在本例中我们仅需要修改 unpause。

> [!Note]
> 在 Solidity 中重写函数时，我们只能将可见性修改得更加开放——具体来说，可以将 external 修改为 public，但反之则不行。对于可变性（Mutability），我们可以将其收紧，例如从 nonpayable 改为 view 或 pure（正如我们在 unpause 中所做的），以及从 view 改为 pure。但有一个重要的例外：如果一个函数被标记为 payable，则必须保持不变——我们不能将其修改为任何其他类型。

### 多重继承 (Multiple Inheritance)

当我们在 Solidity 中使用多重继承时，它依赖于一种称为 C3 线性化（C3 linearization） 的算法来确定合约继承的顺序。该算法确保了继承顺序是严格且可预测的，这有助于避免诸如循环继承（Cyclic inheritance）之类的问题。简单来说，它确定了在查找某个函数时检查基类合约的先后顺序，而这个顺序是从右到左的。这意味着位于最右侧的合约被视为“最派生（Most derived）”的合约。例如，在合约声明 `contract C is A, B { }` 中，合约 B 比合约 A 更具派生性。

除了使用 C3 线性化之外，Solidity 还设有额外的防护机制。其中一项核心规则是：如果多个父合约拥有同名函数，我们必须显式地声明正在重写哪些合约。让我们来看一个例子：
```Solidity
contract A {
    function foo() public virtual returns(string memory){
        return "A";
    }
}
contract B {
    function foo() public virtual returns(string memory){
        return "B";
    }
}
contract C is A, B {
}
```
乍一看，由于 C3 线性化算法理应处理好所有逻辑，这段代码似乎可以正常工作。但在现实中，它无法通过编译。Solidity 会抛出一个错误：“类型错误（TypeError）：派生合约必须重写函数 foo。两个或多个基类定义了具有相同名称和参数类型的函数。” 为了解决这个问题，我们需要像下面这样，显式地重写来自 A 和 B 的 foo() 函数：
```Solidity
contract C is A, B {
    function foo() public override(A, B) returns(string memory){
        return "C";
    }
}
```
因此，尽管 Solidity 使用了 C3 线性化算法，但在编码时大部分情况下我们并不需要担心它，因为 Solidity 会强制我们显式地处理函数重写。

然而，C3 线性化算法真正发挥重要作用的一个地方是：当 Solidity 决定构造函数的执行顺序时。构造函数遵循 C3 线性化的顺序，但有一个特别之处：它们是逆序执行的。如果你仔细思考，这其实很合理：**最派生（most derived）**合约的构造函数应该最后运行，因为它可能会重写（或覆盖）早期构造函数所设置的内容。让我们来看一个例子：
```Solidity
contract Base{
    uint x;
}
contract Derived1 is Base{
    constructor(){
        x = 1;
    }
}
contract Derived2 is Base{
    constructor(){
        x = 2;
    }
}
contract Derived3 is Derived1, Derived2 {
    uint public y;
    constructor() Derived1() Derived2() {
        y = x;
    }
}
```
在这种情况下，变量 y 的最终值将如预期般为 2，因为 Derived2 的构造函数最后运行，并将 x 设置为 2。

需要特别记住的是，你提供构造函数参数的顺序并不会影响执行顺序。例如，即使我们像这样调换构造函数的调用顺序：
```Solidity
contract Derived3 is Derived1, Derived2 {
    uint public y;
    constructor() Derived2() Derived1() { // we switched the order here
        y = x;
    }
}
```
即使我们改变了构造函数中参数传递的顺序，结果依然是一样的。变量 y 的值仍将为 2，因为构造函数的执行顺序是由 C3 线性化算法决定的，而不是由我们调用构造函数的先后顺序决定的。

最后一点提醒：Solidity 在多重继承中使用 C3 线性化，可能会导致 super 关键字的行为超出你的预期。有时，调用 super 可能会触发**同级类**（Sibling class）中的函数，而不是直接父类中的函数。这可能会导致一些令人惊讶的结果——某个方法居然是从一个你甚至没有直接列在继承链中的类里被调用的。这属于一种边际情况（Edge case），所以我们不会深入探讨，但在处理具有复杂继承结构的合约时，请务必记住这一点。

### 错误处理 (Error Handling)

合约调用可能会终止并返回错误。Solidity 中的错误处理由三个函数负责：assert、require 和 revert。

当合约因错误终止时，所有的状态更改（对变量、余额等的修改）都会被回滚（revert）。如果调用涉及多个合约，这种回滚会沿着合约调用链向上追溯。这确保了交易是原子性的（atomic），意味着交易要么全部成功完成，要么对状态不产生任何影响并被彻底撤销。

assert 和 require 函数的运行方式相同：评估一个条件，如果条件为假（false），则停止执行并报错。按照惯例，assert 用于预期结果为真的情况，这意味着我们使用 assert 来测试内部条件（如内部不变量）。相比之下，require 用于测试输入数据（如函数参数或交易字段），以此设定我们对这些条件的预期。

值得注意的是，assert 在失败时的行为与 require 不同：它会耗尽所有剩余的 Gas。这使得它在被触发时更加昂贵，这也是为什么我们通常将其保留给那些“绝不应该被破坏”的不变量（Invariants）。

我们在 onlyOwner 函数修饰器中已经使用了 require，用来测试消息发送者是否为合约的所有者：
```Solidity
require(msg.sender == owner);
```
`require` 函数充当了**门控条件（gate condition）**的作用，如果条件不满足，它会阻止函数其余部分的执行并生成错误。它还可以包含一段有用的文本信息，用于说明错误原因。该错误信息会被记录在交易日志中。建议采用这种做法，以便让用户了解错误原因及修复方法，从而提升用户体验。因此，我们可以通过在 require 函数中添加错误信息来改进代码：
```Solidity
require(msg.sender == owner, "Only the contract owner can call this function");
```
`revert` 函数会停止合约执行并回滚任何状态更改。它有两种使用方式：一种是作为语句直接传递自定义错误（Custom Error）（不带括号），另一种是作为函数带括号并接收一个字符串参数。在 Gas 消耗方面，自定义错误要便宜得多，而错误字符串和自定义错误都会被记录在交易日志中：
```Solidity
revert();
revert("Error string");
revert CustomError(arg1, arg2);
```
合约中的某些条件无论我们是否显式检查，都会产生错误。例如，在我们的 Faucet 合约中，我们并没有检查是否有足够的以太币来满足取款请求。这是因为如果余额不足以进行转账，transfer 函数本身就会失败并报错，从而回滚交易：
```Solidity
payable(msg.sender).transfer(withdrawAmount);
```
然而，进行**显式检查**（Explicit check）并在失败时提供清晰的错误信息可能会更好。我们可以通过在转账之前添加一条 require 语句来实现：
```Solidity
require(this.balance >= withdrawAmount,
 "Insufficient balance in faucet for withdrawal request");
payable(msg.sender).transfer(withdrawAmount);
```
像这样额外的错误检查代码会稍微增加 Gas 消耗，但与省略相比，它提供了更好的错误报告。虽然由于以太坊主网过去的高昂成本，最小化 Gas 消耗曾是一项强制性任务，但 EIP-4844 的引入显著降低了成本，使得 Gas 消耗在今天不再是一个紧迫的问题。然而，在 Gas 效率与详尽的错误检查之间取得平衡仍然非常重要。

Solidity 通过 try/catch 功能为我们提供了对错误处理更强的控制力。这是一个非常实用的特性，让我们在调用外部合约时能更优雅地处理错误。当出现问题时，我们的整个交易不会直接失败并回滚，我们可以捕获错误并决定下一步该怎么做。使用 try/catch 时，我们基本上是将外部调用包装在一个 try 块中。如果调用成功，try 块内的代码正常执行。但如果出了问题——比如被调用的合约耗尽了 Gas、触发了 require 语句或抛出了异常——代码会跳转到 catch 块，我们可以在那里处理错误。

这有个简单的例子：
```Solidity
function sampleExternalCall(address target, uint amount) public {
    try ITargetContract(target).someFunction(amount) {
        // This runs if the call is successful
        emit Success("Call succeeded!");
    } catch {
        // This runs if the call fails
        emit Error("Call failed!");
    }
}
```
我们可以根据错误类型以不同的方式捕获错误。基础的 catch 块会捕获所有错误，但我们也可以捕获特定错误。例如，我们可以使用 catch Error(string memory reason) 捕获返回错误字符串的错误，或者使用 catch (bytes memory lowLevelData) 处理不返回数据的低级别错误。此外，我们还可以使用 catch Panic(uint errorCode) 捕获更严重的 Panic 错误（如溢出或除以零）。

需要注意的是，try/catch 仅适用于外部调用。它对同一个合约内的内部函数调用（Internal calls）没有帮助。如果同一合约内的函数执行失败，它仍会像往常一样回滚，我们无法使用 try/catch 来捕获。


### 事件（events）
当交易完成时（无论成功与否），都会产生一份交易回执（Transaction receipt）。交易回执中包含日志条目（Log entries），这些条目提供了有关交易执行期间所发生操作的信息。**事件**（Events）是 Solidity 中的高级对象，用于构建这些日志。

事件对于轻客户端（Light clients）和 DApp 服务尤其有用，它们可以“监听”（Watch）特定事件，并将其报告给用户界面，或者更改应用程序的状态，以反映底层合约中发生的事件。

事件对象接收参数，这些参数会被序列化并记录在区块链的交易日志中。你可以在参数前加上关键字 indexed，使该值成为索引表（哈希表）的一部分，从而方便应用程序进行搜索或过滤。




























