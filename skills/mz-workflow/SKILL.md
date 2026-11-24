---
name: mz-workflow
description: 民政组新需求的默认工作流（组内公共技能），适用于 Claude Code / Codex / Cursor / CommandCode / OpenCode / Gemini / Kimi / Grok / Cline / Pi 等 agent。当用户在民政组 Java/Maven/Gradle 项目中用一句话提出新需求、新功能或行为变更时使用，例如"加一个XX功能""支持XX""做一个XX接口""把XX改成XX""低保申请超时自动退回"。触发后按需求拷问（grilling）→ 方案设计（brainstorming）→ OpenSpec 提案（openspec-propose）→ 任务拆解（writing-plans）→ 实施推进，提案和计划产出后停下来等实施确认。不要用本技能处理纯问答、代码解释、已定位的小 bug、格式化或依赖升级。即使 brainstorming 的描述也适用于"新功能"，也必须先走本流程。
---

# 民政组需求工作流（mz-workflow）

这是民政组的**公共需求工作流**：把一句话需求，按统一节奏推进到可评审的提案、可执行的计划和可验证的代码。组内成员无论用哪个 agent，都按同一套产出物协作，方便评审、交接和归档。

本技能按民政组项目约定组织需求、方案和实施，使用当前客户端可用的技能与工具。

文中"调用技能 X"按当前 agent 的能力执行：

- **支持技能调用的 agent**（Claude Code、Codex、Cursor、CommandCode、OpenCode、Kimi、Grok、Cline、Pi 等）：按技能名调用 `X`。
- **不支持技能调用**：读取当前 agent 的 skills 目录下 `X/SKILL.md` 并遵循；该技能不存在时，用 `references/` 里的降级版。
- **没有子代理能力**：自己用搜索/读取工具查事实，不要因此把事实问题推给用户。

缺少配套技能时，按[阶段契约与降级表](references/phase-contracts.md)继续。

## 组内约定（优先于本技能的默认值）

- **项目规范优先（仅限做法）**：项目内的 `CLAUDE.md` / `AGENTS.md` / `GEMINI.md` / `README` 可以覆盖产出路径、命名、技术栈等项目做法；**三个 Gate 和“确认前不改生产代码”是不可覆盖的硬规则**，冲突时在开场说明，并以硬规则为准。
- **产出物统一**：提案放 `openspec/changes/<change-id>/`，设计和实施计划分别写入同目录的 `design.md` 与 `plan.md`。
- **三个 Gate 必须停**：需求决策记录、方案选择、变更批准，都要等用户明确确认。
- **确认前不改生产代码**：`openspec-propose` 只授权规划，实施需要新的用户请求。
- **共用一份标准源**：在维护仓库更新本目录，并同步给使用者。
- **技能优先级**：民政组项目优先使用 `mz-workflow`；如果机器上同时安装了 `java-req-flow`，按本技能执行。

## 0. 开场（只发一条消息）

1. 用一句话复述需求（你的理解，不是原话照抄）。
2. 声明流程：**需求拷问 → 方案设计 → [原型验证] → OpenSpec 提案 → 任务拆解 → 实施**（方括号表示条件触发）。
3. 说明：提案和计划产出后会停下来等确认，确认前不改生产代码。
4. 然后立即进入阶段 1，**不要问"要不要走流程"**。

如果用户说"快速做 / 跳过拷问 / 直接改"，走文末的轻量路径。

## 1. 事实扫描（自己查，不问用户）

用当前 agent 的搜索/读取工具（需要委派时按用户授权与宿主规则执行），确认：

- 构建：`pom.xml` / `build.gradle` / `settings.gradle`；Java 版本；Spring Boot 版本；单模块还是多模块
- OpenSpec：项目配置、CLI 版本和进行中的 change。根据当前 CLI 的帮助和实际输出确认初始化状态。
- 项目规范：`CLAUDE.md` / `AGENTS.md` / `GEMINI.md` / `README`；包结构；测试框架
- 需求相关代码：Controller / Service / Mapper / Entity / 配置 / 定时任务 / MQ
- 构建基线：`mvn -v`、父 POM 是否在本地仓库、项目配置的 Maven 仓库是否可达（不要在没验证前声称"测试能跑"）

**事实自己查，决策问用户。** 如果当前工作目录不是项目根（例如在 `chat-sessions/...` 下），停下来让用户打开 Java 项目文件夹，不要猜路径。

## 2. 需求拷问（Gate 1）

调用技能 `grilling`。

- 在 Claude Code 里 **不要调用 `grill-me`**：它带 `disable-model-invocation: true`，模型无法调用；`grilling` 是它的底层引擎，行为一致。
- 其他 agent 如果没有 `grilling`，按 `references/grilling-fallback.md` 的协议执行。
- 按 grilling 的"轮次 / 前沿"协议提问：每轮问完当前能问的问题，每题给出你的推荐答案，等用户回答后再问下一轮。
- 需要环境事实就自己查（获授权时可委派），不要问用户。
- 当前可提问的决策已解决后，整理需求决策记录供用户确认。

产出**需求决策记录**（对话内，模板见 `references/phase-contracts.md`）：

- 目标 / 非目标
- 用户与场景
- 范围边界（做什么、不做什么）
- 验收标准（可测试）
- 非功能约束（性能、并发、兼容、安全、可观测）
- 关键决策与放弃的方案
- 假设与待验证项

**Gate 1**：用户确认决策记录后才进入阶段 3。未确认不得进入方案设计。

## 3. 方案设计（Gate 2）

调用技能 `brainstorming`；不可用时按 `references/phase-contracts.md` 的降级版执行。无论用哪种方式，都明确约束：

- 需求已由 grilling 定稿，**不要再问一遍需求问题**；只发散解决方案。
- 给出 2–3 个方案，写清取舍、风险、对现有 Java 模块/接口/数据的影响，并给出推荐。
- **不要写任何 `docs/` 设计文档**：设计随后由 OpenSpec 的 `design.md` 承载（组内约定覆盖技能默认路径）。
- 不改任何代码。

**Gate 2**：用户选定方案（或明确接受你的推荐）后才进入阶段 3.5。

## 3.5. 原型验证（条件触发）

**前端改动必问**：只要方案涉及前端修改，就问用户「这次涉及前端改动，是否调用 `prototype` 制作 UI 原型？」并等待答复。页面、组件、交互、样式、文案及简单 CRUD 的前端改动都适用，轻量路径也适用。用户确认后制作 UI 原型，用户选择跳过时继续剩余流程。

**逻辑原型独立判断**：涉及复杂状态机、多状态流转、审批分支，或用户要求先验证逻辑时，制作逻辑原型。纯后端简单 CRUD、配置修改、依赖升级和格式化可跳过原型阶段。两类原型都跳过时，直接进入阶段 4。

**执行流程**：

1. **判断原型类型**：
   - **逻辑原型**：涉及状态机、审批流、定时任务等 → 按[逻辑原型模板](references/prototype-templates.md#逻辑原型模板详细版)验证状态、转换和边界条件
   - **UI 原型**：用户确认需要 UI 原型 → 调用 `prototype` 的 UI 原型流程
   - **两者都需要**：先做逻辑原型验证状态流转，再做 UI 原型选择界面风格

2. **产出物**：
   - 逻辑原型：一个可交互的 HTML 文件，模拟状态机的各种路径（按钮触发状态变化、展示历史记录、可视化当前状态）
   - UI 原型：3-5 个不同风格的 UI 变体，通过 picker 切换（参考 `prototype` 技能的 `PICKER.md`）

3. **用户验证**：
   - 将原型放在临时工作目录或用户指定的位置，并展示文件路径或 URL
   - 说明如何操作（点击按钮、切换变体、查看状态）
   - **停下来等待用户反馈**，不要自动进入下一阶段

4. **根据反馈调整**：
   - 发现需求遗漏 → 回到阶段 2 补充需求决策记录
   - 发现方案问题 → 回到阶段 3 调整方案
   - 确认无误 → 进入阶段 4

**UI 原型降级**：如果 `prototype` 技能不可用，按[原型模板](references/phase-contracts.md#原型验证模板阶段-35)构建简化版。

**示例场景**：
```markdown
需求：低保申请超时自动退回

触发条件：涉及状态机（待审核 → 超时退回 → 待提交）

原型验证：
1. 生成逻辑原型 HTML：
   - 按钮：提交申请、审核通过、审核拒绝、等待超时
   - 展示：当前状态、历史记录、超时倒计时
   - 用户可以快速点击各种操作，验证状态流转

2. 用户发现问题：
   - "退回后再提交，审核记录应该保留还是清空？"（需求遗漏）
   - "多次超时退回，是否要限制次数？"（边界条件）

3. 回到阶段 2 补充需求，更新决策记录
```

## 4. OpenSpec 提案（Gate 3 的前半）

调用技能 `openspec-propose`；不可用时用 CLI 降级（见 `references/phase-contracts.md`）。把以下内容作为输入（对话上下文）：

- 原始一句话需求
- 需求决策记录
- 选定方案与放弃项
- 阶段 1 查到的相关代码位置

要求：

- change id 用 kebab-case、动词开头，例如 `add-low-income-timeout-return`。
- 四件套齐全：`proposal.md`、`specs/<capability>/spec.md`、`design.md`、`tasks.md`。
- 新能力必须有 `## Purpose`（≥50 字符）；需求用 SHALL/MUST；每个 Requirement 至少一个 `#### Scenario`，场景用 WHEN/GIVEN/THEN。
- 如果项目尚未初始化：先问用户是否执行 `openspec init --tools <当前 agent>`，**不要隐式创建**。
- `openspec-propose` 只做规划，写完就停，不改代码。

## 5. 任务拆解（Gate 3 的后半）

调用技能 `writing-plans`；不可用时按 `references/phase-contracts.md` 的计划模板直接写。

- 输入：`openspec/changes/<id>/design.md` 和 `specs/`。
- 输出：写到 `openspec/changes/<id>/plan.md`（组内约定覆盖默认的 `docs/superpowers/plans/` 路径）。
- 计划头部写 `Spec: openspec/changes/<id>/design.md`，任务编号与 `tasks.md` 对齐。
- 每个任务必须包含：涉及文件、接口签名、失败测试、通过标准、验证命令。
- Java 专项检查见 `references/java-checklist.md`。

然后运行 `openspec validate <id> --strict`，失败就修到通过。

**Gate 3**：向用户展示 change id、proposal 摘要、specs 变更点、任务数、风险，明确说"回复『实施』才开始改代码"。

> OpenSpec 的边界：propose 只授权规划，实施需要新的用户请求。不要把最初那一句话当成实施授权。

## 6. 实施（用户确认后）

用户批准实施后，首次修改生产代码前，用一句话说明本次改动范围和验证方式。执行中遇到影响验收的阻塞或方案变化时，通过客户端的进度消息同步事实、影响和下一步。

1. 调用技能 `openspec-apply-change`（不可用时按 `tasks.md` 逐项执行），细节看 `plan.md`。
2. 每个任务先调用技能 `test-driven-development`：RED → GREEN → 重构；没有失败测试不写生产代码。
3. 失败时调用技能 `systematic-debugging`。
4. **UI 任务的额外检查**（任务涉及前端界面时）：
   - 完成后调用技能 `break-ui`：用极端数据（长名称、emoji、空值、超大数字）测试 UI，修复布局问题
   - 涉及动画或过渡效果时，调用技能 `animate`：补充动画细节（easing、duration、reduced-motion）
5. 如果用 git worktree：确认新 worktree 能读取已批准的 change。若它依赖尚未提交的文件，使用宿主支持的交接方式，或在原工作区实施。额外提交须有授权。
6. **Git 提交策略**：**不要每完成一个 task 就自动提交代码**。等所有 tasks 完成并通过验证后，统一提交一次或按逻辑模块分批提交。中间过程的 checkpoint 提交需要用户明确授权。

## 7. 收尾（验证与归档）

1. 调用技能 `verification-before-completion`：贴出实际命令和输出，禁止口头声称通过。
2. 调用技能 `requesting-code-review`，按反馈修正。
3. 调用 `openspec-verify-change`；需要归档时用 `openspec-archive-change`。
4. 更新 `tasks.md` 复选框，确认 `openspec validate <id> --strict` 通过。

## 硬规则

- 未确认需求决策记录，不进入方案设计。
- 未选定方案，不进入 OpenSpec。
- 未批准 change，不写生产代码。
- 事实自己查，决策问用户。
- **技能名用裸名**（`grilling`、`brainstorming`、`openspec-propose`、`prototype`、`break-ui`、`animate` 等）。支持技能调用就用技能名；不支持就读取同名 `SKILL.md`；不要写 `superpowers:` 或 `mattpocock-skills:` 前缀。
- 技能缺失时按 `references/phase-contracts.md` 的降级表处理；**绝不假装某个技能已经运行**。
- 原型验证、UI 压力测试、动画补充都是**质量增强手段**，不是必需步骤；触发条件不满足时跳过，不要强行触发。

## 轻量路径（用户明确要求跳过拷问时）

1. 核对验收标准、范围和兼容要求，只问尚未解决且会改变交付的问题。给出精简决策记录，等用户确认（Gate 1）。
2. 给出一个推荐方案和主要取舍，等用户确认（Gate 2）。原型按阶段 3.5 的条件触发。
3. 按阶段 4–5 生成并验证 OpenSpec 提案与计划。用户明确批准实施后再修改生产代码（Gate 3）。

## 安装与维护

从维护仓库安装本目录及 `references/`。将 `mz-workflow` 放入客户端支持的技能目录，或链接到该目录。可独立安装配套技能，缺失时使用上述降级表。

修改流程时，更新本目录及受影响的阶段模板。
