# 报告模板

在仓库根目录输出最终报告时，优先使用下面这个结构。默认由 Reviewer 生成。

默认要求：

- 报告使用中文输出。
- 即使文档正文是英文，报告也仍然使用中文。
- 报告开头要先让用户快速看懂“这次到底改了什么”。

## 标题

使用 `# 文档审阅报告`。

## 章节结构

### Scope

- 列出创建或修改的文档文件。

### Change Summary

- 先总结本次主要改动，要求用户一眼就能知道：
  - 新增了什么
  - 修改了什么
  - 删除了什么，或收紧了哪些原先不准确的表述
- 如果是基于 review 或验证做的修订，要明确哪些改动来自真实验证结果，哪些来自 review 发现。
- 这一节不要写空泛总结，要尽量写成“改动清单的高质量摘要”。

### Summary

- 概述文档覆盖内容。
- 说明目标读者或使用场景。

### Execution Mode

- 说明本次是否使用真正双 agent 工作流。
- 如果使用了双 agent，说明 Writer 和 Reviewer 的职责分工。
- 如果未使用双 agent，说明退回逻辑双角色的原因。

### Repository Alignment

- 说明参考了哪些本地文档结构。
- 说明最终输出如何对齐仓库里的编号、标题、frontmatter 和写作风格。
- 如果借鉴了仓库内某篇文档的章节拆分或组件用法，写明对应文件路径和借鉴点。

### Validation

- 写明使用的 kubeconfig 路径；如果用的是默认 kubeconfig，也要说明。
- 写明验证使用的 namespace。
- 列出 Writer 执行过的命令。
- 对每条关键命令补一句用途和结果，不要只堆命令。
- 说明哪些示例已完整验证、哪些只做了部分验证、哪些未验证。

### Validation Evidence Table

- 必须提供一张结构化证据表。
- 推荐列至少包括：
  - 示例或章节
  - 命令或检查方式
  - 用途
  - 结果摘要
  - 证据类型
  - 验证状态
  - 限制或备注
- 如果某项没有验证，也必须保留这一行，并明确写 `not verified` 和原因。

### Lint

- 必须说明是否在仓库根目录执行了 `yarn lint`。
- 写明 lint 结果是通过还是失败。
- 如果失败，列出与本次改动相关的失败项、处理结果，以及未能修复的原因。
- 如果因为环境问题未能执行，说明具体原因，不能简单写“未执行”。

### Review Findings

- 列出 Reviewer 的关键问题。
- 对每个问题标明最终状态：
  - fixed
  - partially fixed
  - not fixed

- 如果不是 `fixed`，说明原因。

### Review Resolution Table

- 必须列出 Writer 对 findings 的逐条处理结果。
- 推荐列至少包括：
  - finding 编号或标题
  - 状态
  - 处理说明
  - 对应文件或章节

### Red Hat References

- 至少列出 `1-3` 篇 Red Hat 官方文档。
- 对每篇参考文档都写清楚：
  - 文档标题
  - URL
  - 为什么选择它作为本次对标样本
  - 它对应本次文档里的哪些功能点、章节或写法
- 如果没有完全对应文档，标注“最近似参考”，并说明差异来源。

### Red Hat Mapping Table

- 必须提供一张对标表。
- 推荐列至少包括：
  - 本次文档章节/功能点
  - 对应 Red Hat 文档
  - URL
  - 对标原因
  - 当前对齐情况
  - 剩余差距
- 如果某个章节没有合适的 Red Hat 对标项，也要在表中明确写 `无直接对应`，不要直接省略。

### Comparison with Red Hat OCP Documentation

- 用明确的对标矩阵或逐项列表来写，不要只写一段主观总结。
- 至少覆盖这些维度：
  - task orientation
  - prerequisite clarity
  - procedure clarity
  - verification quality
  - troubleshooting quality
  - operational precision
  - risk and limitation disclosure
  - sample realism
- 每个维度都要写清楚：
  - 参考的是哪篇 Red Hat 文档
  - 本次文档对标了哪个功能点或结构
  - 已经对齐的地方
  - 仍然存在的差距
- 单独补一小节，说明哪些实际改动是直接受 Red Hat 文档启发而新增、重写或收紧的。

### Risks and Open Questions

- 列出剩余歧义、缺失验证项、以及仍需确认的产品行为。

### Score

- 给出 `10` 分制评分。
- 用 2 到 4 句话解释评分理由。
- 如果因为缺少对应 Red Hat 参考、验证不充分或差距较大而扣分，要明确说出扣分项。
- 如果触发了封顶条件，也要明确写出是哪一条封顶规则在生效。

## 评分封顶规则

- 未执行 `yarn lint`，或未明确说明无法执行的具体原因时，最高 `8.0`。
- 没有真实的 Validation evidence table，只有笼统描述时，最高 `8.0`。
- 集群相关内容没有真实 `kubectl`、`kubectl explain`、资源检查或等效证据支撑时，最高 `7.5`。
- 没有给出 Red Hat 官方 URL，或 Red Hat comparison 只有主观结论没有映射关系时，最高 `7.5`。
- Review findings 没有逐条关闭状态，或 Writer 没有给出 resolution table 时，最高 `8.0`。
