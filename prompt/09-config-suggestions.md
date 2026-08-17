# 09 配置建议（bot_config + 插件配置）

- 用途：配合 prompt 文件套用时的配置侧建议。prompt 已删掉的"行为类"规则都由这些配置项承接。
- 依据：MaiBot 1.2.0 现行 `docker-config/mmc/bot_config.toml` 与 catsitate_core v0.3.0 插件配置。
- 原则：只给方向和必要改动，数值以实测为准——改完先私聊验证，再放群。

---

## 1. bot_config.toml 建议

| 配置项 | 当前值 | 建议 | 说明 |
|---|---|---|---|
| `[bot] nickname` | `"Catsitate-dev"` | 正式部署改回 `"Catsitate"` | dev 调试名，正式用去掉后缀 |
| `[personality] personality` | 空 | ← 01 | 人格叙事 |
| `[personality] behavior_style` | 空 | ← 02 | 行动准则（已含机制联动精简版 + 工具采用段；完整细则见 06） |
| `[personality] reply_style` | 空 | ← 03 | 表达风格 |
| `[personality] multiple_reply_style` | 已有旧版五条 | ← 05（替换） | 微调版五风格 |
| `[personality] multiple_probability` | `0.3` | 保持 | 已验证的概率 |
| `[chat.reply_style] group_chat_prompt` | 旧版 | ← 04.1（替换） | 删除频率数字版 |
| `[chat.reply_style] private_chat_prompts` | 旧版 | ← 04.2（替换） | 与旧版逐字相同，替换仅为保持一致 |
| `[experimental] emotion_trait` | `"sentimental"` | **保持**（你的决定） | 调和说明见下 |
| `[experimental] enable_behavior_learning` | `false` | 保持关闭（Q19） | 实验特性，经验注入可能违背人设底线 |
| `[chat] max_context_size` | `60/80` | 保持 | 128K 级模型，人设文本占用量可忽略 |
| `[a_memorix]` 全部 | 开启 | 保持 | 记忆不自知规则已在 06 第 10 节，无需动配置 |

### emotion_trait = sentimental 的调和说明

官方后缀会追加："你更敏感细腻，容易被聊天氛围触动，会自然表现出一点惆怅、共情或情绪波动，但不要过度煽情。"

- 与伪三无**不冲突**：01 写的是"表情很淡、语气很平"——表达层；sentimental 后缀管的是内心敏感层。二者叠加 = "容易被触动，但出口依然极淡"，恰是档案"内心的波澜很少抵达表面"。
- 若实测发现情绪外露变多（感叹号、长句安慰、直白情感词冒头）：**优先收紧 03 的表达约束，不要动档位**——01/03 的"不用感叹号、不长篇大论、说不出直白的话"是防线。

### reply_timing（频率唯一入口，prompt 已删频率数字）

| 配置项 | 当前值 | 建议 |
|---|---|---|
| `reply_trigger_mode` | `"reply_necessity"` | 保持。评分式触发，比 frequency 更贴"选择性回复"人设 |
| `talk_value` / `private_talk_value` | `1` | 起步保持。实测话多→调低；话太少→调高 |
| `mentioned_bot_reply` / `inevitable_at_reply` | `true` | 保持——"被@必回"由这两个开关保证 |
| `enable_talk_value_rules` | `false` | 可选开启：现有两条时段值（00:00-08:59→0.8、09:00-18:59→1）与"深夜字多、白天更安静"的人设吻合 |

## 2. catsitate_core 插件配置建议

| 配置项 | 当前值 | 建议 | 说明 |
|---|---|---|---|
| `bot_user_id` | 需确认 | **必须为 `"3545773341"`** | 否则 bot 自身发言识别失效，好感度计数错乱 |
| `level_rule_stranger/familiar/close/best_friend/special` | 默认文案 | ← 07 A 节改写版 | 显示在 `[好感度]` 注入块 + 进入旁路判定稳定段 |
| 结算参数（`window_hours=24`、`delta_max=5`、`daily_settle_min=3` 等） | 默认 | 保持 | 档案-中性参数，不建议动 |
| 衰减参数（`decay_after_days=7`、`decay_max=3`） | 默认 | 保持 | 同上 |
| 旁路模板 | 插件内置 | ← 07 B 节改写版 | 覆盖路径：`data/custom_prompts/zh-CN/catsitate_<id>.prompt` |

## 3. 不建议动的配置

- `[chat] mid_term_memory`、`[a_memorix]` 检索参数——记忆侧与人格无关，保持现状。
- `[expression]` / `[emoji]`——表情库与选择模式已在用，prompt 只要求"猫猫表情是舒适区"，不干涉技术参数。
- 插件 `poke.py`/`sleep.py`/`schedule.py` 的时间参数——已在旧平台验证过（00:30 睡/07:25 醒），与归档素材（08 第 3 节）一致，不动。
