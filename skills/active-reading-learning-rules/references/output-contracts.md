# 输入输出契约

> 术语对齐：用户口令「新建卡片」= ContentCard 入库；本契约中的 `generate_materials` 等模式只产出草稿。`evaluate_review_answer` 不包含四档评分字段——评分由应用层 widget 收集。费曼与深度练习为独立 `mode`。

## 通用请求

```json
{
  "request_id": "string",
  "mode": "generate_materials | evaluate_review_answer | evaluate_deep_practice | feynman_tutor | generate_supplemental_cards",
  "source_context": {
    "card_set_id": "string",
    "content_cards": [
      {
        "id": "string",
        "title": "string",
        "content": "string",
        "source_meta": {}
      }
    ]
  },
  "payload": {}
}
```

`request_id`、`mode` 和至少一张包含正文的内容卡片为必填。响应必须原样返回 `request_id`。

## 通用响应

```json
{
  "request_id": "string",
  "mode": "string",
  "status": "ok | needs_input | needs_human_check | rejected",
  "warnings": [],
  "result": {}
}
```

## generate_materials

`result`：

```json
{
  "draft": true,
  "concepts": [
    {
      "concept": "string",
      "explanation": "string",
      "support_type": "source_supported | model_supplement",
      "source_refs": ["content_card_id#locator"]
    }
  ],
  "review_cards": [
    {
      "question_type": "qa",
      "question": "string",
      "visible_before_submit": {},
      "hidden_until_submit": {
        "reference_answer": "string",
        "key_points": ["string"],
        "source_refs": ["content_card_id#locator"]
      }
    },
    {
      "question_type": "choice",
      "question": "string",
      "visible_before_submit": {
        "options": [
          {"id": "A", "text": "string"},
          {"id": "B", "text": "string"},
          {"id": "C", "text": "string"},
          {"id": "D", "text": "string"}
        ]
      },
      "hidden_until_submit": {
        "correct_option_ids": ["A"],
        "option_explanations": {"A": "string", "B": "string", "C": "string", "D": "string"},
        "source_refs": ["content_card_id#locator"]
      }
    }
  ],
  "open_questions": [
    {
      "question": "string",
      "hidden_until_submit": {
        "reference_points": ["string"],
        "source_refs": ["content_card_id#locator"]
      }
    }
  ],
  "summary": {
    "hidden_until_submit": {
      "text": "string",
      "source_refs": ["content_card_id#locator"]
    }
  }
}
```

## evaluate_review_answer

请求 `payload` 必须包含问答卡、隐藏答案，以及 `response_text` 或 `selected_option_ids`。

```json
{
  "verdict": "correct | partially_correct | incorrect | needs_human_check",
  "correct_parts": ["string"],
  "missing_parts": ["string"],
  "incorrect_parts": ["string"],
  "explanation": "string",
  "source_refs": ["content_card_id#locator"]
}
```

不得返回 FSRS 四档评分或复习日期。

## evaluate_deep_practice

```json
{
  "feedback": {
    "concept_accuracy": ["string"],
    "causal_links": ["string"],
    "evidence_use": ["string"],
    "transfer": ["string"],
    "boundaries": ["string"]
  },
  "revision_focus": ["string"],
  "source_refs": ["content_card_id#locator"]
}
```

## feynman_tutor

`payload` 包含用户解释、历史轮次和已识别缺口。每次只返回一个动作：

```json
{
  "action": "ask | diagnose | complete",
  "message": "string",
  "target_weakness_id": "string | null",
  "resolved_weaknesses": [],
  "unresolved_weaknesses": [
    {
      "id": "string",
      "type": "error | ambiguity | reasoning_gap | missing_link",
      "description": "string",
      "source_refs": ["content_card_id#locator"]
    }
  ]
}
```

## generate_supplemental_cards

请求 `payload` 必须包含 `user_confirmed: true` 和至少一个缺口。否则返回 `rejected`。

```json
{
  "draft": true,
  "weakness_ids": ["string"],
  "review_cards": [],
  "requires_user_review": true
}
```

`review_cards` 复用 `generate_materials` 中的问答式或选择题结构。
