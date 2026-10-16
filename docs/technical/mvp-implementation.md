# MVP 落地与口令约定

日期：2026-10-16

设计源文件：本目录 `docs/product/overview.md`、`docs/product/requirements.md`、`docs/technical/`、`skills/active-reading-learning-rules/`。  
参考实现位于独立运行环境的 `<APP_HOME>/`，不包含在本仓库中。

## 技术形态

| 项 | 约定 |
|---|---|
| 产品壳 | Grok Bot / Flash 聊天内（非独立 Web/App） |
| 持久化 | 运行环境中的 SQLite：`<APP_HOME>/data/active_reading.sqlite` |
| 调度 | 本地 FSRS（`fsrs` 库；时区 Asia/Shanghai 自然日，库内时间 UTC ISO-8601） |
| Bot | 对话 UX：选集、选择题选项卡、四档评分卡、口令路由 |
| CLI | `<APP_HOME>/` 下 `review.py` / `manage.py` / `practice.py` / `feynman.py` / `backup.py` |

## 新建卡片流水线

1. **新建卡片**（用户口令）→ `manage.py content-add`：原始输入进入某卡片集，成为 ContentCard（原料）。
2. **生成练习草稿** → `manage.py generate`（或 `draft-import`）：产出概念 / 问答卡 / 开放题 / 摘要草稿，**不进**今日复习。
3. **发布练习** → `manage.py draft-publish`：确认后问答卡进入可复习队列（FSRS 初始化）。

「新建卡片」**不等于**创建可复习问答卡。

## 口令 ↔ CLI（精简）

| 用户口令 | CLI |
|---|---|
| 今日复习 / 选集 | `review.py sets` → `review.py start --set <id>`（`--set` 必填） |
| 今日总览 | `review.py today` |
| 结束 | `review.py end --session S` → **SESSION_ABORTED** |
| 继续 | 同场或再次 `start --set` 后抽已到点卡 |
| 新建卡片集 | `manage.py set-create --name ...` |
| 卡片集列表 | `review.py sets`（展示还剩 x/全部） |
| 新建卡片 | `manage.py content-add --set ID --title ... --body "..."` |
| 生成练习草稿 | `manage.py generate --content ID` |
| 发布练习 | `manage.py draft-list` → `draft-publish --batch B` |
| 深度练习 | `practice.py summary-start` / `open-list` … |
| 费曼 | `feynman.py start --content / --set / --note …`（不从复习自动进入） |
| 暂停 / 恢复卡片 | `review.py suspend` / `unsuspend` |
| 设置保留率 | `review.py config retention --value 0.9` |
| 备份 / 恢复备份 | `backup.py export` / `import` |

## 还剩 x/全部

`review.py sets` 展示 **还剩 remaining_due / total_active**：

- **x（还剩）**：该集今日到期或已到期、可立刻抽出的活跃问答卡数（未挂起）。
- **全部**：该集未挂起的活跃问答卡总数。

## 会话结束：SESSION_COMPLETE vs SESSION_ABORTED

| 结果 | 触发 | 行为 |
|---|---|---|
| `SESSION_COMPLETE` | 队列暂时无可立刻出的卡（自然完成） | 会话 `status=completed` |
| `SESSION_ABORTED` | 用户说「结束」主动中止 | 会话 `status=aborted`；**未评分卡不打分** |

## Learning 短间隔与通知

- 学习中（learning）短间隔卡在本场自然结束后**仍保留 due**。
- **默认不自动推送通知**；用户再次说「今日复习」并选集后才抽出已到点卡。
- 若跨日未练，按逾期处理。
- 到点提醒为可选后续例程，非 MVP 默认。

## 深度练习 vs 费曼

| | 深度练习 | 费曼 |
|---|---|---|
| 形态 | 摘要 / 开放题「先写再对照」 | 用户主动讲清楚 + 学生式追问 |
| FSRS | 不进日常间隔表 | 不进日常间隔表 |
| 入口 | 用户口令「深度练习」 | 用户口令「费曼」；**不从今日复习自动进入** |

## 相关文件

| 路径 | 作用 |
|---|---|
| `docs/product/overview.md`、`docs/product/requirements.md`、`docs/technical/` | 主产品 / UX 设计 |
| `skills/active-reading-learning-rules/SKILL.md` | Grok Bot 学习规则 Skill |
| `skills/active-reading-learning-rules/references/output-contracts.md` | 输入输出契约 |
| 实现仓库中的 `README.md` | CLI 与 P0 清单 |

## MVP 落地状态（2026-10-16）

| 能力 | 状态 | 说明 |
|---|---|---|
| P0 今日复习闭环（答→揭示→自评→FSRS） | 已在 MVP 实现 | `<APP_HOME>/review.py` |
| 选集必选、还剩 x/全部 | 已实现 | `review.py sets` / `start --set` |
| 会话结束：COMPLETE vs ABORTED | 已实现 | 自然完成 vs 用户「结束」 |
| 草稿生成与发布 | 已实现 | 启发式 + `draft-import`；未发布不进今日复习 |
| 深度练习（摘要 / 开放题） | 已实现 | 不进日常 FSRS |
| 费曼 | 已实现 | 显式启动；不从复习自动进入 |
| 挂起、保留率、审计、备份 | 已实现 | CLI + SQLite |
| LLM 草稿质量 | 部分 | 启发式为主；Flash/LLM 经 draft-import 增强 |
| 定时到点提醒（含 learning 短间隔） | 未做 / 非默认 | 产品可有通知服务；MVP 默认关闭短间隔推送 |
| 微信读书导入 | 未做 | 设计见 R09 |
| 多用户鉴权 | 非本 MVP 范围 | 单用户部署 |
