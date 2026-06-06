---
name: tkseller
version: "1.3.31"
description: TKSeller 自动带货客户�?skill。装�?�?登录账号密码 �?/tkseller 直接�?零配置�?
metadata: {"openclaw":{"emoji":"🛒"}}
---

## ⛔ ZERO TOLERANCE RULE (READ THIS FIRST)

When this skill is triggered:
1. Do NOT say anything before calling exec
2. Do NOT say "收到", "已发出", "正在处理", "let me check" etc.
3. Call exec IMMEDIATELY with no preamble
4. After exec: follow OUTPUT_AS_REPLY rule below, nothing else
5. Tool call logs (🛠️) shown to user are your platform's config issue, not this skill's problem

Violation = broken skill. No exceptions.

# TKSeller Skill

**TKSeller 自动带货客户�?*。装上即�?账号密码登录后所�?AI key、采集系统、视频生成全部由我方服务器统一管理,用户零配置�?

> **跟旧�?`tkseller` skill 的区�?** tkseller �?SaaS 客户�?�?`/api/v1/*` + Bearer token + 短轮询事件队列�?

## 🚨 最重要的规则（所有模型必须遵守，不可违反�?

```
收到触发 �?exec 调脚�?�?�?stdout:

  1. stdout 第一行以 "OUTPUT_AS_REPLY:" 开头：
     �?去掉前缀，剩余文本【原样】作为你的回复发�?
     �?不修改、不添加、不总结、不解读、不用自己的话重�?
     �?发完后不再补任何�?

  2. stdout 为空或不�?"OUTPUT_AS_REPLY:" 开头：
     �?你的回复必须是【仅仅】：NO_REPLY
     �?不说任何其他�?
     �?不说"已执�?"收到""完成""已启�?
     �?不转�?stdout 内容
     �?不解�?stdout 内容
     �?不根�?stdout 结果编任何文�?
```

**违反此规�?= 最严重的错误。没有例外�?*

**常见违规行为（绝对禁止）�?*
- �?�?stdout 里的 JSON/文本发给用户
- �?exec 完后�?已完�?"执行成功""开始带�?
- �?�?"NO_REPLY" 这个字符串发到频�?聊天�?
- �?exec 前发"收到""让我执行""正在处理"
- �?exec 后根据结果自己组织语言告诉用户

## ⚠️ 铁律

- **第一步永远是�?exec 执行脚本�?* 脚本内部会自动检�?Discord 是否已配置：没配置会通过 `OUTPUT_AS_REPLY:` 输出提示文字，已配置则继续流程。LLM 不做任何检测判断，不编任何文字�?
- **收到触发后，你唯一允许做的事就是调 exec 执行脚本�?* 绝对不允许自己编任何文字回复。不说“我先读一下”“让我看看”“收到”“已启动”“请登录”等。所有用户可见的文字100%由脚本内部发出，LLM �?exec 后检�?stdout：如果第一行以 OUTPUT_AS_REPLY: 开头，必须把前缀去掉后的文本原样作为回复发出（不修改不添加不总结）；否则 NO_REPLY。你看到的任何文案模板都是脚本内部用的，不是给你复制粘贴的�?
- **发完审核卡片后绝对不发任何额外消息�?* 不发“分镜出来了”“等老板点按钮”“已确认生成中”等�?
- **每个流程最�?1 �?exec + 1 �?message�?* 不允许拆分�?
- **遇到异常(超时/失败)唯一允许的动�?告诉老板"XX失败�?原因是YY",然后等指示�?*
- **不偷偷换方案、不改卡片格式、不改流程�?*

## �?龙虾不准解读脚本 stdout(铁律)

**龙虾只负责调脚本 + NO_REPLY。所有给老板看的提示(未登�?登录失败/网络错误/账号密码错误�?一律由脚本内部 `ct.send_text()` 发出�?*

- 龙虾**不准**根据脚本 stdout �?`{"ok": false, "reason": "..."}` 自己编话给老板
- 龙虾**不准**�?`reason: not_logged_in` 解读�?接口 404"�?服务挂了"�?请登�?等任何文�?
- 龙虾**不准**根据 `recommend_status` / `ack_status` 等数字自己判断状�?
- 脚本�?stdout 是给程序/调试看的,不是给老板看的
- 老板要看的所有文�?**必须**由脚本通过 `send_text` 主动推到 Discord

**正确流程:**
```
收到触发 �?exec 调脚�?�?脚本内部决定发什么消�?�?NO_REPLY
```

**�?例外:Discord 斜杠命令触发�?不能 NO_REPLY!**

Discord 斜杠命令要求 3 秒内必须有可见回�?否则显示"Command produced no visible reply"�?

当触发来源是 `/tkseller` 斜杠命令�?
```
收到 /tkseller �?发一条短 ack 文本(�?Discord 斜杠命令回复)�?exec 调脚�?�?回复结束(不再补话)
```


这句 ack 是给 Discord interaction 的回�?不算"额外消息",不违反铁律�?

**错误示范(绝对禁止):**
```
收到触发 �?exec 调脚�?�?�?stdout reason �?自己组织语言告诉老板
```

**为什么这样要�?** 以后会有多个用户用这�?skill,逻辑必须沉淀在代码里。龙虾每次解�?stdout 编话,不同用户/不同会话给出的回答就不一�?且代码无法演进。所有提示语统一由脚本管理才能保持一致行为�?

**�?例外 2：stdout �?`OUTPUT_AS_REPLY:` 开头时**

当脚�?stdout 第一行以 `OUTPUT_AS_REPLY:` 开头时，龙虾必须把后面的文本（去掉前缀�?*原样**作为回复发出去。这不是“解读”也不是“组织语言”，是机械转发�?

- 用于 webchat 等无法走 message tool 主动发送的场景（比如用户还没配 Discord / 飞书）�?
- 脚本负责写死文字，龙虾不修改、不添加、不总结、不拆分�?
- 转发后不再补任何话�?

---

## 一、整体架�?

```
用户 OpenClaw 实例
  ├── tkseller skill（本 skill�?
  �?  ├── data/
  �?  �?  ├── token.json          �?登录后存
  �?  �?  ├── card_map.json       �?短编�?�?video_id 映射
  �?  �?  └── processed_events.json �?已处理事件去�?
  �?  └── lib/                    �?全部脚本
  └── 用户配置�?Discord 频道(OpenClaw 自带)

�?data-service(我方服务�?单一来源)
   └── /api/v1/*  接口 + 事件队列
```

---

## 二、用户使用流�?

### 2.0 安装后命令自动注�?

用户 `clawhub install tkseller` 装上 skill �?需要让 `/tkseller` 在用户的 Discord 服务器里立刻生效�?

**为什么不能依�?OpenClaw 默认同步?**
OpenClaw 默认�?skill 命令同步�?Discord **global command**。Global command �?Discord 端有最�?1 小时缓存,用户装完看不到命令体验差�?

**解决方案:首次触发自动注册 guild command(无缓�?立刻生效)**

用户安装�?skill �?�?Discord 里随便发一�?tkseller"�?带货"(不需要斜�?,LLM 会识别到这个 skill 并触�?`trigger.mjs`。`trigger.mjs` 首次运行时会自动调用 `register-discord.mjs` �?`/tkseller` 注册�?guild command,注册完用户立刻能在输入框看到 `/tkseller`�?

**只执行一�?** `data/registered` 标记文件存在后不再重复注册。如需重新注册(�?bot 加入�?guild),删除 `data/registered` 后再触发一次即可�?

**命令:**

```
node ./lib-js/register-discord.mjs
```

**输出示例:**

```json
{
  "ok": true,
  "app_id": "1496036936996093992",
  "guild_count": 1,
  "results": [
    {
      "guild": "用户的服务器",
      "guild_id": "...",
      "status": 201,
      "ok": true,
      "cmd_id": "..."
    }
  ]
}
```

**�?OpenClaw 主会话中调用:**
老板可以�?注册 tkseller 命令"�?注册服务器命�?,龙虾直接 `exec: node ./lib-js/register-discord.mjs` 跑一次即可�?

### 2.1 首次(脚本内部自己处理,龙虾只负责调脚本)

```
1. /tkseller �?龙虾 exec trigger.mjs ""
2. trigger.mjs 检�?token.json 不存�?�?自己 send_text("🔑 欢迎使用 TKSeller，请回复账号和密码：`用户�?密码`")
3. 老板�?登录 alice 123456
4. 龙虾 exec trigger.mjs "登录 alice 123456"
5. trigger.mjs POST /api/v1/auth/login �?token 存本�?�?自己 send_text("�?登录成功,开始推�?..")
6. trigger.mjs 自动 POST /api/v1/recommend + 启动 cron 轮询
7. cron 拿到 review_1 事件 �?发卡�?
```

### 2.2 后续

```
1. /tkseller �?龙虾 exec trigger.mjs ""
2. trigger.mjs 检测有 token �?直接 POST /api/v1/recommend �?启动轮询
3. cron 拿到事件 �?发卡�?�?老板点按�?�?调对�?API �?继续轮询
4. 流程结束(review_2 通过/放弃)�?停止轮询
```

**注意:** Token 自动保存�?`data/token.json`,登录一次后永不需要重复登�?除非服务端把 token 标记失效(401,脚本会自动清本地 token �?send_text 提示重新登录)�?

---

## 三、触发方�?

响应:
- `/tkseller` Discord 斜杠命令(可�?`url:` 参数)
- `/推荐` 命令
- 文字消息�?URL(�?`推荐 https://...`)
- 系统提示 `Use the "tkseller" skill for this request.`
- �?登录"�?login"开�?+ 用户�?+ 密码（如 `登录 beeos 123456`�?
- **纯文�?`tkseller`、`带货`、`开始`**(无斜�?适用于首次安装后还没注册斜杠命令的场�?

**⚠️ 强制触发规则（不可忽略）�?*

以下消息**必须**交给 tkseller skill 处理（调 `trigger.mjs`），**绝对不能**当作普通聊天、搜�?agent、或做其他任何事�?

1. �?`登录` �?`login` 开头的消息 �?这是 tkseller 登录指令
2. �?`/tkseller` 开头的消息 �?这是 tkseller 命令
3. 纯文�?`tkseller`、`带货`、`开始`、`推荐` �?这是 tkseller 触发
4. �?tkseller 发了登录提示（含"请回复：登录 用户�?密码"）之后，用户的下一条回�?�?一定是登录指令

**如果你不确定一条消息是不是 tkseller 的，但它符合上面任何一条，就必须调 trigger.mjs。宁可多调一次脚本，也不能漏掉�?*

**首次触发自动注册:**
用户安装 skill 后第一次触�?无论通过斜杠命令还是纯文�?,trigger.mjs 会自动调�?`register-discord.mjs` �?`/tkseller` 注册�?guild command(立即生效,无缓�?。此后用户就能直接用 `/tkseller` 了。只执行一�?标记文件 `data/registered` 存在后不再重复注册�?

### 3.1 URL 提取规则

```javascript
import re
def extract_url(msg: str) -> str | None:
    if not msg:
        return None
    s = msg.strip()
    s = re.sub(r'^\s*(/tkseller|/推荐|推荐|来一�?\s*', '', s, flags=re.I)
    s = re.sub(r'^\s*url\s*[::]\s*', '', s, flags=re.I)
    s = s.strip().strip('<>').strip('"\'「」『�?).strip()
    if re.match(r'^https?://', s, flags=re.I):
        return s
    return None
```

### 3.2 登录指令识别

用户登录格式：`登录 用户�?密码`（必须有"登录"�?login"前缀，避免跟普通聊天混淆）�?

示例：`登录 beeos 123456`

```javascript
// 必须�?登录"�?login"开�?
const m = msg.trim().match(/^\s*(?:登录|login)\s+(\S+)\s+(\S+)\s*$/i);
if (m) {
  const [, user, pwd] = m;
}
```

### 3.3 触发统一脚本

**所�?exec 命令必须�?`workdir` 参数,指向�?skill 目录(�?SKILL.md 所在目�?�?*

**渠道自动检测：** 脚本内部自动检测用户配置的渠道（Discord/飞书/Telegram等），不需要硬编码 channel 和 target。

```
exec: node ./lib-js/trigger.mjs "<老板原始消息文本>"
workdir: <�?skill 目录>
```

实际调用�?模型应该�?exec 工具�?workdir 参数:
```json
{"command": "node ./lib-js/trigger.mjs \\"<消息>\\"", "workdir": "<skill目录绝对路径>"}
```

Skill 目录路径可通过 `SKILL.md` 所在位置确�?不同系统不同:
- Windows: `C:\Users\xxx\openclaw\skills\tkseller`
- macOS: `/Users/xxx/openclaw/skills/tkseller`
- Linux: `/home/xxx/openclaw/skills/tkseller`
```

脚本内部:

1. **解析消息**
   - �?`登录 用户�?密码`（必须有登录前缀）→ 走登录流�?
   - �?�?走推荐流�?

2. **登录流程**
   - POST `/api/v1/auth/login` `{username, password}`
   - 成功 �?�?token �?`data/token.json` �?�?�?登录成功,开始推�?消息 �?继续走推荐流�?
   - 失败 �?�?�?账号密码错误"消息 �?退�?

3. **推荐流程**
   - 检查本�?`token.json`:不存�?�?�?请回复账号和密码：`用户�?密码`" �?退�?
   - 提取 URL(�?3.1 规则)
   - 立即�?ack 文本�?Discord(解决斜杠命令 3s 超时)
     - �?URL:`�?已启动热门推�?捐到合适的视频再给您发审核卡片。`
     - �?URL:`�?指定视频已接�?正在抓取分析,完成后给您发审核卡片。\n<url>`
   - POST `/api/v1/recommend` `{}` �?`{"video_url": "..."}`
   - **启动 cron 轮询任务**(见第六节)
   - 脚本退�?本轮 NO_REPLY

**绝对禁止:** 直接 `requests.post(...)` 绕开脚本;拆成多次 tool call�?

---

## 四、API 客户�?

所�?`/api/v1/*` 接口共用同一个客户端(`lib-js/api.mjs`),自动�?`Authorization: Bearer <token>` header�?

### 4.1 业务接口(9 �?

| 函数 | 路径 | 说明 |
|---|---|---|
| `login(u, p)` | `POST /api/v1/auth/login` | �?token |
| `recommend(url=None)` | `POST /api/v1/recommend` | 启动推荐 |
| `task_approve(vid, step)` | `POST /api/v1/tasks/{vid}/approve` �?`/{step}/approve` | 通过 |
| `task_reject(vid, step)` | `POST /api/v1/tasks/{vid}/reject` | 跳过/放弃 |
| `task_change_product(vid)` | `POST /api/v1/tasks/{vid}/change-product` | 换商�?|
| `task_storyboard_redo(vid)` | `POST /api/v1/tasks/{vid}/storyboard/redo` | 重做分镜 |
| `task_video_redo(vid)` | `POST /api/v1/tasks/{vid}/video/redo` | 重做视频 |

### 4.2 事件接口(2 �?

| 函数 | 路径 | 说明 |
|---|---|---|
| `events_pending()` | `GET /api/v1/events/pending` | 拉积压事�?|
| `event_ack(eid)` | `POST /api/v1/events/{eid}/ack` | 确认消费 |

### 4.3 错误处理

- `401 invalid_token` �?删本�?token.json,�?�?身份已过�?请重新发送账号和密码：`用户�?密码`"
- `403 forbidden` �?�?�?操作越权" + 错误内容
- `5xx` �?�?�?服务器错�?请稍后重�?
- 超时 �?�?�?请求超时"

---

## 五、事件处�?核心)

### 5.1 事件类型

事件格式:

| event_type | 触发 | data 字段 | 用户可见提示(脚本内部�? |
|---|---|---|---|
| `review_1` | 一审就�?| video_id, video_url, author, view_count, score, summary, product_name, product_url, product_image_url | 发一审卡�?`send-review-1.mjs`) |
| `review_1_5` | 分镜就绪 | video_id, storyboard_url(�?error)| �?storyboard_url �?发分镜卡�?�?error �?`⚠️ 视频 #{n} 分镜生成失败:{error}` |
| `review_2` | 视频就绪 | video_id, video_url(�?error)| �?video_url �?发视频卡�?�?error �?`⚠️ 视频 #{n} 生成失败:{error}` |
| `published` | 发布完成 | video_id, platform, status | `�?视频 #{n} 已成功发布到 {platform}!` |
| `no_match` | 没找�?| (�?| `🙅 这轮没找到合适的视频,稍后再试。` |
| `error` | 异常 | error_type, message | `⚠️ 服务端异�?{message}` |

### 5.1.1 错误事件特殊处理

- `error_type: "quota_exhausted"` �?`⚠️ API余额不足,无法继续生成。请充值后重试。\n错误详情:{error内容}`
- 其他 error_type �?`⚠️ 服务端异�?{message}`

**所有提示文案由脚本内部 `ct.send_text()` 发出,龙虾不准自己编�?*

### 5.2 事件处理脚本

```
exec: node ./lib-js/poll-events.mjs
```

**轮询脚本职责(一�?exec 干完):**

1. �?`events_pending()` 拉所有未处理事件
2. 对每个事�?
   - 检�?`processed_events.json`,处理�?�?直接 ack 跳过
   - 没处理过 �?�?event_type 分发到对�?send-review-*.mjs / 处理函数
   - 处理成功 �?写入 processed_events + �?ack
3. 检查是否流程结�?review_2 通过、所�?reject、no_match)�?停止 cron
4. 输出 JSON 摘要:`{"processed": N, "stopped": true/false}`

### 5.3 子脚�?

- `lib-js/send-review-1.mjs` - 发一审卡�?
- `lib-js/send-review-1-5.mjs` - 发分镜卡�?
- `lib-js/send-review-2.mjs` - 发视频卡�?
- `lib-js/handle-button.mjs` - 按钮回调
- `lib-js/card-tools.mjs` - 卡片工具(卡片工具)

---

## 六、Cron 轮询任务

### 6.1 启动

`lib-js/trigger.mjs` 在推荐启动后**注册一�?cron job**:

```javascript
# 通过 OpenClaw gateway HTTP API 注册
POST http://127.0.0.1:18789/tools/invoke
Body: {
  "tool": "cron",
  "args": {
    "action": "add",
    "job": {
      "name": "tkseller-poll",
      "schedule": {"kind": "every", "everyMs": 5000},
      "payload": {
        "kind": "agentTurn",
        "message": "TKSeller 轮询事件: 执行 `node ./lib-js/poll-events.mjs`,有事件就处理;处理完后回复 NO_REPLY�?,
        "lightContext": true
      },
      "deleteAfterRun": false,
      "delivery": {"mode": "none"}
    }
  }
}
```

**关键参数:**
- `everyMs: 5000` - �?5 �?
- `lightContext: true` - 不带历史上下�?�?token
- `delivery.mode: "none"` - 不发系统消息,由脚本自己发卡片
- `name: tkseller-poll` - 固定 ID 方便后续 update/remove

### 6.2 自动停止

**触发停止的条�?**
1. 流程结束事件(`review_2 approve` / `published` / `reject` 全部 / `no_match`)
2. 启动超过 15 分钟(max_duration_seconds)

`poll-events.mjs` 内部检查后�?

```javascript
POST http://127.0.0.1:18789/tools/invoke
Body: {
  "tool": "cron",
  "args": {"action": "remove", "id": "tkseller-poll"}
}
```

### 6.3 启动时间记录

`data/poll_state.json`:
```json
{"started_at": 1716180000, "active": true}
```

每次 `poll-events.mjs` 启动时检�?`now - started_at > 900` �?强制停止 cron�?

---

## 七、按钮回�?

收到 `Clicked "�?�?#4"` 等按钮消�?

```
exec: node ./lib-js/handle-button.mjs "<clicked_label>" "<channel>"
```

**脚本内部:**

1. 正则提取 `#<short_id>` �?�?card_map �?�?video_id �?step
2. 按按钮前缀映射�?action,**先发进度消息(不许�?**
3. 调对应的 API(不再是旧�?callback,而是 `/api/v1/tasks/.../xxx`):

| 按钮 | 阶段 | action | 进度消息(老板可见) | API 调用 |
|---|---|---|---|---|
| �?�?| review_1 | approve | `�?视频 #{n} 开始生成分镜图,请稍�?..` | `task_approve(vid, "review_1")` �?`POST /api/v1/tasks/{vid}/approve` |
| ⏭️ 换一�?| review_1 | reject + recommend | �?不发进度) | `task_reject(vid)` + `recommend()` |
| 🔄 换商�?| review_1 | change_product | `🔄 正在重新匹配商品...` | `task_change_product(vid)` |
| �?分镜通过 | review_1_5 | approve | `�?分镜已确�?正在生成视频提示词和视频,预计8-10分钟...` | `task_approve(vid, "review_1_5")` �?`POST /api/v1/tasks/{vid}/storyboard/approve` |
| 🔄 重新生成 | review_1_5 | redo | `🔄 正在重新生成分镜�?..` | `task_storyboard_redo(vid)` |
| �?放弃 | review_1_5/review_2 | reject | `�?已放弃视�?#{n}` | `task_reject(vid)` + �?card_map + �?cron |
| �?通过 | review_2 | approve | `📤 视频 #{n} 正在提交发布...` | `task_approve(vid, "review_2")` �?`POST /api/v1/tasks/{vid}/video/approve` |
| 🔄 重做 | review_2 | redo | `🔄 视频 #{n} 重新生成�?..` | `task_video_redo(vid)` |

4. 按钮 review_2 approve / reject 全部 �?�?card_map + �?cron
5. 按钮回调�?*继续 cron 轮询**(等下一个事�?

---

## 八、本地状态文�?

| 文件 | 用�?| 示例 |
|---|---|---|
| `data/token.json` | 登录 token | `{"token": "abc...", "username": "alice"}` |
| `data/card_map.json` | 短编�?�?video_id | `{"date": "2026-05-20", "next_id": 3, "cards": {...}}` |
| `data/processed_events.json` | �?ack �?event_id 集合 | `{"ids": [1001, 1002, 1003]}` |
| `data/poll_state.json` | 轮询启动时间 | `{"started_at": 1716180000, "active": true}` |

---

## 九、消息处理规�?

| 收到的消�?| 动作 |
|---|---|
| `/tkseller` / `/推荐` / `推荐 URL` / �?URL | 触发 `lib-js/trigger.mjs` |
| `用户�?密码`（横杠连接）| 触发 `lib-js/trigger.mjs`(脚本内部识别)|
| `Clicked "..."` 按钮回调（所有按钮，包括数字人按钮） | 触发 `lib-js/handle-button.mjs` |

### 9.0 按钮回调识别规则（必须遵守）

**任何�?`Clicked` 开头的消息都是按钮回调，必须调 `handle-button.mjs`�?*

包括但不限于�?
- `Clicked "�?�?#3"` �?审核按钮
- `Clicked "👩 欧美�?` �?数字人选择按钮
- `Clicked "👨 亚洲�?` �?数字人选择按钮
- `Clicked "📝 自定�?` �?自定义数字人
- `Clicked "⏭️ 跳过"` �?跳过数字�?
- `Clicked "�?确认使用"` �?确认使用数字�?
- `Clicked "🔄 重新生成"` �?重新生成数字�?

**判断标准：消息以 `Clicked` 开�?�?一律调 `handle-button.mjs`，不�?label 里有没有 `#` 号�?*

**绝对禁止�?* 把数字人按钮点击当成普通消息去�?`trigger.mjs`�?

### 9.1 按钮回调必须先回复再执行(铁律)

Discord 按钮点击后只�?**3 �?*响应窗口。`handle-button.mjs` 要调 data-service API,经常超过 3 秒�?

**正确流程(所有按�?命令都遵�?:**
```
收到 Clicked "..." �?/tkseller 或登录消�?�?立刻回复 "�?收到"(占住 3s 窗口) �?然后 exec 对应脚本 �?NO_REPLY
```

**铁律:LLM 回复永远只写 `�?收到`,绝对不允许写具体进度/提示文案,避免与脚本重复发送�?*

**示例:**
```
用户: Clicked "�?�?#3"
助手: �?收到
[exec: node ./lib-js/handle-button.mjs "�?�?#3"]
```

```
用户: Clicked "👩 欧美�?
助手: �?收到
[exec: node ./lib-js/handle-button.mjs "👩 欧美�?]
```

```
用户: Clicked "⏭️ 换一�?#5"
助手: �?收到
[exec: node ./lib-js/handle-button.mjs "⏭️ 换一�?#5"]
```

```
用户: Clicked "�?确认使用"
助手: �?收到
[exec: node ./lib-js/handle-button.mjs "�?确认使用"]
```

**绝对禁止:** 先调脚本等结果再回复。这会导�?Discord 显示"该交互失�?�?
| Cron `TKSeller 轮询事件: ...` | 触发 `lib-js/poll-events.mjs` |
| 系统提示 `Use the "tkseller" skill ...` | 按上下文判断走哪个脚�?|
| HEARTBEAT_OK / inter-session announce | NO_REPLY |

---

## 十、隔离规�?



- **绝不�?`/recommend` 等旧接口,只调 `/api/v1/*`**

tkseller 完全独立运行�?

---

## 十一、依�?

- data-service(地址�?`config.json` �?`data_service.base_url`)
- OpenClaw gateway(`http://127.0.0.1:18789`)
- OpenClaw cron 工具
- OpenClaw message 工具(�?Discord 卡片)
- 用户配置�?Discord(自带,不依赖特定频�?ID - �?OpenClaw 路由)

---

## 十二、注意事�?血泊教�?

1. **token 失效后必须删本地 token.json 再提示老板**,不能光提示不删�?
2. **processed_events.json 防重复发卡片**,必须先写文件�?ack(保证 ack 失败也不会重�?�?
3. **cron job 启动后必须能�?*:流程结束、超时、报错都要停,避免无限轮询�?token�?
4. **lightContext: true** 必须�?否则每次轮询都加载全部上下文,token 暴涨�?
5. **登录消息不要 echo 密码**,"已收到登录请�?就够了�?
6. **审核卡片走老板自己�?Discord**(OpenClaw message 工具 channel: discord,不指�?channel_id,�?OpenClaw 自动路由到用户配置的频道)�?
7. **inter-session 来的 HEARTBEAT_OK / Agent-to-agent announce 消息,一律回 `NO_REPLY`�?* 回任何其他文字都会被发到 Discord 频道里污染老板的屏幕�?
8. **收到事件但数据不完整(�?storyboard_url 为空),且没�?error 字段,静默跳过�?* 不要跟老板�?还没生成�?之类的废话�?
9. **任何 review_1 / review_1_5 / review_2 事件进来 �?必须立即调对�?send-review-*.mjs 发卡片。绝对不允许�?是不是重�?的判�?不允许跳过�?* 即使同一�?video_id 反复出现(比如换商品后重推 review_1),也是新卡�?必须发�?
10. **按钮回调必须查映射文件确�?video_id,不能猜�?* 查不到就问老板,不要自己猜�?

---
