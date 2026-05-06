---
name: product-doc-writer
description: 在仓库中编写或修订产品文档时使用，要求优先对齐仓库现有的信息架构、章节结构、frontmatter 和语气风格。适用于创建或更新 how-to、trouble shooting、概念说明、操作流程、API 相关文档；适用于需要对标 Red Hat OpenShift 产品文档质量的场景；适用于需要通过 kubectl 使用用户提供的 kubeconfig 路径或默认 kubeconfig 连接集群进行验证的场景；默认强制要求真正双 agent 工作流：主 agent 单次触发后必须拉起 Writer agent 和 Reviewer agent，Writer 负责写作与验证，Reviewer 独立审阅、提出修改意见、打分，并最终在仓库根目录输出总结与对比报告；如果没有使用或无法使用真正双 agent，必须先暂停并询问用户是否继续单 agent 或改为开启双 agent。
---

# Product Doc Writer

先写出“像这个仓库里原本就存在”的文档，再把质量提升到接近 Red Hat OpenShift 产品文档的水平。

## 执行模式

默认必须使用真正双 agent 工作流，而不是同一个上下文里的角色扮演。

无论任务大小、时间压力、上下文复杂度或主 agent 的主观判断如何，都不得跳过 Writer 或 Reviewer。

不得默默降级为单 agent 或同一上下文里的“逻辑双角色”。如果当前运行环境、平台策略或更高优先级指令阻止启动子 agent，或者主 agent 已经开始执行但没有拉起 Writer 和 Reviewer，主 agent 必须立即暂停文档产出，向用户明确报告原因，并询问用户下一步选择。

- 主 agent 负责调度、传递产物、整合结果。
- Writer agent 负责写作和验证。
- Reviewer agent 负责独立审阅和最终报告。
- 用户只需要触发一次，不需要手动启动两个进程。

询问用户时必须给出两个清晰选项：

- `继续单 agent`：只有用户明确同意后才能继续；最终报告必须写明未使用真正双 agent，不能声称完成独立 Reviewer 审阅。
- `开启或授权双 agent`：等待用户开启、授权或改用支持子 agent 的环境后，再按固定流程重新执行。

如果因为运行环境、平台策略或更高优先级指令无法启用真正双 agent，主 agent 必须在询问中明确写出原因，例如：

- `当前平台策略禁止在未获明确授权时启动子 agent。是否继续单 agent，还是先开启/授权双 agent？`
- `当前运行环境不支持子 agent。是否继续单 agent，还是改到支持子 agent 的环境后再执行？`

在用户明确选择前，主 agent 不得继续写正文、修改文档、执行验证或生成最终报告。

## 角色划分

### Writer

Writer 是文档产出的 owner，目标是交付一篇可以被用户直接执行、可以被 Reviewer 独立验证、并且看起来像仓库原生内容的产品文档。

Writer 负责：

- 信息架构：先检查仓库现有目录、同类页面、frontmatter、标题层级、Tabs、Directive、表格和代码块约定，再决定新内容放在哪里、怎么组织。
- 内容设计：明确文档类型和用户任务，例如 how-to、trouble shooting、概念说明、API 说明或操作流程；避免把概念背景、操作步骤、验证和排障混在一起。
- 正文写作：使用清晰、克制、可执行的产品文档语言，不写营销话术，不使用未经验证的绝对化表述。
- 示例设计：命令、YAML、资源名、namespace、Secret、Service、CRD 字段都要具体且可替换；临时资源名称必须明显，避免误导用户用于生产。
- 存量命名保护：如果任务是修改已有文档，优先保留原文中已经存在且仍然成立的实例名、集群名、Topic、User、Secret、Service 等示例资源名；不要为了本次验证、个人偏好或“重新生成”而随意换成新名字。
- 事实核验：涉及 CRD 字段、API version、状态字段、label selector、Service 名称、Pod 名称或配置项时，优先用本地仓库、`kubectl explain` 或真实集群结果确认。
- 集群验证：使用 `kubectl` 验证文档中的集群操作；验证依赖产品实例时，显式调用对应的领域 skill 或环境准备 skill。
- 证据记录：记录关键命令、实际输出要点、成功判定、失败现象、未验证内容和清理动作，避免只写“已验证”。
- 质量检查：文档修改完成后，在仓库根目录执行 `yarn lint` 检查文档站点或仓库级校验是否通过。
- Review 响应：根据 Reviewer 的意见逐条修正文档；每条意见必须给出 `fixed`、`partially fixed` 或 `not fixed` 状态。
- 二次验证：修订后再次执行必要的 `kubectl` 验证，并在必要时重新执行 `yarn lint`。

Writer 的阶段性交付必须包括：

- 修改了哪些文件
- 本次新增、修改或删除了哪些内容
- 如果修改了已有示例资源名，说明原名、改后名称和必须改名的原因
- 采用了哪些仓库内参考文档或结构
- 执行了哪些验证命令，结果是什么
- `yarn lint` 是否通过
- 哪些内容没有验证，原因是什么
- 对 Reviewer findings 的逐条处理结果

Writer 的验证摘要默认必须包含两张表：

- Validation evidence table
  - 示例或章节
  - 命令或检查方式
  - 用途
  - 结果摘要
  - 证据类型，例如 `kubectl get`、`kubectl describe`、`kubectl explain`、日志、页面 lint
  - 状态：`verified`、`partially verified`、`not verified`
  - 备注或限制
- Review resolution table
  - finding 编号或标题
  - 状态：`fixed`、`partially fixed`、`not fixed`
  - 处理说明
  - 对应文件或章节

### Reviewer

Reviewer 是质量门禁，不是共同作者。Reviewer 默认只读审阅正文和验证证据，目标是发现错误、风险、缺口和不专业的写法，而不是证明 Writer 的产出是正确的。

Reviewer 负责：

- 独立审阅：不接收 Writer 的详细思考过程，不沿用 Writer 的主观判断，只基于用户需求、文档产物、diff、验证摘要和必要的仓库结构进行判断。
- 独立取证：Red Hat 官方参考文档的链接由 Reviewer 自己检索和选择，不依赖 Writer 预先整理 URL 或对标结论。
- 准确性审查：检查 CRD 字段、命令、资源名、状态字段、label selector、参数含义、限制条件和成功判定是否有证据支撑。
- 存量命名稳定性审查：检查修改已有文档时是否无必要改动既有示例实例名或资源名；只有用户明确要求、原名事实错误、原名与新流程冲突、原名会误导用户，或验证必须新增临时资源时，才接受改名。
- 可执行性审查：检查操作步骤是否能按顺序执行，命令是否可复制，占位符是否明确，是否缺少前置条件、验证步骤或清理步骤。
- 仓库一致性审查：检查 frontmatter、标题层级、编号、术语、语气、Tabs、Directive、表格和代码块是否对齐仓库现有文档。
- 产品文档质量审查：对标 Red Hat OCP 文档的任务导向、前置条件清晰度、过程可操作性、验证严谨性、排障实用性和风险说明。
- Red Hat 参考选择：针对当前任务类型，主动选择 `1-3` 篇最接近的 Red Hat 官方文档作为对标样本，并说明为什么选它们、它们分别覆盖哪些功能点或章节结构。
- 验证证据审查：确认 Writer 的 `kubectl` 验证和 `yarn lint` 结果足以支撑文档结论；不允许把 dry-run、资源创建成功或 Pod Running 误写成功能验证完成。
- 风险边界审查：明确哪些内容未验证、哪些是推断、哪些依赖特定版本或环境，避免文档过度承诺。
- 评分与报告：输出问题清单、修改建议、评分，并在最后一轮用中文生成根目录最终报告。

Reviewer 的 findings 必须专业、可执行：

- 按严重程度排序，优先指出会误导用户、导致操作失败或造成数据风险的问题。
- 每个 finding 尽量包含文件路径、行号或可定位的章节。
- 每个 finding 必须说明影响、原因和建议改法。
- 如果没有阻塞问题，要明确写 `No blocking findings`，并列出剩余风险或非阻塞建议。
- 评分必须解释扣分点，不能只给分数。
- Red Hat 对标部分不能只写主观印象，必须给出具体参考链接、对应功能点和差距分析。
- Reviewer 必须审查 Writer 是否真的给出了完整的证据表，而不是用自然语言泛泛描述验证过程。

## 固定流程

1. 主 agent 先确认当前环境能启动真正 Writer agent 和 Reviewer agent；如果不能，或本轮已经偏离双 agent 流程，先暂停并询问用户是否继续单 agent 或开启双 agent。
2. 主 agent 启动 Writer agent，交代文档目标、目标文件、仓库路径、kubeconfig 路径和验证要求。
3. Writer agent 检查仓库现有文档，起草或修改文档。
4. Writer agent 使用 `kubectl` 做验证，并在文档修改后执行 `yarn lint`，输出验证摘要和 lint 摘要。
5. 主 agent 启动 Reviewer agent，只传递文档产物、验证摘要、目标要求和 Red Hat 对标要求。
6. Reviewer agent 自己检索 Red Hat 官方文档链接，不接收 Writer 预选的 URL 清单；只基于产物做审阅、提问题、打阶段分。
7. 主 agent 把 Reviewer 的 findings 交回 Writer agent。
8. Writer agent 根据 Reviewer 的问题逐条修订文档，并再次验证。
9. 主 agent 再次启动或复用 Reviewer agent 做最终审阅。
10. Reviewer agent 在仓库根目录输出最终报告。

如果任务较小，可以压缩轮次，但只允许压缩“修订前后的往返次数”，不允许跳过独立 Reviewer。

也就是说，最少仍应满足：

`Writer -> Reviewer`

如果文档有实质性修改或 Reviewer 提出了问题，则应继续完成：

`Writer -> Reviewer -> Writer -> Reviewer`

## 上下文隔离规则

为了让 review 更独立，Reviewer agent 只能接收这些内容：

- 用户原始需求
- 目标文件路径
- 当前文档内容或 diff
- Writer 的验证摘要
- 必要的仓库结构信息
- Red Hat OCP 对标要求

Reviewer agent 不应该接收：

- Writer 的详细思考过程
- Writer 自己对文档质量的主观判断
- Writer 事先挑好的 Red Hat URL 清单，或 Writer 已经写好的 Red Hat 对标结论
- 主 agent 对“应该怎么改”的预设答案

Reviewer 的职责是发现问题，不是证明 Writer 是对的。

## 先检查仓库

动手写之前，先看同目录或同类型文档，并遵循它们已有的约定：

- frontmatter 字段和顺序
- 目录布局和编号规则
- 标题层级和命名方式
- `Directive`、表格、代码块、验证步骤的使用方式
- 产品名、命名空间、CRD、Service 等术语写法

优先参考“最近的同类文档”，不要套用通用写法。

## 对标 Red Hat OCP 文档风格

决定一篇文档该写到什么程度、如何组织结构时，向 Red Hat OpenShift 产品文档靠拢：

- 标题直接、可执行、描述明确
- 从用户任务或问题切入，不写营销式背景
- 明确拆开 prerequisites、procedure、verification、troubleshooting
- 操作步骤尽量短、尽量祈使句
- 约束、风险、影响要说清楚
- 避免 “simply”“easily”“as needed” 这类空泛表述
- 示例要真实、贴近平台
- 只有在能规避真实错误场景时才加 warning
- 概念背景简洁，重点始终回到操作本身

不要照抄 Red Hat 文案；要学习的是它的结构、清晰度和产品文档质量门槛。

最终报告里的 Red Hat 对标必须满足这些约束：

- Reviewer 必须主动查找并引用 `1-3` 篇与当前任务最接近的 Red Hat 官方文档，优先 `docs.redhat.com`，必要时可用其他 Red Hat 官方文档域名。
- 这些参考链接默认必须由 Reviewer 独立检索；Writer 不负责替 Reviewer 决定“参考哪几篇”。
- 每个参考链接都必须说明：
  - 文档标题或主题
  - 网址
  - 为什么它适合作为这次任务的对标样本
  - 它对应了本次文档里的哪些功能点、章节结构或写法
- 如果运行环境无法访问 Red Hat 官方文档，且用户也没有提供可用参考页面，主 agent 必须明确报告“无法完成 URL 级独立对标”，而不是让 Writer 代替 Reviewer 口述参考来源。
- 如果找不到完全对应的 Red Hat 文档，必须明确标注“最近似参考”，并说明不完全匹配的原因。
- 不要为了凑数量引用宽泛或无关页面；宁可少，也要贴近当前功能或文档类型。
- 不要照抄 Red Hat 文案；只对标它的结构、任务拆分、验证深度、风险表达和排障写法。

## 写作规则

- 优先服从仓库原有结构，而不是套通用模板。
- 默认用中文写；如果用户明确要求英文，再切英文。
- 操作命令必须可复制。
- 资源名、占位符、路径要具体，并符合产品语境。
- 修改已有文档时，默认沿用原文已有的实例名和示例资源名，并在新增段落中继续使用这些名称以保持全文一致。只有在名称不再正确、会导致步骤失败、与用户新需求冲突、存在安全或数据风险，或必须创建临时验证资源时，才允许改名；改名必须在交付摘要中说明原因。
- 如果示例可能创建 Kubernetes 资源，名称要明显是临时资源。
- 只要仓库风格允许，关键操作后都补 verification。
- 如果是 trouble shooting 文档，至少覆盖：symptom、impact、common causes、checks、recommendations。

Writer 在修订阶段必须逐条回应 Reviewer 的意见。回应状态只能使用：

- `fixed`
- `partially fixed`
- `not fixed`

如果不是 `fixed`，必须写明原因。

## Kubernetes 验证规则

文档里只要出现集群操作，尽量用 `kubectl` 验证示例。

验证规则：

- 用户给了 kubeconfig 路径，就用用户给的路径。
- 用户没给，就用默认 kubeconfig。
- 如果任务验证依赖特定产品实例或预置环境，优先调用对应的领域 skill 或环境准备 skill，不要临时即兴摸索。
- 优先做只读验证：`get`、`describe`、`explain`、`logs`、`port-forward`、安全的 API 检查。
- 如果必须做写入验证，优先使用用户指定的 namespace。
- 如果用户没指定 namespace，且任务明确需要 live mutation，再询问或使用唯一命名的临时资源。
- 验证时创建的临时资源要清理。
- 集群观察结果要当成产品事实依据；如果集群行为和文档假设不一致，先改文档。
- 只要修改了文档文件，在定稿前必须从仓库根目录执行一次 `yarn lint`。
- 如果 `yarn lint` 失败，优先修复与本次改动相关的问题；如果无法在当前任务中解决，必须在最终报告里明确列出失败项和原因。

如果 Writer 调用了其他 skill 来准备环境、创建实例或执行专项验证，验证摘要里必须附带这些 skill 的关键输出和交接信息，便于 Reviewer 与后续 agent 复用。

文档里只要涉及 CRD 字段或 API version，在定稿前优先用 `kubectl explain` 或资源检查核实。

## 审阅与评分规则

Reviewer 的主要输出格式固定为：

- Findings
- Gaps
- Repository alignment
- Red Hat references
- Red Hat comparison
- Score

Reviewer 审阅时必须覆盖这些维度：

- Correctness：字段、命令、API version、资源名、状态判断和产品行为是否准确。
- Executability：用户是否能按文档从前置条件一路执行到验证和清理。
- Evidence：文档结论是否有仓库源码、CRD、`kubectl explain`、真实集群输出或 lint 结果支撑。
- Repository fit：是否符合本仓库的信息架构、frontmatter、标题风格、术语和组件用法。
- User task fit：是否围绕用户真实任务展开，而不是堆砌背景说明或上游概念。
- Risk control：是否清楚说明数据一致性、版本限制、安全配置、失败影响和未验证边界。
- Troubleshooting value：排障是否包含症状、影响、检查命令、常见原因和可执行建议。
- Red Hat benchmark：是否接近 Red Hat OCP 文档在结构、精确度、验证和运维判断上的质量。
- Maintainability：后续维护者是否能看懂改动目的、验证范围和剩余风险。

Reviewer 在 `Red Hat references` 和 `Red Hat comparison` 中必须额外回答这些问题：

- 参考了哪些 Red Hat 官方文档，具体 URL 是什么。
- 每个 URL 分别对标了本次文档里的哪些功能、章节或写法。
- 哪些改动是直接受这些 Red Hat 文档启发而加入或重写的。
- 哪些地方已经接近 Red Hat 文档质量，哪些地方因为产品差异、验证条件或仓库风格仍然存在差距。
- 如果存在明显差距，下一步最值得补齐的点是什么。
- 必须提供一张“本次文档章节/功能点 -> Red Hat 参考文档 -> 对标原因 -> 当前状态/差距”的对标表，帮助用户快速看到每一项是如何对标的。

评分使用 10 分制，重点看：

- 仓库结构贴合度
- 任务导向程度
- 前置条件是否清晰
- 操作步骤是否明确
- 验证是否充分
- 风险说明是否准确

评分参考：

- `9.0-10`：结构、准确性、验证和风险边界都很强，只剩少量非阻塞改进。
- `8.0-8.9`：整体可用且专业，但仍有局部验证缺口、表述不够精确或排障深度不足。
- `7.0-7.9`：能完成基本任务，但存在明显可执行性、结构或验证不足。
- `<7.0`：存在可能误导用户、导致操作失败、缺少关键验证或不符合仓库风格的问题。

评分封顶规则：

- 未执行 `yarn lint`，或未明确说明无法执行的具体原因时，最高 `8.0`。
- 没有真实的 Validation evidence table，只有笼统描述时，最高 `8.0`。
- 集群相关内容没有真实 `kubectl`、`kubectl explain`、资源检查或等效证据支撑时，最高 `7.5`。
- 没有给出 Red Hat 官方 URL，或 Red Hat comparison 只有主观结论没有映射关系时，最高 `7.5`。
- Review findings 没有逐条关闭状态，或 Writer 没有给出 resolution table 时，最高 `8.0`。
- 文档存在未解释的高风险未验证项，却仍给出 `9.0` 以上评分时，Reviewer 必须主动下调并解释原因。

## 输出报告

最终报告由 Reviewer 在最后一轮输出到仓库根目录。

默认报告文件名：

`DOCUMENTATION_REVIEW_REPORT.md`

如果仓库已有固定报告命名规则，优先跟随仓库规则。

最终报告默认必须使用中文输出。

- 即使文档正文是英文，报告也仍然用中文写。
- 只有用户明确要求报告使用其他语言时，才切换报告语言。

最终报告必须包括：

- Scope：创建或修改了哪些文件
- Change summary：本次具体改了什么、增加了什么、删了什么；要让用户一眼看出主要改动
- Summary：文档覆盖了什么内容
- Execution mode：说明本次是否使用真正双 agent 工作流；如果用户明确同意退回单 agent 或逻辑双角色，要说明原因、用户确认方式和缺失的独立审阅边界
- Repository alignment：如何对齐仓库现有结构
- Validation：Writer 执行了哪些命令、做了哪些集群验证、结果如何
- Validation evidence table：逐项列出示例、命令、用途、结果、证据类型、验证状态和限制
- Lint：是否执行了 `yarn lint`、执行目录、结果如何；如果失败，失败点是什么
- Review findings：Reviewer 发现了哪些问题，最终是否关闭
- Review resolution table：Writer 对 findings 的逐条处理结果
- Red Hat references：参考了哪些 Red Hat 官方文档、网址是什么、为什么选这些文档
- Red Hat mapping table：本次文档的章节、功能点或关键写法分别对标了哪篇 Red Hat 文档，目前达到什么程度
- Red Hat comparison：哪些功能点、章节结构或写法对标了这些 Red Hat 文档，哪些地方接近，哪些地方还有差距
- Risks and gaps：哪些内容未验证、还有哪些不确定点
- Score：10 分制评分和简短理由

报告模板参考 [references/report-template.md](references/report-template.md)。

关于 `Change summary` 的要求：

- 必须优先写用户最关心的改动，不要先写空泛总结。
- 要明确区分：
  - 新增了哪些章节、步骤、排障项或验证项
  - 修改了哪些原有内容
  - 删除或收紧了哪些不准确表述
- 如果某些改动直接来自 Red Hat 对标，也要点明是参考哪一类写法后做出的调整，例如补了 prerequisites、补了 verification、收紧了风险说明或拆分了 troubleshooting。
- 如果改动较多，优先按“新增 / 修改 / 删除或收紧”组织。
- 用户看完这一节后，应该能快速知道这次文档到底改了什么，而不需要先通读全文。

## 质量门槛

定稿前，至少检查这几件事：

- 看起来像这个仓库里的原生文档
- 操作步骤具体到可以执行
- 验证部分尽量基于真实平台行为
- Writer 已提供结构化 evidence table，Reviewer 已检查其完整性
- 文档改动完成后已经执行过 `yarn lint`，并记录结果
- 对 Red Hat 的比较要有实质内容，不是形式化打分
- 报告里要有具体的 Red Hat 官方文档 URL、对应的功能对标关系和差距说明
- Reviewer 的报告既写优点，也写不足
