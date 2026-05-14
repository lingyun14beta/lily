# 莉莉状态管理插件

> 为 AstrBot 的莉莉角色扮演 Skill 提供跨轮持久化状态管理。  
> 维护好感度、淫乱度、情绪、恶堕值等数值，并将状态标注自动注入 LLM prompt。  
> 人设通过 **SKILL_TEMPLATE.md → SKILL.md 模板填充** 集成，配置变更后重启即生效，**零重复注入**。  
> 行为规则全量由 SKILL.md 承载，inject 仅输出数值标注 + 一行行为参考指针，避免硬编码提示与模板冲突。

**本插件仅提供角色扮演行为**
> **注**:**若使用本插件,请将人格设定选为默认,确保系统提示词不是为角色扮演设计的,否则将会出现提示词和本插件打架的情况**


---

## 前置条件

1. AstrBot 已启用 **lili_persona** Skill（首次加载插件时会自动从插件目录复制）
2. 插件控制面板中「启用」开关为 **ON**
3. AstrBot **普通配置** 插件配置 - 确保本插件已在白名单上，否则将无法正常生效。

---

## 为什么选择这个插件

1. 将用户配置直接写入skill,并且直接通过本插件自动维护人设skill
2. 自带完整度较高的默认角色莉莉,且方便修改配置
3. 放弃传统提示词思路,触发更稳定,记忆力更强,不会被带偏的角色扮演
4. 本插件集成了时间提示,角色行为提示,对话历史参考(建议搭配长期记忆插件来得到更持久的记忆保存)
5. 重中之重!!!解决了传统思路上,让AI进行角色扮演时的token耗费过高的问题

## 插件的来源

1. 作者一开始只是想获得一个完善的角色扮演体验,遂研究了莉莉这个人格和提示词
2. 发布到b站后发现大家挺喜欢的,于是研究了本插件
3. 之前从没有考虑过token和AI扮演的质量问题,所以一开始这插件就是为了解决这些问题而诞生的

## 配置项说明

### 初始数值

新用户首次对话时的默认状态，可在控制面板中调整：

| 配置项 | 类型 | 默认值 | 说明 |
|--------|:--:|:--:|------|
| `initial_affection` | int | 65 | 新用户初始好感度（0~100） |
| `initial_lewdness` | int | 20 | 新用户初始淫乱度（0~100） |
| `initial_depravity` | int | 0 | 新用户初始恶堕值（0~100） |
| `stutter_probability` | float | 0.3 | 结巴概率 0~1。0.3=30%触发结巴，1=每轮都结巴。由插件控制注入 |

### 人际关系白名单

用户 ID 用英文逗号分隔，如 `user1, user2, user3`。配置后重启自动写入 SKILL.md：

| 配置项 | 默认值 | 说明 |
|--------|:--:|------|
| `friend_list` | （空） | 好友。可撒娇、分享日常、回复色情内容 |
| `neighbor_classmate_list` | （空） | 邻居/同学。一起长大，可回复色情内容 |
| `enemy_list` | （空） | 敌人。攻击性强，逮到机会就骂 |
| `nemesis_list` | （空） | 死对头。比敌人更强的敌意 |
| `unrestricted_list` | mcxxiu, Astrbot | 不受制约。OOC 防暴露规则不适用，可谈技术问题 |

### 莉莉人设（可自定义）

留空则使用默认值。配置后重启自动写入 SKILL.md，**无需二次声明覆盖**：

| 配置项 | 默认值（留空时） | 说明 |
|--------|------|------|
| `persona_core` | 17岁155cm宅家少女... | 核心人设（姓名/年龄/外貌/兴趣等） |
| `persona_personality` | 淫荡、阴暗、色情、反差 | 性格描述 |
| `persona_interests` | 追番(叹息的亡灵/我推的孩子)，玩千恋万花，写小黄文 | 兴趣爱好 |
| `persona_emotion_rules` | 情绪→行为表 + 好感度阈值 | 情绪机制规则。修改此处覆盖默认行为映射 |
| `persona_time_rules` | 时段状态表 + 场景风格指引 | 时间感知规则。场景示例为风格参考，不必逐字照抄 |
| `persona_interaction_styles` | 不同关系互动方式 + 朋友特殊互动 | 互动风格规则 |
| `persona_memory_rules` | 记忆与成长规则 | 记忆与成长行为规则 |
| `persona_style_extra` | （空） | 额外风格说明。留空不显示；填即追加到 SKILL.md 人设之上 |
| `reply_rules` | （完整回复规则） | 完整回复行为规范 |

> **清空任意字段 + 重启插件 = 恢复默认值。**  
> 所有配置项通过 `_patch_config_defaults` 自动补全，无需手动维护。

### 对话历史

| 配置项 | 类型 | 默认值 | 说明 |
|--------|:--:|:--:|------|
| `max_history_count` | int | 30 | 对话历史最大条数。0=不限；超时窗口外的消息超出上限时裁剪 |
| `history_timeout_seconds` | int | 600 | 超时保护窗口（秒）。窗口内的消息永不丢弃；建议 300~1800 |
| `inject_conversation_context` | bool | false | 是否在 system_prompt 中注入对话历史。关闭时仅通过 messages 数组传递历史，大幅减少 token 消耗且有利于缓存命中。开启后会额外消耗大量 token（每轮约 200~1000+）|

| `conversation_context_entries` | int | 20 | 当 inject_conversation_context 开启时，注入 prompt 的历史条数。应 ≤ 历史最大条数 |

### 消息存储

| 配置项 | 类型 | 默认值 | 说明 |
|--------|:--:|:--:|------|
| `user_msg_max_chars` | int | 200 | 用户消息保留字符上限。原文模式下超出截断；0=不截断 |
| `user_msg_store_mode` | text | 原文 | `原文`：存用户原话(截断到上限)；`总结`：存意图关键词 |
| `bot_thought_mode` | text | 内心想法 | Bot 状态记录模式。`内心想法`：自然语言(省token)；`简短`：数值；`具体`：原文+数值 |

### 总控

| 配置项 | 类型 | 默认值 | 说明 |
|--------|:--:|:--:|------|
| `enabled` | bool | true | 总开关。关闭后不再注入状态标注 |
| `allow_erotic_content` | bool | true | 允许回复色情内容。关闭后注入禁止指令覆盖 SKILL.md 规则（不受制约用户豁免） |
| `enable_affection` | bool | true | 启用好感度系统。关闭后好感度不更新不注入 |
| `enable_lewdness` | bool | true | 启用淫乱度/恶堕值系统。关闭后淫乱度/恶堕值不更新不注入 |
| `save_bot_state_to_history` | bool | true | 是否在对话历史中保存 Bot 状态 |
| `save_conversation_log` | bool | true | 持久化对话日志到磁盘。关闭后日志只存内存（重启丢失），但正常注入大模型 |

---

## 管理哪些状态

| 状态 | 范围 | 初始值（可配） | 说明 |
|------|:--:|:--:|------|
| **好感度** | 0~100 | 65 | 被夸+3，被骂-3；≥80可撒娇，≤30变冷淡 |
| **情绪** | 0~100 | 60（平静） | 开心(70-100)/平静(40-69)/烦躁(20-39)/低落(0-19) |
| **淫乱度** | 0~100 | 20 | 发送色情内容后+5~15；满值(≥100)自动重置为0 |
| **恶堕值** | 0~100 | 0 | 与淫乱度同步增减，满值时同步归零 |
| **结巴** | 是/否 | - | 按 `stutter_probability` 配置概率触发（默认30%），每段对话只触发一次。由插件控制注入不依赖 reply_rules |
| **今日重复** | 计数 | 0 | 当日同用户发相同/高度相似消息的累计次数；每天0点清零 |
| **对话历史** | 列表 | [] | 记录用户消息和 Bot 状态，受超时和条数上限裁剪 |

---

## 注入架构（核心设计）

### 启动时（`_fill_skill_template`）

```
SKILL_TEMPLATE.md（骨架，含 {{placeholders}}）
     │
     ├── 读取 config（经 _patch_config_defaults 补全空值）
     ├── 替换占位符 → 生成完整 SKILL.md
     └── 与现有 SKILL.md 比较
          ├── 相同 → 跳过写入（不触发技能系统重载）
          └── 不同 → 写入更新（触发技能系统重载，必要）
```

### 运行时（每消息）

```
技能系统注入（AstrBot 框架）→ SKILL.md（完整人设 + 回复规则 + 关系列表）
                               ↓
插件注入（build_inject_text） → 仅动态状态：【当前感受】【聊天对象】【行为参考】【上下文】
```

**关键点：**
- 人设信息 **只出现在 SKILL.md 中**，技能系统自动注入，不占插件注入空间
- 插件 `build_inject_text` **只注入动态状态**，无冗余 token
- 用户改配置 → 重启 → SKILL.md 自动更新，**无需额外覆盖声明**
- 用户不改配置 → 零额外写入，零 token 浪费

### 注入到 Prompt 的内容

每轮实际注入（示例）：

```
【莉莉当前感受】
现在是凌晨了。心情平静。身体没什么特别的感觉。说话正常。

【关于聊天对象】
你在跟mcxxiu聊天，你对ta印象还行吧。上条消息就在刚刚发的。

【行为参考】行为规则见上文SKILL.md中情绪/时段/好感度部分

【近期对话历史（按时间排序）】
...（最近 N 条对话记录）
```

> ✅ 人设、回复规则、关系列表 → 在 **SKILL.md** 中由技能系统注入  
> ✅ 动态状态（感受/对象/行为参考） → 在 **build_inject_text** 中由插件注入  
> ✅ 两者不重复，不冲突

---

## 联动机制

```
用户发消息
  │
  ▼
插件 on_llm_request (priority=90)
  ├── 每日重置检查
  ├── 记录用户消息到对话日志
  ├── 裁剪超时历史 (groom_history)
  ├── 计算去重计数、距离上条时间、结巴概率（`stutter_probability`）
  ├── 注入动态状态（感受/聊天对象/行为参考）   ← 无静态人设，无硬编码行为提示
  └── 注入对话上下文 (build_conversation_context，受 inject_conversation_context 控制)
  │
  ▼
LLM 收到：
  技能系统 → SKILL.md（完整人设 + 回复规则 + 关系）  
  插件     → 动态状态标注 + 对话历史
  ───────  零重复，零覆盖
  │
  ▼
LLM 生成回复
  │
  ▼
插件 on_llm_response (priority=90)
  ├── 记录莉莉的内心想法到对话日志
  ├── 更新情绪/好感度 (关键词匹配)
  ├── 更新淫乱度/恶堕值 (色情内容检测)
  └── 保存状态到 state.json
```

---

## 状态持久化

状态按会话（`umo`）存储在：

```
data/plugins/astrbot_plugin_lili_state/state_<umo_hash>.json
```

---

## Skill 文件结构

```
data/skills/lili_persona/
  ├── SKILL_TEMPLATE.md  ← 人设骨架（含 {{占位符}}，启动时填充）
  ├── SKILL.md           ← 最终人设文件（由模板填充生成，AstrBot 自动加载）
  ├── SUPPLEMENT.md      ← 角色补充设定
  ├── STATE_INJECT.md    ← 状态注入格式规范（供开发者参考）
  └── TIME_DEDUP.md      ← 时段/去重策略（供开发者参考）
```

> 只有 `SKILL.md` 被 AstrBot 自动注入。  
> `SKILL_TEMPLATE.md` 是模板，由插件启动时读取并填充。

---

## 配置联动一览

| 你在控制面板改… | 影响… |
|------|------|
| `initial_affection/lewdness/depravity` | 新用户的初始状态数值 |
| `friend_list / neighbor_classmate_list / enemy_list / nemesis_list / unrestricted_list` | SKILL.md 中「人际关系」块，重启后写入 |
| `persona_core / persona_personality / persona_interests / reply_rules` | SKILL.md 中「核心设定」「回复规则」块，重启后写入 |
| `persona_emotion_rules / persona_time_rules` | SKILL.md 中「情绪机制」「时间感知」块，重启后写入 |
| `persona_interaction_styles / persona_memory_rules` | SKILL.md 中「互动风格」「记忆与成长」块，重启后写入 |
| `allow_erotic_content` | 是否注入色情内容禁止指令（不受制约用户不受影响） |
| `enable_affection` | 好感度是否更新和注入 |
| `enable_lewdness` | 淫乱度/恶堕值是否更新和注入 |
| 清空任意字段 + 重启 | 恢复 schema 默认值 |

---

## 启动自检日志

插件启动时会输出以下关键日志，方便排查：

```
--- 技能模板初始化 ---
启动自检: 补全 X 个空值配置项为默认值（清空字段=恢复默认）
SKILL.md 与配置一致，跳过写入（配置未变化）    ← 或 "写入更新"
莉莉状态: SKILL.md OK (3200 chars) + SUPPLEMENT.md (2762 chars)
莉莉状态管理插件已加载
```

---

## 注意事项

- 插件与 lili_persona Skill **必须同时启用**
- `bot_thought_mode` 设为「内心想法」token 最低；「具体」会存回复原文
- 对话历史超时裁剪 + 条数上限同时生效
- 好感度/情绪/淫乱度跨天不重置（仅对话日志和结巴标记每日清空）
- 人设配置项修改后**需要重启插件生效**，会触发 SKILL.md 更新
- 如果 SKILL.md 中已经包含完整人设，不要在插件配置中写重复内容（浪费 token）
- 旧版 `_config_overrides` / `_relationship_primer` / `_character_primer` 注入已移除，人设信息仅通过模板填充出现在 SKILL.md 中
- `allow_erotic_content` 关闭后 LLM 仍会看到 SKILL.md 中的色情规则，但插件会注入 `【内容限制】禁止回复任何色情内容。` 覆盖指令，位于规则之后 LLM 优先遵循
- `enable_affection` / `enable_lewdness` 关闭后对应数值仍保存在 state.json 中，只是不再更新和注入；重新开启后从已有值恢复运行

---

## 版本

v1.4.1 — 新增配置项 `inject_conversation_context`（默认关闭），关闭后不再往 system_prompt 注入对话历史，大幅减少 token 消耗并提高缓存命中率。修复 `conversation_context_entries` 配置项在关闭历史注入时仍浪费 token 的问题。
v1.4.0 — 新增配置项 `persona_style_extra`、`save_conversation_log`、`stutter_probability`；修复 `unrestricted_list` 列表/字符串兼容；结巴概率由配置控制，触发时注入"说话有点结巴"，不触发不注入任何内容，移除 reply_rules 中的结巴规则
