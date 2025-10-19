---
title: Agentic Coding：编程新范式
abbrlink: 58749
date: 2025-07-30 09:33:07
tags:
description:
categories:
cover: https://image.aruoshui.fun/i/2025/10/18/x49mxg-0.webp
swiper_index:
---

# 参考文章
{% link 综述：AI Agent 与 Agentic AI 有什么区别？,https://zhuanlan.zhihu.com/p/1908131472205930839,https://image.aruoshui.fun/i/2025/02/13/o4yax8-0.webp%} 

{% link Vibe Coding vs. Agentic Coding: Fundamentals and Practical Implications of Agentic AI, https://arxiv.org/pdf/2505.19443, https://image.aruoshui.fun/i/2025/10/18/x606cw-0.webp%}

{% link AI Agents vs. Agentic AI: A Conceptual taxonomy applications and 
challenges, https://arxiv.org/pdf/2505.10468, https://image.aruoshui.fun/i/2025/10/18/x606cw-0.webp%} 


{% link 综述：一文讲透Vibe Coding的机理和能力边界,https://zhuanlan.zhihu.com/p/1922579978085726171,https://image.aruoshui.fun/i/2025/02/13/o4yax8-0.webp%}

{% link AI守护者：Anthropic——技术革新与道德引领的双轨传奇,https://zhuanlan.zhihu.com/p/1402564536,https://image.aruoshui.fun/i/2025/02/13/o4yax8-0.webp%}

{% link Anthropic官方揭秘内部团队如何使用 Claude Code（附完整版手册）,https://zhuanlan.zhihu.com/p/1914366776751420120,https://image.aruoshui.fun/i/2025/02/13/o4yax8-0.webp%}

{% link CSDN, https://zhuanlan.zhihu.com/p/666861211, https://image.aruoshui.fun/i/2025/01/02/ni2zi4-0.webp%} 

{% link Claude Data Analysis Assistant, https://github.com/liangdabiao/claude-data-analysis, https://image.aruoshui.fun/i/2025/01/13/p2416y-0.webp%} 

![Vibe Coding & Agentic Coding](https://image.aruoshui.fun/i/2025/10/18/yqmu25-0.webp)

# 面向Agentic Code开发模式
康奈尔大学[《Vibe Coding vs. Agentic Coding: Fundamentals and Practical Implications of Agentic AI》]( https://arxiv.org/pdf/2505.19443)的综述性论文深入探讨了人工智能辅助软件开发领域中两种新兴范式的本质区别和实际应用价值。

目前的开发模式已经逐渐开始转向：
![开发谱系图](https://image.aruoshui.fun/i/2025/10/19/3kqgc5-0.webp)

- **Vibe Coding**强调通过基于提示的、对话式的流程来实现直观的、人在回路中的人机交互，从而支持构思、实验和创造性探索；

- **Agentic Code**则通过目标驱动的 agent 实现自主软件开发，这些 agent 能够以极少的人工干预来规划、执行、测试以及迭代任务。



## Vibe Coding —— 专注于高层次的产品构想和功能迭代，将具体的代码实现完全交给AI
{% note success flat %}
直觉式编码（Vibe Coding）这个概念由AI研究者Andrej Karpathy首次提出，它描述了一种全新的软件开发模式。
{% endnote %}
![Andrej Karpathy-特斯拉到OpenAI联合创始人](https://image.aruoshui.fun/i/2025/10/18/xn1rzq-0.webp)

{% note info flat %}
- Andrej Karpathy (OpenAI联合创始人)：他将Vibe Coding描述为“完全沉浸于感觉，拥抱指数级增长，甚至忘记代码的存在”。在他看来，这意味着无条件地“全部接受”AI生成的代码，不再阅读差异（diffs），遇到错误时直接将错误信息粘贴给AI去修复。这是一种适用于快速原型或周末项目的有趣尝试。
- Steve Yegge (《Vibe Coding》联合作者)：他的定义更为直接——“AI编写代码，人类进行监督。”
- Addy Osmani (《Beyond Vibe Coding》作者)：他认为，Vibe Coding是“让强大的LLM成为编码伙伴，由它们处理繁重的代码生成工作，以便你能专注于更高层次的目标。
{% endnote %}

![Vibe Coding的两个角色](https://image.aruoshui.fun/i/2025/10/18/xwsjqk-0.webp)

- 直觉式编码重新定向了开发者的工作重点，从掌握语法和低级操作转向了**意图表达、架构设计和交互式调试**。
- 未来更有可能应用到**产品经理（后向整合）**、**架构师（前向整合）** 的工作范畴内，加速 `showcase` 甚至 `mvp` 的快速形成。
- vibe 对人员要求挺高的，基本上需要同时承担**产品+研发+研究**的工作。基于这些基础知识的理解，才能指挥好 AI 生成有一定质量的应用。
![产品指挥家](https://image.aruoshui.fun/i/2025/10/19/3hdrdq-0.webp)

### Vibe Coding的主要应用场景
Vibe Coding最理想的应用场景是快速原型开发 (Prototyping)。它的核心优势在于速度——能够在数小时内将一个想法变为可交互的原型。

{% note info flat %}
其他应用场景包括：
- 头脑风暴：快速探索疯狂的新想法，不受实现细节的束缚。
- 构建开发者工具：为自己或团队创建一些内部小工具。
{% endnote %}

### 进行Vibe Code需要具备的核心能力
![五大能力](https://image.aruoshui.fun/i/2025/10/19/3xcxov-0.webp)
- **思维（战略问题构建）** ————经过严谨的思考过程，形成一份精心编写的**《产品需求文档》（PRD）**，作为 LLM 的详细背景蓝图。
- **框架（架构意识）**————开发人员具备对**常见软件框架、库和架构模式的了解**，引导 LLM 使用恰当、可靠且符合行业标准的技术，从而缩小解决方案的范围，提高代码质量和可维护性。
- **检查点（版本控制）**————LLM生成性和有时的不可预测性，**保障快速迭代**周期
- **调试（协作式错误解决）**————人的**监督**至关重要，引导流程并验证 AI 提出的解决方案。
- **上下文（信息提供）**————**Vibe 编码的效率与提供给 LLM 的上下文的质量和全面性成正比，丰富的上下文可减少歧义，帮助 LLM 生成更准确和相关的代码**

### Vibe Coding缺陷——不适用于生产环境
![显而易见的劣势](https://image.aruoshui.fun/i/2025/10/19/43qm4a-0.webp)
- 代码**质量**隐忧
- **技术债务**及维护挑战
- **依赖**与风险：过度依赖AI生成代码可能导致对特定AI提供商的依赖

![风险评估](https://image.aruoshui.fun/i/2025/10/19/47gbgr-0.webp)


## Agentic Coding
### AI Agent 与 Agentic AI
{% note success flat %}
生成式 AI（Generative AI） 为例，比如能写文章、翻译、聊天的大型语言模型（LLMs），以及能理解和生成图像的大型图像模型（LIMs）。它们的核心能力在于「生成内容」，它们是被动反应式的。你给它一个 Prompt，它响应一次，然后就「忘记」了这次交互，等待下一个 Prompt。它们没有自己的目标，不会主动去感知环境、采取行动，更无法调用外部工具去完成现实世界的任务。它们就像一个被困在数字世界里的「理论家」，只能在自己的世界里创作，无法伸出手去触碰外部世界。

但现实世界的任务往往需要 AI 不仅能「说」或「写」，还需要能「做」。比如，帮我预定机票、查询最新的股票价格、分析一份 PDF 文档、或者控制一个机器人完成某个操作。裸的生成式 AI 本身做不到这些。

有需求就有变革。越来越多的研究者开始思考，如何让这些强大的生成模型变得更加「主动」和「有用」，能够像一个真正的「智能体」（Agent）那样，替我们去完成任务。
{% endnote %}
### 什么是 AI Agent？

AI Agent「狭义地」理解为一个由强大的生成模型驱动，并配备了各种外部工具的单体智能体。它的出现，让 AI 从一个「内容生成器」变成了一个「任务执行者」。在 LLM/LIM 这个「大脑」的基础上，增加了几个关键模块，形成一个经典的 「感知 - 推理 - 行动 - 观察」循环（Perceive-Reason-Act-Observe Loop） ：

![AI Agent架构](https://image.aruoshui.fun/i/2025/10/19/57o5zc-0.webp)

1. 客户支持聊天机器人： 理解客户问题，调用后台 API 查询订单状态，生成定制化回复。
2. 自动化邮件助手： 根据邮件内容自动分类、标记优先级、甚至起草回复。
3. 智能日程助手： 理解含糊的日程指令，检查日历，协调参与者时间，自动创建会议。
4. 基础数据报告生成： 根据自然语言查询，连接数据库，生成简单的报告或图表。

#### 单体 Agent 的固有局限
- **缺乏因果理解：** LLMs 擅长发现相关性，但不理解因果关系。这导致 Agent 在面对从未见过的情况或需要模拟干预时表现脆弱，容易犯低级错误。
- **继承 LLM 的局限性：**
    - **幻觉**： Agent 可能会一本正经地胡说八道，生成虚假信息，这在需要高准确性的应用中是致命的。
    - **推理深度不足**： 尽管有 CoT 等技术，LLM 在处理需要深层逻辑推理和复杂规划的问题时仍然可能力不从心。
    - **知识时效性**： LLM 的知识停留在训练数据截止日期，除非通过工具调用获取实时信息，否则无法处理最新情况。
    - **Prompt 脆弱性**： 微小的 Prompt 改动可能导致 Agent 行为差异巨大，难以稳定控制。
- **规划与恢复能力有限**： 单体 Agent 在执行长流程、多步骤任务时，如果中间某一步失败或出现意外，往往难以有效地检测错误、理解原因并自主恢复或调整计划。

### 什么是 Agentic AI？
Agentic AI 理解为协同作战的智能体「团队」，更接近 Multi-Agent 的概念。代表了一种范式转变，指的是由多个 AI Agent 组成的，能够相互协作、动态协调、共同追求一个高层级复杂目标的系统。

![Agentic AI架构](https://image.aruoshui.fun/i/2025/10/19/5d6nej-0.webp)

#### 核心特征
1. **多 Agent 协同（Multi-Agent Collaboration）：** 系统由多个具备不同能力或角色的 Agent 组成。例如，在一个软件开发 Agentic AI 系统中，可能有一个 Agent 负责需求分析（Product Manager Agent），一个负责架构设计（Architect Agent），一个负责编写代码（Coder Agent），一个负责测试（Tester Agent），甚至还有一个负责协调整个流程（CEO Agent）。
2. **任务动态分解与分配（Dynamic Task Decomposition and Assignment）：** 当接收到一个复杂的高层目标时，Agentic AI 系统能够将其自动分解为多个更小的、可由不同 Agent 处理的子任务，并动态地分配给合适的 Agent。
3. **Agent 间通信与协调（Inter-Agent Communication and Coordination）：** Agent 团队成员之间需要能够有效地沟通、共享信息、同步状态、协商决策。这通常通过标准化的通信协议、消息队列或共享内存来实现。
4. **编排层/元 Agent（Orchestration Layer / Meta-Agent）：** 这是 Agentic AI 系统的「大脑」或「指挥中心」。它负责管理整个 Agent 团队，监控任务进度，解决 Agent 之间的冲突，确保所有 Agent 的努力都朝着最终的高层目标前进。它可以是一个独立的 Agent，也可以是系统的一个核心组件。
5. **持久记忆（Persistent Memory）：** Agentic AI 系统通常拥有比单 Agent 更强大的记忆能力，而且这种记忆是共享的。团队成员可以访问共同的知识库（语义记忆）、任务历史（情景记忆）或向量数据库（向量记忆），确保信息一致性和上下文连续性，支持长期、多阶段的任务。


## 由Vibe Coding向Agentic Coding转变

![Agentic Coding](https://image.aruoshui.fun/i/2025/10/19/5m5e71-0.webp)

{% timeline 由Vibe Coding向Agentic Coding转变,blue %}

<!-- timeline **Vibe Coding 是以人为本的控制与反应式反馈：** -->
Vibe 编码架构本质上是基于反应式模型运行的，在这种模型中，人类开发者始终是唯一负责验证、错误检测和迭代改进的主体。
<!-- endtimeline -->

<!-- timeline **Agentic Coding 以反馈驱动的自主性为核心架构原则：** -->

通过包括规划、执行、测试、评估和纠正迭代在内的多级反馈循环运行，所有这些步骤之间无需人工提示即可协调进行。闭环反馈能够在重复性和确定性的编程环境中实现高保真度，例如依赖项管理、持续集成/持续部署（CI/CD）配置或为大规模系统自动生成测试套件。

<!-- endtimeline -->

{% endtimeline %}

## 实际工作流程

{% timeline **开发者角色与思维模型**,blue %}

<!-- timeline **Vibe Coding 对话式创作和探索性交互** -->
- **意图架构师：** 以自然语言制定项目目标，通过提示迭代来完善意图。
- **创意总监：** 评估、编辑和筛选人工智能生成的输出内容，使其与设计意图和用户体验保持一致。
- **探索者：** 利用人工智能在对未知 API 进行试验、测试用户界面模式或在几乎不了解相关知识的情况下搭建新功能时发挥效用。
<!-- endtimeline -->

<!-- timeline **Agentic Coding 以反馈驱动的自主性为核心架构原则：** -->
- **战略规划师：** 为代理指明任务、目标以及架构限制，使其依此行动。
- **监督者：** 监控执行跟踪日志、性能报告和系统输出。
- **审核人员：** 在集成之前，验证代理生成的更改的正确性、可维护性和安全性。

<!-- endtimeline -->

{% endtimeline %}


{% timeline **工作流模式**,blue %}

<!-- timeline **Vibe 编码工作流本质上是探索性的且非线性的** -->
这种模式对于界面原型设计、低风险实验或知识发现而言是最优的。

示例 - 仪表板原型设计

1. 开发人员：“构建一个 React 仪表板，包含用户数量、收入和流失率图表。”
2. AI：使用 Chart.js 和虚拟数据生成用户界面。
3. 开发人员：“添加工具提示并支持导出为 CSV 格式。”
4. AI：添加了悬停逻辑和导出按钮。
5. 开发人员：“编写 Cypress 测试。”
6. AI：输出端到端测试覆盖率。
   

<!-- endtimeline -->
<!-- timeline **Agentic Coding遵循基于任务规划、状态管理以及递归反馈循环、的结构化工作流程。** -->

示例 - 自动化依赖项升级

开发人员：“将所有 npm 包升级到最新安全版本。”
1. Agent：
   - 解析 package.json 文件
   - 更新依赖项版本
   - 执行测试套件
   - 解决兼容性问题
   - 生成变更日志
2. 开发人员：查看日志并批准拉取请求。

<!-- endtimeline -->
{% endtimeline %}



{% timeline **使用场景**,blue %}

<!-- timeline **Vibe 编码工作流本质上是探索性的且非线性的** -->
1. **创意探索**
2. **快速原型开发**：Vibe 编码在根据自然语言提示构建功能性的最小可行产品（MVP）方面表现出色。
3. **学习新技术**：无论是入职培训还是技能提升，vibe 工具都能充当实时的编程导师。
   ![Vibe Coding 的使用场景](https://image.aruoshui.fun/i/2025/10/19/e6usxy-0.webp)
   

<!-- endtimeline -->
<!-- timeline **Agentic Coding遵循基于任务规划、状态管理以及递归反馈循环、的结构化工作流程。** -->
1. **代码库重构**：代理工具能够自主分析遗留系统，检测过时的代码和架构，从而优化系统性能和可维护性。

2. **常规工程任务**：这些任务包括自动化依赖项升级、代码格式化、测试重新生成以及持续集成/持续部署（CI/CD）流水线维护。代理在整个大型代码库中应用一致的标准，从而提高可维护性并减少人工工程开销。

3. **回归错误修复**：代理系统擅长基于日志的错误诊断、根本原因分析以及自主代码修复。它们通过运行测试套件、应用补丁和更新变更日志来缩短平均修复时间（MTTR），无需开发人员干预，非常适合关键任务服务。

![Agentic Coding的使用场景](https://image.aruoshui.fun/i/2025/10/19/e7gkn3-0.webp)

<!-- endtimeline -->
{% endtimeline %}

## 五大技术底座

| 模块 | 代表技术 | 作用 |
| ---- | -------- | ---- |
|  1. **大模型**    |   GPT-5、Claude Sonnet4.5 、Gemini 2.5 Pro 全部支持 Tool-Use       |   推理+代码生成   |
| 2. **Prompt 工程**     |   ReAct、Chain-of-Thought、Scratchpad 让模型“边想边干”       |  多步规划、反思    |
| 3. **工具链**     |  编译器、调试器、测试框架、LSP、git 一个都不能少        | 闭环验证     |
| 4. **记忆&上下文**     |  向量库、滑动窗口、RAG、KV-Cache 压缩        | 跨轮次上下文     |
| 5. **MCP**     | 上下文 Schema 标准、会话状态序列化、工具调用元数据封装、跨服务上下文同步机制         | 统一模型在不同工具、记忆模块和执行环境间传递上下文的格式与语义，确保多组件协同的一致性与可扩展性     |

---
# 使用Claude Code Sub-Agents实现自动化数据分析工厂

## 五个智能体角色
![image](https://image.aruoshui.fun/i/2025/10/19/h5pqgb.webp)

## 自定义斜杠命令


## 分步骤完成数据分析
1. 先让AI进行探索性**数据分析**：`exploratory`
在claude code中执行 ： `/analyze sample.csv exploratory`
我们这里第一步是`/analyze`，意思就是先分析看看，考察一下情况。这个是利用了claude code的command技术。
**AI自动选择合适的agent去处理工作：**
![Agent自动调用](https://linux.do/uploads/default/original/4X/1/a/c/1acdfba4507380e5f3bd1a172aaceab62af93ad9.png)
![分析效果](https://linux.do/uploads/default/original/4X/4/0/6/406c5d21dff4d5930fae0b57877fdb9309db67c3.png)

2. 下一步我们试试**可视化**：
在claude code中执行 ：`/visualize your_data.csv distribution`
`/visualize` 就是可视化命令， `distribution`就是可视化数据分布，这里你可以按你要求的可视化内容，AI会理解的。例如：` /visualize your_data.csv `趋势
![过程](https://linux.do/uploads/default/original/4X/0/7/b/07bc72c5bcdfb6126779b6020df2b3a2c488e06b.png)
![可视化效果](https://linux.do/uploads/default/original/4X/4/8/d/48dc0ee5ca1a67532590ff17420605777b68b10d.png)
只需要一行命令，就可以得到这么多专业数据分析可视化成果。同时生成代码，可以检查和改变。

3. 下一步让AI生成**数据分析详细报告**
在claude code中执行 ：`/report your_data.csv`
![分析报告](https://linux.do/uploads/default/original/4X/9/a/2/9a21a5177ab3bd61bcac589c977866033fb947ca.png)

4. 还有其他更深入的功能：
   1. `/generate [language] [type]: Generate code `，这个是生成编程语言版本的深度数据分析。
   2. `/hypothesis [dataset] [domain]: Generate hypotheses` ，这个是预测类型的数据分析，例如分类，趋势，关联，预测建模后，还会自动进行假设检验，确定模型准确性。

## 一条龙全自动化数据分析
1. 执行命令 `/do-all`——调用5个agent分工合作，专业的数据分析师，**一条龙解决数据分析**的步骤和分析。
![调用过程](https://linux.do/uploads/default/optimized/4X/4/f/9/4f9a0baab8f97ab2dd0c5a740dbd7abcb34b7321_2_1134x1000.png)

2. claude code 工作一小时——干到完成为止
![分工合作过程](https://linux.do/uploads/default/optimized/4X/d/7/3/d73b5d03d4ffe98a0286990c5f4de0fb5d68da8f_2_1060x1000.png)

3. 得到数据分析结果
![分析结果](https://linux.do/uploads/default/optimized/4X/0/f/4/0f4a07ce6849984d146df749a36f1b5f1f4783ff_2_1086x998.png)
![总结](https://linux.do/uploads/default/original/4X/4/a/a/4aa6f7b2483c4052f6d778e4bc1401d2908e1753.png)
![生成的代码文件](https://linux.do/uploads/default/optimized/4X/e/6/5/e657655085cd3f52a1cb78dc61de3aa03d270515_2_358x998.png)
![可视化效果](https://linux.do/uploads/default/optimized/4X/c/7/3/c73bb6ea0859657f5d07c6f1e88c16833ddcaca4_2_1380x868.png)

## 一个开发模式

---
# 《How Anthropic teams use Claude Code》
{% note success flat %}
黄仁勋特别提到：Nvidia的软件工程师和芯片设计师 100%使用Cursor 来辅助日常工作。他表示：“现在我们的所有工程师都有AI助手，工作效率大幅提升。”
{% endnote %}

2025-06-06，Anthropic官方公开了一份手册，揭秘他们内部10个不同团队（涵盖技术、科研、产品、营销、法律等团队）是怎么使用Claude Code的，场景案例非常丰富，其中的大部分实践经验也可以迁移使用在Cursor、Cline等AI编程工具上。

![《How Anthropic teams use Claude Code》](https://image.aruoshui.fun/i/2025/10/19/gsjij1-0.webp)

## 不同团队的实践做法

### 技术和工程团队 (数据基础设施、产品开发、安全、推理、RL工程)
利用 Claude Code 来加速现有的软件开发生命周期。
**代码库理解与上手：** 新员工或需要在不熟悉的代码库中工作的工程师，会请求 Claude Code 解释系统架构、识别关键文件和依赖关系，从而显著缩短上手时间。
**功能开发与原型设计：**
- **监督式开发：** 在开发核心业务逻辑时，工程师会提供详细指令，实时监督 Claude Code 编写代码，确保质量和合规性。
- **自主式开发：** 对于非核心功能（如实现Vim模式）或快速原型，工程师会启用“自动接受模式”，让 Claude Code 自主编写、测试和迭代，自己仅做最终的审查和微调。
**自动化测试：** 工程师在完成功能开发后，会让 Claude Code 编写全面的单元测试，并自动覆盖容易忽略的边界情况。
**代码审查与重构：** 用于审查基础设施代码（如Terraform计划）以评估风险，或处理那些对于编辑器宏来说太复杂、但又不足以投入大量开发精力的重构任务。
**调试与事故响应：** 当遇到复杂问题时（如 Kubernetes 集群故障），团队会向 Claude Code 提供堆栈跟踪、错误日志甚至UI截图，让其引导找到问题根源并提供修复指令。安全工程团队通过这种方式，将原本需要10-15分钟的基础设施调试时间缩短到了5分钟左右。

### 数据科学与可视化团队
需要精密的可视化工具来解析模型性能，但构建这类工具通常要求掌握陌生语言和框架的专业技能。Claude Code使这支团队无需成为全栈开发人员，即可构建生产级质量的分析仪表板。
- **构建专业级应用：** 即便团队成员对 JavaScript/TypeScript “知之甚少”，他们依然成功利用 Claude Code 构建了数千行代码的 React 应用，用于模型性能的可视化分析。
- **从一次性到持久化：** 他们不再依赖用完即弃的 Jupyter notebook，转而创建可复用的永久性 React 仪表板，用于未来模型的持续评估。

### 非技术团队 (产品设计、增长营销、法律)

这类团队的用法体现了 Claude Code “赋能”的核心价值，让他们能够独立完成过去必须依赖工程师才能实现的任务。
**产品设计直接上手：** 设计师不再仅仅交付静态模型，而是直接使用 Claude Code 进行前端的视觉微调（如字体、颜色），甚至进行复杂的状态管理更改。他们还可以通过粘贴模型图的方式快速生成可交互的原型。

**营销自动化：**
  - 广告创意生成：增长营销团队创建了自动化流程，可分析现有广告数据，并大规模生成符合严格字符限制的新广告文案，将数小时的工作缩短至几分钟。
  - 设计素材批量生产：开发 Figma 插件，通过编程方式一键生成上百个广告图片变体，将创意产出提升10倍。

**法律工具原型化：** 法律团队成员为有语言障碍的家人构建了定制化的辅助沟通应用，并为部门设计了“电话树”系统原型，以帮助同事快速找到合适的律师。
