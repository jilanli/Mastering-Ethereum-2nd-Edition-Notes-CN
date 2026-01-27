# 第八章：智能合约与 Vyper

Vyper 是一种成熟的面向 EVM 的合约编程语言，它致力于通过提高代码的可读性，为开发者提供更卓越的可审计性。事实上，Vyper 的核心原则之一就是让开发者几乎不可能写出具有误导性的代码。

在本章中，我们将探讨智能合约中的常见问题，介绍 Vyper 合约编程语言，并将其与 Solidity 进行对比，展示两者之间的差异。
## 漏洞与 Vyper
仅在 2023 年，以太坊生态系统中就有近 20 亿美元因智能合约漏洞而被盗。漏洞是通过代码引入智能合约的。可以有力地论证，这些以及其他漏洞并非开发者刻意引入，但无论如何，设计不当的智能合约代码显然导致了以太坊用户资金的意外损失，这并不理想。Vyper 的设计初衷是让编写安全代码变得更容易，或者同样地，让无意中写出具有误导性或脆弱的代码变得更难。

## 与 Solidity 的对比
Vyper 尝试降低不安全代码编写难度的途径之一，是刻意省略了 Solidity 的一些特性。这一设计选择反映了 Vyper 以安全为首的原则，及其从 Python 的清晰与简洁中汲取的灵感。对于考虑在 Vyper 中开发智能合约的人来说，理解 Vyper 不具备哪些特性以及原因至关重要。在本节中，我们将探讨这些被省略的特性并给出合理解释。

尽管通过削减功能集来减少歧义，但 Vyper 也在不断进化，以满足开发者和审计员的实际需求。例如，最初将合约限制在单个文件中的哲学有助于最大化可审计性，但随着协议变得庞大且复杂，这最终成为了瓶颈。为了解决这一问题，现代 Vyper 引入了一套成熟的模块系统，允许开发者将合约拆分为多个文件，同时保持对状态访问和代码复用的严格控制。这套模块系统遵循的是**组合**（Composition）原则而非传统的继承，在结构化与可读性之间取得了更好的平衡。Vyper 在高安全性要求的用例中扮演着越来越重要的角色，如去中心化金融（DeFi）协议和质押系统，在这些领域，开发者极度强调代码的清晰度与审计的便捷性。

### 修饰器 (Modifiers)
正如我们在第 7 章中看到的，在 Solidity 中你可以使用修饰器来编写函数。例如，下方的 changeOwner 函数在执行过程中会运行一个名为 onlyBy 的修饰器代码：
```Solidity
function changeOwner(address _newOwner)
 public
 onlyBy(owner)
{
 owner = _newOwner;
}
```
这个修饰器强制执行了一条关于所有权的规则。如你所见，这个特定的修饰器充当了一种代表 changeOwner 函数进行预检查的机制：
```Solidity
modifier onlyBy(address _account)
{
 require(msg.sender == _account);
 _;
}
```
但修饰器并不仅仅用于执行检查。事实上，作为修饰器，它们可以在调用函数的上下文中显著改变智能合约的运行环境。简单来说，修饰器具有渗透性 (Pervasive)。

让我们看另一个 Solidity 风格的例子：
```Solidity
enum Stages {
 SafeStage,
 DangerStage,
 FinalStage
}
uint public creationTime = block.timestamp;
Stages public stage = Stages.SafeStage;
function nextStage() internal {
 stage = Stages(uint256(stage) + 1);
}
modifier stageTimeConfirmation() {
 if (stage == Stages.SafeStage &&
 block.timestamp >= creationTime + 10 days)
 nextStage();
 _;
}
function a()
 public
 stageTimeConfirmation
 // More code goes here
{
}
```
一方面，开发者应当始终检查其代码调用的任何其他代码。然而，在某些特定情况下（例如时间紧迫或因疲劳导致注意力不集中），开发者可能会忽略掉某一行代码。如果开发者必须在一个庞大的文件中反复跳转，同时还要在大脑中跟踪函数调用层级并记住智能合约变量的状态，这种情况发生的可能性就会更高。

让我们深入分析前面的例子。假设一名开发者正在编写一个名为 a 的公共函数。由于这名开发者刚接触该合约，他使用了一个由他人编写的修饰器。乍一看，stageTimeConfirmation 修饰器似乎只是针对调用函数执行一些关于合约生存时间的检查。但开发者可能没有意识到的是，该修饰器还调用了另一个函数 nextStage。在这个简单的演示场景中，仅仅调用公共函数 a，就会导致智能合约的 stage 变量从 SafeStage 变为 DangerStage。

Vyper 已经完全废除了修饰器。 Vyper 的建议如下：如果你只是利用修饰器进行断言（Assertion）检查，那么直接将这些检查和断言作为函数体的一部分写入；如果你需要修改智能合约的状态或执行类似操作，同样应当将这些更改显式地作为函数的一部分。这样做提高了可审计性和可读性，因为读者不需要在脑海中（或手动地）将修饰器代码“包裹”在函数周围来推断其具体行为。

### 类继承
继承允许程序员通过获取现有软件库的功能、属性和行为，来利用预先编写好的代码。继承功能强大，能够促进代码重用。Solidity 不仅支持多重继承，还支持多态性；尽管这些是面向对象编程（OOP）的核心特性，但 Vyper 并不支持它们。

Vyper 坚持认为，继承的实现强制要求编码者和审计员在多个文件之间反复跳转，才能理解程序的真实行为。此外，Vyper 还认为多重继承会使代码变得过于复杂而难以理解——这一观点在 Solidity 官方文档中也得到了默契的承认，文档中给出了一个关于多重继承如何导致问题的典型示例（注：即著名的“钻石继承”问题）。 

### 内联汇编
内联汇编赋予了开发者直接访问 EVM（以太坊虚拟机）底层的权限，允许 Solidity 程序通过直接调用 EVM 指令来执行操作。例如，以下汇编代码将内存地址`0x80`处的值加 3：
```Solidity
3 0x80 mload add 0x80 mstore
```
这种做法会导致开发者无法通过搜索变量名来定位该变量被读取或修改的所有位置。Vyper 认为，为了获得这种额外的底层控制力而牺牲可读性，代价过于高昂，因此不支持内联汇编。

### 函数重载
函数重载允许开发者编写多个同名的函数。在特定场合下具体使用哪一个函数，取决于所提供的参数类型。以下面两个函数为例：
```Solidity
function f(uint256 _in) public pure returns (uint256 out) {
 out = 1;
}
function f(uint256 _in, bytes32 _key) public pure returns (uint256 out) {
 out = 2;
}
```
第一个函数（命名为 f）接收一个 uint256 类型的输入参数；第二个函数（同样命名为 f）接收两个参数，一个是 uint256 类型，另一个是 bytes32 类型。拥有多个同名但接收不同参数的函数定义可能会令人困惑，因此 Vyper 不支持函数重载。

### 变量类型转换
与 Solidity 相比，Vyper 在类型转换上采取了截然不同的方法，优先考虑显式且安全的类型处理而非开发便捷性。该语言要求所有类型转换必须使用内置的 convert() 函数显式完成，这确保了开发者始终清楚数据类型何时以及如何被修改。

我们可以将类型转换分为两类：可能丢失信息的转换和不会丢失信息的转换。Vyper 的 convert() 函数能够处理这两种情况，但始终带有显式的边界检查，以防止非预期行为。例如，当从较大的整数类型转换为较小的整数类型时，如果数值超出了目标类型的边界，Vyper 将回滚（Revert）该交易。

Vyper 中的类型转换语法非常直观：
```Solidity
# Converting between integer types
small_value: uint8 = 42
large_value: uint256 = convert(small_value, uint256)  # Safe upcast
back_to_small: uint8 = convert(large_value, uint8)   # Bounds-checked downcast
```
这种显式方法意味着，虽然 Vyper 代码在处理类型转换时可能比 Solidity 更冗长，但它也安全得多。由于每次转换都必须是刻意且显式的，因此不存在意外截断数值或出现非预期溢出行为的可能性。如果转换会导致数据丢失，或者输入值超出了目标类型的有效范围，convert() 函数将回滚交易。

## 装饰器（Decorators）
在每个函数开始处可以使用以下装饰器：

**@internal**

使函数无法从合约外部访问。这是函数的默认可见性，因此该装饰器是可选的。

**@external**

使函数公开可见并可执行。例如，在查看合约时，以太坊钱包也会显示此类函数。

**@view**

带有此装饰器的函数不允许修改状态变量。事实上，如果函数尝试修改状态变量，编译器将拒绝编译整个程序（并抛出相应错误）。

**@pure**

带有此装饰器的函数不允许读取任何区块链状态，也不允许调用非 pure 方法或其他合约。

**@payable**

只有带有此装饰器的函数才被允许接收以太币（Value）。

**@deploy**

用于标记合约的构造函数（Constructor）。该函数在合约部署到区块链时仅运行一次，通常用于设置初始状态变量和配置。在 Vyper 中，只有 __init__() 函数可以被标记为 @deploy；如果你想在合约中包含构造逻辑，则必须使用此装饰器。

**@raw_return**

带有此装饰器的函数返回原始字节（Raw Bytes），而不应用 ABI 编码。这在代理合约（Proxy）和辅助合约中特别有用，因为你需要转发另一个合约调用的准确输出，而不希望对其进行二次编码。它有重要限制：只能用于 @external 函数，不能用于接口定义，且从其他合约调用此类函数时，应使用 raw_call 而非接口调用。

**@nonreentrant**

该装饰器为函数加上“锁”，防止对受此装饰器保护的任何函数进行重入调用（Reentrant Calls）。重入锁确保这些函数在执行完成前无法再次进入。它被用来防御重入攻击。Vyper 还支持默认非重入的编译指令，使所有外部函数默认具备此特性。例如，如果合约 A 使用此装饰器进行保护并向合约 B 发起外部调用，合约 B 任何试图回调合约 A 受保护函数的尝试都会导致交易回滚。

> [!Note]
> Vyper 版本 0.2.15、0.2.16 和 0.3.0 中的非重入（nonreentrant）特性曾包含一个严重漏洞。该漏洞于 2023 年中期被发现，并被用于攻击 Curve 协议。我们将在第 9 章深入探讨重入漏洞。

Vyper 对装饰器的逻辑实施是显式且严格的。例如，如果一个函数同时拥有 `@payable` 装饰器和 `@view` 装饰器，Vyper 的编译过程将会失败。这合乎逻辑，因为根据定义，一个能够接收价值（Value）的函数必然更新了状态，因此不能是 `@view`。此外，每个 Vyper 函数必须被标记为 `@external` 或 `@internal`（两者必选其一且不能同时使用！）。

## 函数与变量的排序
Vyper 的作用域和声明方式遵循 C99 作用域规则，这比你最初预期的更具灵活性。虽然 Vyper 合约通常仍包含在单个文件中（除非使用模块系统），但早期版本中存在的严格排序要求在“模块级声明”中已经放宽了。

在模块作用域内（即任何函数体之外）声明的变量和函数在整个合约中都是可见的，甚至在它们正式声明之前即可使用。这意味着你可以在文件中变量或函数出现之前就引用状态变量或调用函数，类似于许多现代编程语言处理“前向声明”（Forward Declarations）的方式。

以下是一个展示现代 Vyper 作用域工作原理的实际示例：
```Solidity
# This function can reference the state variable below
@external
def get_stored_value() -> uint256:
    return self.stored_data  # References variable declared later
# This function can call the function above
@external
def check_if_positive() -> bool:
    return self.get_stored_value() > 0
# State variable declaration - accessible by functions above
stored_data: public(uint256)
```
然而，在函数作用域内，Vyper 依然保持着严格的排序规则。局部变量必须先声明后使用，并且局部变量不能遮蔽（Shadow）常量、不可变变量（Immutable）或其他模块级声明的名称。这种作用域处理方式在 Python 的灵活性与智能合约开发的需求之间取得了平衡，因为在合约开发中，清晰的状态变量可见性和函数关系对于安全审计至关重要。

## 编译
试验 Vyper 最简单的方法是使用 Remix 在线编译器。它允许你仅通过 Web 浏览器就能编写并编译智能合约（你需要在插件管理器中激活 vyper-remix 插件）。

> [!Note]
> Vyper 自带了[内置的通用接口](https://oreil.ly/GazPW)，例如 ERC-20 和 ERC-721，这使得开发者可以开箱即用地与这些合约进行交互。在 Vyper 中，合约实例必须被声明为全局变量。声明一个 ERC-20 变量的示例如下：`from vyper.interfaces import ERC20 token: ERC20`

你也可以使用命令行来编译合约。每个 Vyper 合约都保存为以 .vy 为后缀的文件。安装 Vyper 后，可以通过运行以下命令进行编译：
```Bash
vyper ~/hello_world.vy
```
编译器提供了丰富的输出选项。若要获取 JSON 格式且易于阅读的 ABI 描述，请使用：
```Bash
vyper -f abi ~/hello_world.vy
```
在开发和测试阶段，你可能还需要字节码（Bytecode）、操作码（Opcodes）或接口文件等额外输出。编译器支持多种输出格式：
```Bash
vyper -f abi,bytecode,interface,source_map ~/hello_world.vy
```
现代 Vyper 还引入了高级优化模式。你可以通过 `--optimize gas`（默认值）优化 Gas 效率，或通过 `--optimize codesize` 减小合约体积。使用 `--experimental-codegen` 可以启用实验性的 Venom IR 管道，从而获得更卓越的优化效果。

虽然 Remix 和命令行编译器非常适合学习和实验，但处理大型项目的开发者通常需要全面的开发框架。[ApeWorx](https://oreil.ly/AJRSa)（原名 Ape）为 Vyper 提供了出色的支持，具备自动化测试、部署脚本以及多网络集成等功能。[Foundry](https://oreil.ly/CWIKj) 虽然主要侧重于 Solidity，但也支持 Vyper 开发，并提供强大的测试和模拟功能。这些框架为专业智能合约开发者构建复杂应用提供了所需的成熟开发环境。

## 在编译器层面防御溢出错误
在涉及真实价值时，软件中的溢出错误可能是灾难性的。例如，[2018 年 4 月中旬的一笔交易](https://oreil.ly/zOy06)显示，超过 $5.7 \times 10^{58}$ 个 BEC 代币被恶意转账。这笔交易是由 Beauty Chain 的 ERC-20 代币合约（BecToken.sol）中的一个整数溢出问题导致的。

Vyper 的核心特性之一始终是其内置的溢出保护，这降低了历史上一直困扰智能合约开发的溢出错误风险。Vyper 的溢出保护方法非常全面：它包含等同于 SafeMath 的保护机制，能处理整数算术中必要的异常情况，确保加、减、乘、除等运算默认安全，并在发生溢出或下溢时抛出异常。此外，每当加载字面量常量、将值传递给函数或为变量赋值时，Vyper 都会使用**夹具**（Clamps）来强制执行数值限制。

值得注意的是，最近版本的 Solidity（0.8.0 及更高版本）也在编译器层面集成了原生溢出检查，这与 Vyper 从一开始就提供的一样。这意味着现代 Solidity 中的算术运算现在自动包含溢出检查，无需 SafeMath 等额外库即可显著降低溢出风险。虽然这一变化使 Solidity 更接近 Vyper “安全第一”的方法，但 Vyper 的实现仍然更为全面，包括了夹具操作和全语言范围内更一致的边界检查。核心区别在于哲学：Vyper 从底层设计之初就将溢出保护作为核心原则，而 Solidity 是作为一种增强功能来添加它，以解决历史漏洞。这种方法上的差异反映了 Vyper 更广泛的承诺：通过默认设置让编写不安全的代码变得更难。

## 读取与写入数据
尽管存储、读取和修改数据的成本很高，但这些存储操作是大多数智能合约不可或缺的组成部分。智能合约可以将数据写入两个地方：

**全局状态 (Global State)**

给定智能合约中的状态变量存储在以太坊的**全局状态树**（Global State Trie）中。智能合约只能存储、读取和修改与其特定合约地址相关的数据（即智能合约不能直接读写其他智能合约的状态）。

**日志 (Logs)**

智能合约可以通过**日志事件**（Log Events）向以太坊的链数据写入信息。在 Vyper 中，声明和使用事件的语法简洁明了，符合 Vyper 对代码清晰度的追求。

Vyper 中的事件声明看起来与结构体（Struct）声明非常相似。例如，声明一个名为 MyLog 的事件可以这样写：
```Solidity
event MyLog:
    arg1: indexed(address)
    arg2: uint256
    message: indexed(bytes[100])
```
你可以拥有最多四个索引参数（Indexed Arguments）（这些参数会成为可搜索的“主题”），以及任意数量的非索引参数，后者将成为事件数据的一部分。索引参数对于过滤和搜索事件非常有用，而非索引参数则可以包含更大量的数据。

执行日志事件时使用 log 语句，其语法非常直观：
```Solidity
log MyLog(msg.sender, 42, b"Hello, Vyper!")
```
你也可以使用 pass 语句创建不带参数的事件：
```Solidity
event SimpleEvent: pass
# Later in your code:
log SimpleEvent()
```
虽然智能合约可以通过日志事件向以太坊链数据写入信息，但它们无法读取自己创建的链上日志事件。然而，通过日志事件写入数据的一个优势是，日志可以被公链上的轻客户端发现并读取。例如，已发布区块中的 logsBloom 值可以指示该区块中是否存在特定的日志事件。一旦确定了日志事件的存在，就可以从给定的交易收据（Transaction Receipt）中获取具体的日志数据。

## 结语
Vyper 是一种强大且引人入胜的面向合约的编程语言。其设计倾向于“正确性”，将安全性与简洁性置于首位。这种方法可以帮助程序员编写出更优质的智能合约，并避开那些可能导致严重漏洞的特定陷阱。

然而，重要的是要认识到，任何选择都有权衡。虽然 Vyper 严苛的设计原则增强了安全性及代码清晰度，但也限制了开发者在其他语言中可能获得的某些灵活性。此外，Vyper 的使用范围和成熟度尚不及 Solidity，这意味着开发者可利用的资源、库和工具相对较少。对于那些寻求社区支持、预构建解决方案和详尽文档的人来说，这可能会带来挑战。

接下来，我们将更详细地探讨智能合约的安全性。一旦你了解了智能合约中可能出现的所有安全问题，Vyper 设计中的一些细微差别将会变得更加清晰。









