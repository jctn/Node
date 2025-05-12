# Cursor官方指南：如何在Cursor中处理大型代码库？（附个人实践解读） - 知乎
最近发现 Cursor 的官方文档多了个 Guide 分类，其中有一篇主题为“**如何在 Cursor 中处理大型代码库**”的指南，内容有一定启发性，所以分享给大家。

：[https://docs.cursor.com/guides/advanced/large-codebases](https://link.zhihu.com/?target=https%3A//docs.cursor.com/guides/advanced/large-codebases)

![](https://pic3.zhimg.com/v2-78375a4b9d4f5f0f707874a8735abf20_1440w.jpg)

以下是这篇指南的中文翻译 & 我的实践解读（_解读部分会用斜体展示_）：

* * *

相比处理小型项目，处理大型代码库会遇到更多的新挑战。借鉴我们从 Cursor 自身代码库的扩展经验 & 客户管理大型代码库的洞察，我们发现了处理复杂代码库的有效模式。

在这份指南中，我们将介绍一些我们认为对处理大型代码库行之有效的方法。

![](https://picx.zhimg.com/v2-b58b103183bd26fdc867986ea0f15515_1440w.jpg)

**一、使用 Chat 快速熟悉陌生代码**
----------------------

如果你是第一次接触大型代码库，可能会比较有挑战。

相比用grep、搜索和点击来找到你需要的代码库特定部分。我们更建议你使用 Chat，你可以直接向 Cursor 提问来找到你的目标内容，并获得关于它是如何工作的详细解释。

_注：1、这里的 Chat 并不是 v0.45及更早版本的 Chat 模式，而且泛指和 Cursor 的对话，包括当前的 Ask、Agent、Manual；_

_2、grep 是一个经典的命令行工具，用于在文本中搜索指定的字符串或模式。_

在下面这条视频中，我们正在寻求 Cursor 帮助以找到代码库索引的实现细节，同时让它给出一些示例来帮助我们更好地理解。

[https://mintlify.s3.us-west-1.amazonaws.com/cursor/images/guides/advanced/large-codebases/qa.mp4](https://link.zhihu.com/?target=https%3A//mintlify.s3.us-west-1.amazonaws.com/cursor/images/guides/advanced/large-codebases/qa.mp4)

_除了用 Chat 来快速索引和理解代码库特定内容，我们还可以用 Chat 来快速了解项目结构，比如：_

```text
请分析项目代码库，用mermaid图表展示项目结构，以及各种文件的依赖关系等

```

_不过这种方法需要结合mermaid.live、[http://draw.io](https://link.zhihu.com/?target=http%3A//draw.io)等工具来实现可视化。_

_![](https://picx.zhimg.com/v2-51596215f67800277846754a9bb3cc8f_1440w.jpg)
_

_如果你需要熟悉的项目刚好在 GitHub 上，那么可以尝试用 Devin 推出的 [DeepWiki](https://zhida.zhihu.com/search?content_id=257450184&content_type=Article&match_order=1&q=DeepWiki&zhida_source=entity)（[www.deepwiki.com](https://link.zhihu.com/?target=http%3A//www.deepwiki.com/)）来熟悉项目。之前在知识星球也介绍过：_

_DeepWiki 可以理解为 GitHub 代码库的维基百科。原本的 Github 代码库只是简单地展示 README.md 文档，而 DeepWiki 会深入分析代码结构、文件、配置，帮我们生成以下这些内容，包括：_

*   _结构化的项目文档：像维基百科一样，清晰地列出项目概览、核心模块、关键概念等；_
*   _可交互的可视化mermaid图表：可以直接展示各种文件的依赖关系、模块的调用关系等；_
*   _智能问答：支持用自然语言直接提问关于代码库的任何具体问题。_

_![](https://pic3.zhimg.com/v2-a678f5afa3c66b2185ea3d91bcfeb748_1440w.jpg)
_

_DeepWiki 的使用也不麻烦：_

_1、直接在它的官网点击或者搜索任意 GitHub 仓库，就可以看到一个类似维基百科的介绍页；_

_2、或者在打开的 GitHub 仓库，直接修改它的URL，比如 [https://github.com/microsoft/playwright-mcp](https://link.zhihu.com/?target=https%3A//github.com/microsoft/playwright-mcp) ，只需要将URL中的GitHub替换成deepwiki，即 [https://deepwiki.com/microsoft/playwright-mcp](https://link.zhihu.com/?target=https%3A//deepwiki.com/microsoft/playwright-mcp)，同样可以得到对应 GitHub 仓库的一个维基百科介绍页。_

_如果不想跳出 Cursor 就使用 DeepWiki 能力，可以考虑 DeepWiki MCP，这是非官方推出的一个 MCP Server，它实现的能力就是通过 MCP 接收 DeepWiki 的URL，爬取所有相关页面，将它们转换为 Markdown 格式，并返回一个文档或页面列表。_

_：[https://github.com/regenrek/deepwiki-mcp](https://link.zhihu.com/?target=https%3A//github.com/regenrek/deepwiki-mcp)_

为了让 Cursor 更深入地理解代码库结构，建议在【Cursor Settings-Features】中启用【Include project structure】，以获得更好的性能。

![](https://pic2.zhimg.com/v2-3eb3e28ba3e32d2b5039bf937f60c415_1440w.jpg)

**二、编写特定领域知识的规则（Rules）**
------------------------

如果让你给新协作者介绍代码库，你会提供哪些背景信息以确保他们可以快速上手？

对于每个组织或项目，都存在一些隐性知识可能没有完全记录在你的文档中。而**有效地使用规则，可以让 Cursor 全面了解代码库**（包括这些隐性知识）**。** 

比如，你可以编写一条简短的规则，来告诉 Cursor 怎么实现新功能或服务：

```text
---
描述：添加一个新的 VSCode 前端服务
---
1. **接口定义：**
- 使用 `createDecorator` 定义一个新的服务接口，并确保包含 `_serviceBrand` 以避免错误。
2. **服务实现：**
- 在一个新的 TypeScript 文件中实现该服务，扩展 `Disposable`，并使用 `registerSingleton` 将其注册为单例。
3. **服务贡献：**
- 创建一个贡献文件来导入和加载该服务，并在主入口点注册它。
4. **上下文集成：**
- 更新上下文以包含新服务，从而允许在整个应用程序中访问。
---

```

又比如，你希望 Cursor 根据文件格式来遵守相应规则，就可以使用 [Auto Attached 模式](https://zhida.zhihu.com/search?content_id=257450184&content_type=Article&match_order=1&q=Auto+Attached+%E6%A8%A1%E5%BC%8F&zhida_source=entity)来实现。

```text
---
globs: *.ts
---
- 使用 bun 作为包管理器。相关脚本请参考 [package.json](mdc:backend/reddit-eval-tool/package.json)
- 文件命名统一使用 kebab-case（短横线连接的小写命名）
- 函数名与变量名统一使用 camelCase（驼峰命名法）
- 所有硬编码的常量使用 UPPERCASE_SNAKE_CASE（全大写+下划线分隔）
- 优先使用 `function foo()` 的函数定义方式，而非 `const foo = () =>`
- 使用 `Array<T>` 的形式，而不是 `T[]`
- 推荐使用具名导出（named exports），例如 `export const variable ...` 或 `export function ...`，避免使用默认导出（default export）

```

_关于Cursor Rules，我之前分享过系列文章，涵盖最早的.cursorrules，以及最新的Project Rules：_

_[cursor教程 | 如何根据不同项目写好一份合格的cursorrules?](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzIyMzk3MTEwNQ%3D%3D%26mid%3D2247484363%26idx%3D1%26sn%3D6f670f86a4994220076fd56bee3ce569%26scene%3D21%23wechat_redirect)_

_[.cursorrules将被移除，大家现在就可以迁移使用Project Rules，控制代码更精准](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzIyMzk3MTEwNQ%3D%3D%26mid%3D2247485250%26idx%3D1%26sn%3D60a91bb5d2938cc2896b490f4eb8fa27%26scene%3D21%23wechat_redirect)_

_[Cursor Rules在实际开发中的三种层级&实际应用（附20个常用Rules）](https://link.zhihu.com/?target=https%3A//t.zsxq.com/O7fds)_

_在早期，我对Cursor Rules的理解更多是给AI立规范。_

_因为Cursor中的各种大模型具有不同的编码特点，加之大模型的随机性，决定了我们和大模型的每次对话，其实就相当于和一个新的协作者在结对编程。这就好比团队每天会来一位新同事和你协作推进同一个项目，你需要反复给新同事介绍项目要求，这种沟通一定会把你给“逼疯”的。所以需要Cursor Rules来给AI立规范。_

_之前的这种理解，更多是放在个体和Cursor的结对编程上。随着对企业Cursor使用场景的了解，我发现Cursor Rules，不只是给AI的规范，也是确保团队成员在大型项目中使用统一的AI辅助标准。_

_值得一提的是，编程只是Cursor的应用场景之一，还有很多人活用它作为 AI IDE 在文本编辑上的优势，去做论文写作、小说创作、知识库整理等等，Cursor Rules在这些场景中就可以起到统一上下文的作用。_

**三、紧跟“计划制定”的过程**
-----------------

对于大的项目改动，前期投入足够时间去思考并制定一个精确、合理的计划，可以显著提高 Cursor 的输出质量。

当你发现提示词改来改去，依然没有得到自己想要的结果，不如退一步，从头开始创建一个更详细的计划，就像你给同事编写 PRD（Product Requirements Document，产品需求文档）一样。**通常，这里面最困难的部分是确定应该做出哪些改动，比较适合我们人类来完成**。有了正确的指令后，我们就可以将实施的部分交给 Cursor。

当然，AI也能帮忙制定计划。

你可以在 Cursor 中打开 Ask 模式，将你从项目管理系统、内部文档或零散的想法中拥有的任何上下文输入进去。想想你在代码库中要用到的文件和依赖项。这可能是一个包含你想要集成的代码片段的文件，也可能是一整个项目文件。

这是一个“让AI帮助制定计划”的示例prompt：

```text
制定一个关于如何创建新功能的计划（类似于 @existingfeature.ts）
如有任何问题，请向我提问（最多 3 个）
请务必搜索代码库
@Past Chats（我之前的探索提示）
以下是来自 [项目管理工具] 的更多背景信息：
[粘贴的工单描述]

```

我们可以要求模型通过“向人类提问”来制定计划并收集上下文，并且参考之前的探索提示和工单描述。建议使用类似于 [Claude-3.7-Sonnet](https://zhida.zhihu.com/search?content_id=257450184&content_type=Article&match_order=1&q=Claude-3.7-Sonnet&zhida_source=entity) 、[Gemini-2.5-Pro](https://zhida.zhihu.com/search?content_id=257450184&content_type=Article&match_order=1&q=Gemini-2.5-Pro&zhida_source=entity) 或 o3 这类推理模型，因为它们可以理解项目改动的意图并更好地制定计划。

![](https://pic2.zhimg.com/v2-e24a3b77f62899afb06cbd2d8c8ff901_1440w.jpg)

_关于在Cursor开发中大型项目，目前大家应用比较广泛的就是Plan&Act模式：_

*   _Plan模式：负责制定计划_
*   _Act模式：负责实施计划_

_之前分享的文章 [Cursor解决bug总在绕圈？可以尝试引入 Memory Bank](https://link.zhihu.com/?target=https%3A//t.zsxq.com/E1KCK)，就是介绍怎么在Cursor中应用Plan&Act模式来实现更高效地开发。_

```text
---
描述：
全局设置：
始终应用：true
------
## 模式规则
你有两种操作模式：
1、Plan 模式：你将与用户共同制定计划，收集所有必要信息以进行更改，但不会实际执行任何更改
2、Act 模式：你将根据计划对代码库进行实际修改
- 你默认以Plan模式开始，只有在用户批准计划后才会切换到Act模式
- 每次响应开头需标明当前模式，计划模式显示'# 模式：Plan'，执行模式'显示# 模式：执行'
- 除非用户明确输入指令要求切换至Act模式，否则你将始终保持Plan模式
- 每次响应后自动返回Plan模式，用户输入计划指令时也会切换回Plan模式
- 在Plan模式下若用户要求执行操作，你需要提醒当前处于Plan模式，需要先批准计划
- 在Plan模式下，每次响应都必须输出完整的更新后计划

```

_当然，这种方法也可以借助Cursor的自定义模式（即Manual模式）来实现：_

_![](https://picx.zhimg.com/v2-ce9a1c3699dbb43dac31af39975ccf3b_1440w.jpg)
_

_大家在【Cursor Settings-Features】中开启开启【Custom modes】后，就可以在对话框进行Plan模式和Act模式的配置，后续就可以根据自己的实际开发流程自由切换工作模式。_

_![](https://pic3.zhimg.com/v2-399e57cef68a54cde3ea274de43167f6_1440w.jpg)
_

_就像前面官方提到的，对于大的项目改动，前期投入足够时间去思考并制定一个精确、合理的计划是非常值得的。所以在Plan阶段，其实还可以进一步拆解为Research模式和Plan模式：_

*   _Research模式：主要做前期调研，包括需求收集、功能理解以及模拟实现_
*   _Plan模式：主要对前期调研的内容转化成具体可执行的计划，前面提到的PRD就是计划的一种展现形式_

_这里简单提下这三种模式的配置，大家可以根据自己的实际情况进行调整：_

_**Research模式**：模型可以选择Claude-3.7-Sonnet、Gemini-2.5-Pro等推理模型；tools可以按截图所示选择，其中MCP非必选；Advanced option的custom instructions可以复制使用下面的指令：_

_![](https://pic4.zhimg.com/v2-f5c4334f18ed3204f18a74f244df4149_1440w.jpg)
_

```text
在提出解决方案之前，从工作空间和代码库的多个来源中收集全面信息。
分析代码和近期变更，但不得自动修复问题。
不得修改任何代码。 如需使用代码展示解决方案，直接在回复中以纯Markdown文本格式提供。
在提供解决方案时，保留相关上下文信息（如文件路径、函数名或模块），以便用户理解。
避免基于不明确的假设进行分析或建议，必要时向用户请求澄清。
以一致的格式（如代码块、列表或标题）呈现分析结果和解决方案，便于用户快速阅读。

```

_**Plan模式**：和Search模式的配置基本一致，区别在于Advanced option的custom instructions：_

_![](https://picx.zhimg.com/v2-f2e49c5206a1da3382edc4160e4f3a31_1440w.jpg)
_

```text
**充分研究和审查**：
在开始制定计划前，需全面研究和审查所有相关细节，包括我们讨论过的内容、文档、代码库和外部资源。

**制定详细实施计划**：基于研究结果，创建详细的实施计划，但不直接修改代码，计划需要包含以下内容：
- 代码级别的变更指南，需完全基于代码库审查。
- 潜在风险分析及应对措施（如兼容性问题、性能影响）。
- 测试策略（如单元测试、集成测试）以验证变更效果。


**使用Mermaid图表**：对于复杂流程，使用Mermaid图表（流程图/时序图/状态图）进行说明：
- 使用清晰的标签和节点连接。
- 不同操作类型使用颜色编码（如输入为蓝色，处理为绿色，输出为橙色）。

**计划文件存储**：
- 所有计划文件必须存储在 .plans/ 目录下。
- 文件命名格式为 PLAN-{id}-{summary}.md：
  - {id} 为 .plans/ 目录及其子目录中的唯一编号。
  - {summary} 为任务的简短描述。
  - 文件采用 Markdown 格式，包含任务完成状态（如 [ ] 未完成，[x] 已完成）等。

```

_Act模式：一般可以直接选择Agent模式。_

**四、选择合适的工具来完成工作**
------------------

在有效使用 Cursor 的过程中，选择合适的工具至关重要。考虑你要达成的目标，并选择能让你保持流畅的工作方式的方法。

![](https://pic4.zhimg.com/v2-d29648cc7458cf3ac4c1336bfdf65c3d_1440w.jpg)

每个工具都有其优势：

*   使用 Tab 进行快速编辑，让你成为掌控者
*   当你需要对代码的特定部分进行专注的修改时，Cmd + K 会是神来之笔
*   对于需要 Cursor 理解更广泛上下文的大规模修改，Chat 模式再合适不过了

当你使用 Chat 模式（可能会感觉有点慢，但非常强大）时，通过提供良好的上下文来帮助它帮助你。使用@files 指向你想要参考的类似代码，或使用@folder 来更好地理解你的项目结构。并且不要害怕将更大的更改分解成更小的部分——从头开始聊天有助于保持事情专注和高效。

_这部分乍一看似乎和处理大型代码库没太大关系，但如果将这部分内容放到大型项目的代码重构中去重新理解，你就会发现它非常有实践价值。_

_对于大型项目的代码重构，一般没法一次性完成，拆分（如模块化）是比较常见的做法。当拆分后，大家就可以根据每个阶段、模块、步骤的特点来选择对应的工具，比如：_

_**初期规划阶段**：使用 **Chat 模式分析项目结构，识别需要重构的模块或文件。结合@folder 理解全局上下文，制定重构计划。** _

_**局部优化阶段：使用 Cmd + K** 针对特定代码块进行优化，如重构函数、提取公共逻辑或替换过时 API。_

_**快速调整阶段：使用 Tab** 补全重复性代码或修复小问题，保持编码流畅。_

_**验证与收尾：再次使用 Chat 模式**检查跨文件一致性，或用 **Cmd + K** 微调，确保重构后的代码符合预期。_

_根据任务规模和复杂性灵活切换工具，并结合上下文管理和任务分解，我们就可以在 Cursor 中高效完成大型项目重构，最大化工具的价值。_

**总结：** 

1、缩小变更范围，不要试图一次做太多；

2、有条件的话，对话中尽可能包含相关上下文；

3、使用 Chat、Cmd + K 和 Tab 进行它们最擅长的事情；

4、经常创建新的对话_（之前在知识星球也分享过，每次完成一个新功能，最好新开对话，避免之前的上下文对后面的功能开发造成污染）_；

![](https://pic3.zhimg.com/v2-0c01cd87b331a8dcc9c7ae236cedf37c_1440w.jpg)

5、使用 Ask 模式制定计划，使用 Agent 模式实施计划。

以上就是Cursor官方处理大型代码库的指南以及我的实践解决，希望对大家有启发。

可以搭配Cursor首席设计师 Ryo Lu 分享的 Cursor 正确用法一起阅读 >>>[Cursor官方下场谈Cursor正确用法，以及我的实践解读](https://zhuanlan.zhihu.com/p/1900338212569854320)