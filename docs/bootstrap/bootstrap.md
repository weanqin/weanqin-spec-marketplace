# 科研工作流装配手册（AI 可执行）

> **读者**：目标项目里执行装配的 AI 助手。**前提**：项目已 `trellis init`（或即将 init）、平台为 ZCode、trellis ≥ 0.6（支持 `init --registry`）。
> 本手册覆盖 spec 模板**之外**的配套部件——官方 marketplace 只装载 `.trellis/spec/`，研究记忆 hook、审查 agent、workflow 门禁、档案模板由本手册装配。
> 执行原则：逐步做、每步验证；workflow.md 的增改必须适配目标项目当前结构，不得破坏既有 `[workflow-state:*]` 标签块。

---

## 第 1 步 · 安装 spec 模板

```bash
# 新项目
trellis init --registry gh:<GitHub用户名>/spec-marketplace/marketplace --template dl-research
# 已有项目（只补缺失文件）
trellis init --registry gh:<GitHub用户名>/spec-marketplace/marketplace --template dl-research --append
```

**验证**：`.trellis/spec/research/index.md` 与 `.trellis/spec/data/index.md` 存在。

## 第 2 步 · 研究 hook

1. 复制 `assets/inject-research-context.py` → `.zcode/hooks/inject-research-context.py`
2. 在 `.zcode/config.json` 的 `SessionStart`（matcher `startup|clear|compact`）hooks 数组追加：

```json
{
  "type": "command",
  "command": "python3 \"${ZCODE_PROJECT_DIR}/.zcode/hooks/inject-research-context.py\"",
  "timeout": 15
}
```

**验证**：`echo '{"cwd":"<项目根>"}' | ZCODE_PROJECT_DIR=<项目根> python3 .zcode/hooks/inject-research-context.py` 输出含「研究记忆未初始化」的 JSON。

## 第 3 步 · 实施前审查 agent

复制 `assets/trellis-review.md` → `.zcode/agents/trellis-review.md`。

**验证**：文件存在；agent 以 `trellis-review` 类型可被 Agent 工具派发（新会话生效）。

## 第 4 步 · 研究记忆档案模板

复制 `assets/research-memory/{context,progress,environment}.md` → `.trellis/research/`（目录不存在则创建）。**保留各文件顶部的 `<!-- trellis-research: uninitialized -->` 标记**——未初始化状态下 hook 会注入「索取资料、禁止推断」守卫提示。

## 第 5 步 · config.yaml

```yaml
session_auto_commit: false   # 非 git 仓库时；git 仓库可保持默认 true

hooks:
  after_create:
    - 'cp "$(dirname "$TASK_JSON_PATH")/../../spec/research/experiment-template.md" "$(dirname "$TASK_JSON_PATH")/experiment.md"'
```

**验证**：`task.py create "播种测试" --slug seed-test --description "..."` 后任务目录出现 `experiment.md`，随后删除测试任务。

## 第 6 步 · workflow.md 增改（按项目实际结构适配）

以下为权威参照文本（源自已验证的项目定制）。落在哪些位置由目标项目 workflow 结构决定：面包屑文本改 `[workflow-state:planning]` 块（保持标签开闭一致）；门禁改 Phase 1.4；其余为段落级插入。

**(a) `[workflow-state:planning]` 块内追加两行**：

```
Research tasks: declare variant directories and output paths in `design.md` (`.trellis/spec/research/path-isolation.md`); register purpose+config in `{TASK_DIR}/experiment.md` before any run.
Start gate (1.4): complex tasks dispatch `trellis-review` (methodology + implementation risk, 1–2 dispatches), then present the implementation briefing (改动点 / 关键逻辑 / 潜在风险); run `task.py start` ONLY after the user explicitly confirms. Lightweight tasks still require briefing + explicit confirmation.
```

**(b) Phase 1.4 改为「Plan review & activation」三段门禁**：1.4.1 复杂任务派发 `trellis-review`（Mandate A 方案与方法论 / Mandate B 实施与风险，默认两次独立派发，发现影响方案的问题则回到 1.1 修改后重审）→ 1.4.2 主会话向用户交付固定格式实施简报（改动点 / 关键逻辑 / 潜在风险 / 路径与环境 / 验证方案 / 审查发现处理）→ 1.4.3 用户明确肯定（如「确认开始」「同意」）后才 `task.py start`；沉默、跑题、对方案的正面评价都不算同意。

**(c) 「Trellis System」区插入 Research Memory System 段**（要点）：`.trellis/research/{context,progress,environment}.md` 三份档案由 SessionStart hook 注入；维护规则为节点/决策/环境变化当次回填；**初始化机制**——档案带 `<!-- trellis-research: uninitialized -->` 标记时 hook 跳过，全部未填时注入「未初始化」提示；初始化 = 用户提供资料后据实填写并删标记，**不得从仓库推断**。

**(d) Phase 2.1 派发要求追加**：研究任务的 implement 派发提示必须包含——已确认的运行环境（来自 environment.md，未记录先问用户）、变体目录与输出落点、跑前登记 / 跑后回写 experiment.md。

**(e) Phase 2.2 检查项追加**：experiment.md 无空 Run 块；产物全部落在任务声明的变体目录内。

**(f) Phase 3.3 追加「Research record sync（研究任务 · required）」**：Run 块四要素齐全；progress.md 当前状态同步 + 里程碑追加；有长期价值的实验提炼进长期实验笔记；方向/环境结论变化当次更新 context.md / environment.md。

**(g) Guardrails 追加**：实施同意门禁（审查→简报→明确同意，缺一不得开工）；路径隔离；环境确认（未记录必问，不猜不自装）；研究连贯性（当次回填研究记忆 + 收尾记 journal）。

**验证**：`UserPromptSubmit` hook 正常输出 planning 面包屑（标签开闭一致、无解析降级）。

## 第 7 步 · 总体验证清单

- [ ] 新会话 SessionStart 注入「研究记忆未初始化」提示（未初始化时的正确行为）
- [ ] 用户提供资料初始化后，注入为实际档案内容
- [ ] `task.py create` 自动播种 experiment.md
- [ ] planning 面包屑含 Start gate 提示
- [ ] 复杂任务流程走到 1.4 时派发 trellis-review 并出具实施简报

## 已知边界

- marketplace 与 `trellis update` 只管 `.trellis/spec/`；本手册装配的部件更新需手动同步（或重读本手册比对）
- 本手册面向 ZCode；其他平台 hook 注册方式不同，需按平台映射调整
