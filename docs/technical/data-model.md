# 数据模型

| 实体 | 关键字段 | 关系 |
|---|---|---|
| CardSet | id、name、description、tags、status | 包含多个 ContentCard；不含书籍专属必填字段 |
| ContentCard | id、card_set_id、title、content、content_type、source_url、source_meta、external_source_id、version、status、invalid_reason、invalidated_at、voided_at | 承载任意阅读输入，支持多个 KnowledgeNote；status 为 draft、active、invalid 或 voided |
| KnowledgeNote | id、content_card_id、concept、explanation、source_excerpt、status | 可派生多张 ReviewCard |
| ReviewCard | id、knowledge_note_id、question_type、question、answer、key_points、options、correct_option_ids、version、status、invalid_reason、invalidated_at、voided_at | question_type 为 qa 或 choice；status 为 draft、active、invalid、suspended 或 voided；一对一关联 ReviewState |
| ReviewState | review_card_id、state、due_at、stability、difficulty、last_review_at | 由 FSRS 服务更新 |
| ReviewLog | id、review_card_id、card_version、response_text、selected_option_ids、rating、reviewed_at、state_before、state_after | 每次复习一条；按题型保存文本或选项 |
| Practice | id、type、content_card_id、prompt、reference、due_at、status | 对应开放题或摘要 |
| FeynmanSession | id、knowledge_note_id、first_explanation、revised_explanation、status | 包含多条 DialogueTurn |
| DialogueTurn | id、session_id、role、content、source_refs | 保存费曼对话 |
| Weakness | id、session_id、description、status | 可生成补充 ReviewCardDraft |
| DeepPracticeSchedule | id、practice_type、target_id、scheduled_at、status、source | 只保存用户明确安排的深度练习 |
| AIJob | id、type、request_id、model_version、status、input_version、output | 跟踪 Grok Bot 调用 |
| ImportRecord | id、provider、external_source_id、raw_content_hash、content_card_id、imported_at、status | 记录微信读书等外部来源的导入映射，用于预览、去重和失败重试 |
| UserSettings | timezone、day_boundary、retention、daily_limits、max_interval | 控制队列与调度 |

## 核心业务规则

1. Grok Bot 返回的内容全部先进入草稿态，未经用户确认不得加入复习。
2. 卡片集和内容卡片不绑定书籍结构；除正文外，不要求作者、封面、章节或其他来源专属字段。
3. 微信读书数据只是一种输入来源。导入后仍统一保存为卡片集与内容卡片，不引入必须依赖书籍结构的核心实体。
4. 失效表示内容暂时不可信或不可用，可在修正和审核后恢复；作废表示用户确认不再使用，为不可恢复的终态，但不得删除历史记录。
5. 内容卡片失效后，其派生问答卡进入来源失效待处理状态并退出复习队列；内容卡片作废时，不得未经用户确认自动作废派生问答卡。
6. 问答卡只支持问答式和选择题两种题型；两种题型使用同一套 FSRS 状态，但保存不同的作答数据。
7. 主动回想必须遵循“先回答、后揭示”，不能通过接口预加载答案到客户端。
8. 用户评分是 FSRS 的唯一语义评分来源。Grok Bot 只能提供评价建议。
9. 选择“困难”表示已经正确回想但较费力；没有回想出来必须选择“忘记”。
10. 问答卡复习与深度练习是两条独立流程。完成问答卡复习后不得自动进入费曼解释。
11. 开放题、摘要和费曼练习属于理解训练，只有用户主动开始或明确安排后才进入深度练习队列，不与问答卡 FSRS 状态混用。
12. 问答卡连续出现“忘记”或“困难”时，系统只能建议费曼练习，由用户选择现在开始、稍后安排或忽略。
13. 费曼练习发现缺口后先保存缺口；只有用户选择生成并确认补充问答卡，问答卡才进入 FSRS。
14. 来源变化后，旧日志保持原引用；涉及答案含义的重大修改需要用户决定是否重置问答卡。
15. 同源问答卡暂缓只影响展示顺序，不产生失败记录。
16. 所有生成和评分提交都带请求 ID，重复请求不能重复创建内容或推进状态。
17. Grok Bot 无法从材料确认的内容必须标记为推断或待核实。
18. 时区和日界线统一以用户设置为准，默认 Asia/Shanghai。
19. 面向用户时，「新建卡片」仅表示原始输入入库到卡片集；生成练习草稿与发布练习是后续独立步骤，未发布不进今日复习。
20. 今日复习开场必须指定卡片集；列表须展示「还剩 x/全部」。
21. 复习场结束区分主动中止（SESSION_ABORTED，用户说「结束」）与自然完成（SESSION_COMPLETE，队列暂无可立刻出的卡）；中止时未评分卡不打分。
22. 短间隔 learning 卡到点后默认不主动通知，直至用户再次发起「今日复习」并选集（除非用户另开提醒例程）；跨日未练按逾期处理。
23. Bot 聊天壳下，选择题选项与四档评分优先用结构化选择 UI（widget），仍遵守先答后揭示；「想不起来」可用危险样式选项。
