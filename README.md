# spec-marketplace

深度学习科研的 [Trellis](https://docs.trytrellis.app) spec 模板市场。`trellis init --registry` 消费，作为各科研项目 `.trellis/spec/` 的起点；配套的科研工作流部件（研究记忆 hook、实施前审查 agent、workflow 门禁）经 `docs/bootstrap/` 装配。

## 快速使用

```bash
# 新科研项目
trellis init --registry gh:weanqin/weanqin-spec-marketplace/marketplace --template dl-research

# 已有项目补装（只补缺失文件）
trellis init --registry gh:weanqin/weanqin-spec-marketplace/marketplace --template dl-research --append

# 装配 spec 之外的科研工作流部件（AI 可执行手册）
# 让项目里的 AI 阅读 docs/bootstrap/bootstrap.md 并执行
```

安装源会被记录到项目，`trellis update` 可从本仓库刷新 spec；本地修改按冲突流程提示保留或覆盖。

## 模板清单

| id | 说明 |
|---|---|
| `dl-research` | 深度学习科研规范（基础）：实验日志、路径隔离、可复现性、产物与 checkpoint 管理、评估口径、数据契约。方向无关，装后按方向本地演化。详见 [marketplace/specs/dl-research/README.md](./marketplace/specs/dl-research/README.md) |

## 仓库结构

```
marketplace/            # trellis 消费的清单目录（index.json + specs/）
docs/bootstrap/         # spec 外部件的 AI 可执行装配手册
assets/                 # 手册引用的权威文件副本（hook / review agent / 研究档案模板）
```

## 版本契约

- 模板 **id 是公开 API**：`dl-research` 只做增量演进（新增文件、新增条目、非破坏性修订）
- 破坏性重写 → 发布新 id（如 `dl-research-v2`），或在消费侧用 `#ref` 钉住旧版本
- 模板内容装进项目后归项目所有；本地改动与上游刷新的冲突由 `trellis update` 流程处理

## 回沉淀流程（项目经验 → marketplace）

1. 项目中验证有效的新规范/新模板，先在项目内稳定运行
2. 去除项目特定内容（数据集名、路径、结论），泛化为通用条款
3. 提交到本仓库对应模板（新增文件需同步该模板 README 与本文件清单）
4. commit 时在 message 里注明来源项目与验证场景

## 维护约定

- 模板内容保持与研究方向无关；方向特定内容只存在于各项目的 `.trellis/research/` 档案与本地 spec
- 不入库：密钥、运行态（tasks/workspace）、平台 prompt 文件、单仓库 PRD
- 本仓库语言：中文
