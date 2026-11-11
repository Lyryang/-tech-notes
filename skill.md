## engineering（工程类技能）

**用户手动调用（输入命令触发）**

- **`/ask-matt`**：不知道该用哪个技能时问它，它会帮你路由到合适的技能/流程。
- **`/grill-with-docs`**：与你进行"拷问式"访谈，边聊边梳理项目术语并更新 `CONTEXT.md` 和 ADR 文档，适合需求对齐+建立领域语言。
- **`/triage`**：把 issue 按状态机流程逐步分诊、推进处理。
- **`/improve-codebase-architecture`**：扫描代码库找出可"加深模块"的重构机会，生成可视化 HTML 报告，再和你逐一拷问确认。
- **`/setup-matt-pocock-skills`**：每个仓库首次使用前运行一次，配置 issue 追踪器、分诊标签、文档存放路径。
- **`/to-spec`**：把当前对话内容直接整理成规格文档并发布到 issue 系统，无需再访谈。
- **`/to-tickets`**：把方案/规格/对话拆解成一组"tracer-bullet"式任务票，并标注彼此的阻塞关系。
- **`/implement`**：按规格或任务票实现功能，在约定节点调用 `/tdd`，完成后走 `/code-review`。
- **`/wayfinder`**：为超出单次会话容量的大型工作，在 issue 追踪器上建立决策票地图并逐步推进解决。

**模型自动调用（AI 根据情境自行触发）**

- **`prototype`**：为验证设计问题快速搭建一次性原型（单个 HTML 文件或若干组件）。
- **`diagnosing-bugs`**：针对疑难 bug/性能回归的严谨诊断流程：先复现问题，再逐步收窄定位。
- **`research`**：针对某个问题，基于高可信一手资料做调研，产出带引用的 Markdown 文档（可后台运行）。
- **`tdd`**：红-绿-重构的测试驱动开发循环，逐个垂直切片实现功能/修复。
- **`domain-modeling`**：主动构建并打磨项目领域模型，用词汇表和边缘场景反复推敲术语。
- **`codebase-design`**：设计"深模块"（小接口、大功能）的共享准则和词汇。
- **`code-review`**：从"规范"和"代码异味"两个维度审查自某基准点以来的 diff。
- **`resolving-merge-conflicts`**：逐个冲突块处理进行中的 git merge/rebase 冲突，按意图溯源解决。
- **`wizard`**：生成交互式 bash 向导，引导人工完成只有人才能做的步骤（如开通基础设施、设置凭证）。

## Productivity（通用效率类，非代码专属）

**用户手动调用**

- **`/grill-me`**：对一个计划/设计进行持续拷问式访谈，直到方案的每个分支都想清楚。
- **`/handoff`**：把当前对话压缩成交接文档，方便另一个 agent 接手继续工作。
- **`/teach`**：跨多次会话教你一个新技能或概念，把当前目录当作有状态的学习工作区。
- **`/to-questionnaire`**：把自己回答不了的决策问题整理成 Markdown 问卷，交给能回答的人异步或当面填写。
- **`/wait-what`**：当某条消息没听懂时立刻触发，让 agent 用你 `CONTEXT.md` 里的术语重新解释一遍。

**模型自动调用**

- **`grilling`**：可复用的访谈引擎，对计划/决策/想法进行持续追问直到每个分支都清楚（被 `/grill-me` 等技能复用）。
- **`writing-for-agents`**：指导如何为 agent 编写文档（技能说明、AGENTS.md/CLAUDE.md 等）。

------

安装方式：

在项目根目录下执行命令： `npx skills@latest add mattpocock/skills`，之后选择即可