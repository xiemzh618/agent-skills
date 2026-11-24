# 阶段契约与模板（mz-workflow）

## 阶段输入/输出一览

| 阶段 | 技能 | 输入 | 输出 | Gate |
|---|---|---|---|---|
| 1 事实扫描 | 搜索与读取 | 一句话需求 | 构建、OpenSpec、代码位置、构建基线 | 无 |
| 2 需求拷问 | `grilling` | 一句话需求 + 事实 | 需求决策记录（对话内） | 用户确认决策记录 |
| 3 方案设计 | `brainstorming` | 决策记录 | 2–3 个方案 + 推荐（对话内） | 用户选定方案 |
| 3.5 原型验证 | 逻辑原型模板 / `prototype` | 选定方案 + 决策记录 | 按需制作逻辑或 UI 原型 | 涉及前端先询问是否制作 UI 原型，制作后由用户验证 |
| 4 变更提案 | `openspec-propose` | 决策记录 + 选定方案 | `openspec/changes/<id>/` 四件套 | — |
| 5 任务拆解 | `writing-plans` | `design.md` + `specs/` | `plan.md` + 对齐的 `tasks.md` | 用户批准 change |
| 6 实施 | `openspec-apply-change` + TDD | `tasks.md` + `plan.md` | 代码 + 测试 + 验证输出 | 每任务 RED/GREEN |
| 6.5 UI 质量增强 | `break-ui` + `animate` | 完成的 UI 任务 | 极端数据测试 + 动画补充（条件触发） | 无 |
| 7 收尾 | `openspec-verify-change` / `openspec-archive-change` | 实施结果 | 验证报告 / 归档 | 验证通过 |

## 需求决策记录模板

```markdown
## 需求决策记录：<一句话标题>

### 目标
- ...

### 非目标
- ...

### 用户与场景
- 谁：...
- 什么时候：...
- 现在怎么做：...
- 期望怎么做：...

### 范围边界
- 做：...
- 不做：...

### 验收标准
1. Given ... When ... Then ...
2. ...

### 非功能约束
- 性能：...
- 并发/幂等：...
- 兼容性：...
- 安全：...
- 可观测：...

### 关键决策
| 决策 | 选择 | 放弃的方案 | 理由 |
|---|---|---|---|

### 假设与待验证项
- [ ] ...
```

## OpenSpec 产物规则

- 目录：`openspec/changes/<change-id>/`
- 文件：`.openspec.yaml`、`proposal.md`、`specs/<capability>/spec.md`、`design.md`、`tasks.md`
- change id：小写字母/数字/连字符，不能以连字符开头或结尾，不能有连续连字符，最长 200 字符
- delta 段落只能是：`## ADDED Requirements`、`## MODIFIED Requirements`、`## REMOVED Requirements`、`## RENAMED Requirements`
- 每个需求：`### Requirement: <名称>` + SHALL/MUST 描述
- 每个场景：`#### Scenario: <名称>` + WHEN/GIVEN/THEN；每个需求至少一个场景
- 新能力：必须有 `## Purpose`（≥50 字符）；已有能力的 delta 不要加 Purpose
- MODIFIED：必须整段复制原需求再改；REMOVED：必须写 **Reason** 和 **Migration**
- 需求描述建议 ≤500 字符（strict 会标记过长）
- 常用命令（以 `openspec --help` 为准）：
  - `openspec list --json`（查看 change 状态。初始化判断以当前 CLI 的输出为准）
  - `openspec new change <id> --description "<text>"`
  - `openspec status --change <id> --json`
  - `openspec instructions <proposal|specs|design|tasks|apply> --change <id> --json`
  - `openspec validate <id> --strict`
  - `openspec archive <id> --yes`

## 实施计划模板（plan.md）

```markdown
# <Feature> 实施计划

Spec: openspec/changes/<id>/design.md
Tasks: openspec/changes/<id>/tasks.md

## Goal
...

## Architecture
...

## Tech Stack
...

## Global Constraints
- Java 版本 / Spring Boot 版本
- 事务、幂等、兼容性约束
- 构建与测试命令

### Task 1: <名称>
**Files**
- Create: `path/To/File.java`
- Modify: `path/To/Existing.java`
- Test: `src/test/java/.../FileTest.java`

**Interfaces**
- Consumes: ...
- Produces: `ReturnType methodName(ParamType param)`

**Steps**
- [ ] 写失败测试（贴出测试代码或断言）
- [ ] 运行：`mvn -q -pl <module> -Dtest=<TestClass> test`（Gradle：`./gradlew :<module>:test --tests <TestClass>`），确认 RED
- [ ] 写最小实现
- [ ] 再运行，确认 GREEN
- [ ] 提交
```

## 原型验证模板（阶段 3.5）

### 逻辑原型模板

用于验证状态机、审批流、定时任务等复杂逻辑。

```html
<!DOCTYPE html>
<html>
<head>
  <title>逻辑原型：<需求名称></title>
  <style>
    body { font-family: system-ui; max-width: 800px; margin: 40px auto; }
    .state-machine { border: 1px solid #ccc; padding: 20px; margin: 20px 0; }
    .current-state { font-size: 24px; font-weight: bold; color: #007bff; }
    .actions { margin: 20px 0; }
    button { margin: 5px; padding: 10px 20px; cursor: pointer; }
    .history { margin-top: 20px; max-height: 300px; overflow-y: auto; }
    .history-item { padding: 5px; border-bottom: 1px solid #eee; }
  </style>
</head>
<body>
  <h1>逻辑原型：<需求名称></h1>

  <div class="state-machine">
    <div>当前状态：<span class="current-state" id="current-state">初始状态</span></div>
    <div id="state-details"></div>
  </div>

  <div class="actions">
    <h3>操作：</h3>
    <!-- 根据状态机动态生成按钮 -->
  </div>

  <div class="history">
    <h3>历史记录：</h3>
    <div id="history-list"></div>
  </div>

  <script>
    // 状态机逻辑
    const stateMachine = {
      currentState: '初始状态',
      history: [],

      transition(action) {
        const from = this.currentState;
        // 根据当前状态和操作计算新状态
        const to = this.calculateNextState(from, action);

        this.history.push({
          timestamp: new Date().toLocaleTimeString(),
          from, to, action
        });
        this.currentState = to;
        this.render();
      },

      calculateNextState(from, action) {
        // 状态转换逻辑（根据实际需求填充）
        return from;
      },

      render() {
        document.getElementById('current-state').textContent = this.currentState;
        // 渲染历史记录
        const historyHtml = this.history.map(h =>
          `<div class="history-item">${h.timestamp}: ${h.from} → ${h.to} (${h.action})</div>`
        ).join('');
        document.getElementById('history-list').innerHTML = historyHtml;
      }
    };
  </script>
</body>
</html>
```

### UI 原型模板

用于验证新增或改版的界面，生成 3-5 个不同风格的变体。

参考 `prototype` 技能的 `PICKER.md`，生成：
- 3-5 个不同风格的 UI 变体（如：简洁风格、密集风格、卡片风格）
- 底部居中的 picker（Demo data / Worst case）
- 通过 URL 参数 `?v=1` 切换变体
- 键盘快捷键 `1-5` 和 `←/→` 切换

## UI 质量增强模板（阶段 6.5）

### break-ui 检查清单

UI 任务完成后，用极端数据测试：

| 测试数据 | 用途 | 预期问题 |
|---|---|---|
| 超长名称（50+ 字符） | 用户名、标题、描述 | 文本溢出、布局撑破 |
| 超长 email | 邮箱地址 | 横向滚动、按钮被推出屏幕 |
| 单字符名称 | 缩写、昵称 | 头像显示异常、对齐问题 |
| Emoji | 用户名、评论 | 字体回退、高度异常 |
| 空值 | 可选字段 | 占位符错误、空白行 |
| 零项 | 列表、表格 | 空状态缺失 |
| 1 项 | 列表计数 | 复数形式错误（"1 items"） |
| 1000+ 项 | 分页列表 | 性能问题、滚动卡顿 |
| 超大数字 | 计数、金额 | 数字溢出、千分位缺失 |

### animate 检查清单

涉及动画或过渡效果时：

| 元素 | 检查项 | 正确做法 |
|---|---|---|
| 弹窗/抽屉 | easing、duration | `ease-out`，200-500ms |
| 下拉菜单 | easing、duration | `ease-out`，150-250ms |
| Toast | 中断处理 | 用 transition，不用 keyframes |
| 按钮按下反馈 | duration | 100-160ms |
| Hover 效果 | 触摸屏适配 | `@media (hover: hover)` 保护 |
| 所有动画 | reduced-motion | `@media (prefers-reduced-motion)` |
| 入场动画 | transform-origin | 从触发点展开（弹窗除外） |
| 高频动画（100+/天） | 是否需要动画 | 不要动画，保持即时响应 |

## 降级表

| 缺失 | 降级做法 |
|---|---|
| `grilling` | 用 `references/grilling-fallback.md` 的协议，明确告诉用户"grilling 技能不可用，使用内置协议" |
| `brainstorming` | 直接在对话里给 2–3 个方案 + 取舍 + 推荐，不写设计文档 |
| `prototype` | 用户确认需要 UI 原型后，按上面的 UI 原型模板用单个 HTML 文件构建简化版 |
| `openspec-propose` | CLI：`openspec new change <id> --description ...`，再按 `openspec instructions` 逐个写四件套，最后 `openspec validate <id> --strict` |
| `writing-plans` | 直接在 `openspec/changes/<id>/plan.md` 写任务清单，格式按上面的模板 |
| `openspec-apply-change` | 按 `tasks.md` 逐项实现，每项走 RED/GREEN，更新复选框 |
| `test-driven-development` | 仍先写失败测试并运行确认 RED，再写最小实现；没有现成测试框架时至少写可执行的验证脚本 |
| `break-ui` | 按上面的检查清单手工构造极端数据，截图对比 demo 和 worst-case，记录所有布局问题 |
| `animate` | 按上面的检查清单手工审查 CSS/JS 动画代码，补充缺失的 easing、reduced-motion、hover 保护 |
| `systematic-debugging` | 复现 → 最小化 → 定位根因 → 加回归测试；按项目日志和调试工具手工执行 |
| `verification-before-completion` | 贴出实际命令和输出，禁止口头声称通过 |
| `requesting-code-review` | 按 `plan.md` 和 diff 自查清单，或走项目现有评审流程 |
| `openspec-verify-change` / `openspec-archive-change` | `openspec validate <id> --strict` + 手工核对 specs/tasks；归档用 `openspec archive <id> --yes` |
| OpenSpec CLI 不可用 | 继续整理需求与方案。到需要生成或验证 change 时，说明 CLI 缺失和待完成的步骤。 |

## 触发验收清单

在民政组 Java 项目会话里测试：

1. 正例：`给低保申请加一个超时自动退回功能` → 应自动进入本技能，第一条回复复述需求并开始 grilling。
2. 正例：显式调用 `mz-workflow` 处理临时救助超期提醒 → 进入对应阶段。
3. 正例：`新增一个特困人员认定结果查询接口` → 应自动进入本技能。
4. 反例：`这个类是什么意思？` → 不应触发。
5. 反例：`把 UserService 的拼写改成 UserSvc` → 不应触发完整流程。
6. 反例：`这个空指针怎么修？`（已有堆栈）→ 应走 `systematic-debugging`，不走本流程。
