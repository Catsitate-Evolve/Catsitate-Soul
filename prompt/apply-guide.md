# 套用说明（apply-guide）

## 1. 文件 → 配置位置映射表

| 交付文件 | 目标位置 | 套用动作 |
|---|---|---|
| 01-persona.md | `bot_config.toml` → `[personality]` → `personality` | 整段粘贴成品文本 |
| 02-behavior-style.md | `[personality]` → `behavior_style` | 整段粘贴；再把 06 成品全文追加在其后 |
| 03-reply-style.md | `[personality]` → `reply_style` | 整段粘贴 |
| 04-chat-prompts.md | `[chat.reply_style]` → `group_chat_prompt` / `private_chat_prompts` | 两段分别粘贴 |
| 05-multiple-styles.md | `[personality]` → `multiple_reply_style` | 数组逐条替换；`multiple_probability = 0.3` 保持 |
| 06-mechanism.md | 追加到 `behavior_style` 末尾（推荐） | 见下方注意事项 |
| 07-plugin-prompts.md | A 节→插件配置 `level_rule_*`；B 节→`data/custom_prompts/zh-CN/catsitate_<id>.prompt` | 逐条替换 / 建 8 个覆盖文件 |
| 08-extra-prompts.md | 无（归档） | 未来功能启用时查阅 |
| 09-config-suggestions.md | 无（建议） | 套用时对照执行 |
| prompt-final.md | 组装成品 | 可整段复制 `[personality]` + `[chat.reply_style]` 两段 |

## 2. 三种套用方式

### 方式一：WebUI（推荐）
1. 打开 `http://localhost:18001` → 对应配置页；
2. 按映射表逐项粘贴；`multiple_reply_style` 是数组，逐条填入；
3. 旁路模板走 WebUI 的提示词管理（对应 `catsitate_*` 条目）或方式三。

### 方式二：直接改文件
1. 编辑 `MaiBot-dev/docker-config/mmc/bot_config.toml`；
2. 改前备份：`cp bot_config.toml old/bot_config_$(date +%Y%m%d_%H%M%S).toml`（沿用项目 `old/` 备份习惯）；
3. 重启核心容器生效：`docker compose restart maim-bot-core`。

### 方式三：custom_prompts 覆盖（旁路模板专用）
- 把 07 B 节的 8 个改写版分别存为 `MaiBot-dev/data/custom_prompts/zh-CN/catsitate_<id>.prompt`；
- 加载顺序：`data/custom_prompts/zh-CN/…` → `prompts/zh-CN/…` → 插件内置默认，无需改插件本体；
- 不改的 4 个（msg_react/sentinel/sleep_confirm 原样、image_relook 轻改）可只建需要覆盖的 4 个文件：`catsitate_favorability`、`catsitate_decay`、`catsitate_schedule_generate`、`catsitate_sleep_review`（image_relook 若要风格约束也建）。

## 3. 注意事项

1. **前缀缓存纪律**：旁路模板的 system 文本属稳定段——本次改写的 favorability/decay/schedule_generate/sleep_review 都动了稳定段，部署后**首次调用会重建缓存**（插件按块字节级复用），属正常现象，不必处理。
2. **06 追加位置**：`behavior_style` 只进 Planner。把 06 接在其后，Planner 决策时能"感知状态并遵守系统边界"；06 的总则/边界也建议复制一份到 `chat_prompts`（若你担心 Replyer 生成时不受约束，Replyer 侧由 03 的"不说系统词汇"性质底线 + 04 的硬底线兜底）。
3. **验证顺序**：先私聊（你本人，3341299096）→ 检查：极简句式、「…」频率、不问不问、被戳反应、报日程自然度、好感度等级态度 → 再放群聊观察：选择性回复、贴表情时机、@必回。
4. **出问题时的回滚**：bot_config 直接换回 `old/` 备份重启即可；旁路模板删掉 custom_prompts 对应文件即回插件默认。
5. **昵称**：确认正式部署时 `nickname = "Catsitate"`（当前 dev 为 `Catsitate-dev`），否则角色自称带"-dev"后缀。
