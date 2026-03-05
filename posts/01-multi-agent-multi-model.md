# 同一个 AI，不同平台自动切模型，记忆还共享？

> 我让一个 AI agent 同时跑在 Discord 和 Telegram 上，各自用不同的模型，但它在两边都认识我、记得我说过的话。配置不到 20 行。

## 场景

我的 AI agent 同时跑在 Discord 和 Telegram 上。两个平台的使用场景很不一样——Discord 更多是日常闲聊和创意互动，Telegram 偏工作沟通和效率工具。

自然就有了一个需求：**不同平台用不同的模型**。

闲聊场景我想用一个更灵活、限制更少的模型，能聊得开、有个性。工作场景我想用一个更稳定、推理能力更强的模型，回答精准、不废话。

但问题是，我不想维护两套完全独立的 agent。它们应该共享同一份性格设定、同一份记忆——毕竟是同一个"人"，只是在不同场合表现不同而已。

## OpenClaw 怎么实现

OpenClaw 原生支持多 Agent 配置。核心思路是三步：

**第一步：定义两个 Agent，指向同一个 Workspace**

```json
"agents": {
  "list": [
    {
      "id": "discord-agent",
      "name": "我的 AI (Discord)",
      "default": true,
      "workspace": "~/.openclaw/workspace",
      "model": "provider-a/model-a"
    },
    {
      "id": "telegram-agent",
      "name": "我的 AI (Telegram)",
      "workspace": "~/.openclaw/workspace",
      "agentDir": "~/.openclaw/agents/telegram/agent",
      "model": "provider-b/model-b"
    }
  ]
}
```

关键点：
- 两个 agent 的 `workspace` 指向同一个目录——这意味着它们读的是同一份 `IDENTITY.md`、`SOUL.md`、`USER.md` 和记忆文件
- 但 `agentDir` 必须不同——这里存的是认证信息和会话状态，共用会冲突
- 每个 agent 可以指定自己的 `model`

**第二步：用 Bindings 做路由**

```json
"bindings": [
  {
    "agentId": "discord-agent",
    "match": { "channel": "discord" }
  },
  {
    "agentId": "telegram-agent",
    "match": { "channel": "telegram" }
  }
]
```

这段配置告诉 OpenClaw：Discord 来的消息交给 discord-agent 处理，Telegram 来的消息交给 telegram-agent 处理。简单粗暴。

**第三步：重启 Gateway**

改完配置，重启一下 gateway 就生效了。

## 效果

重启后，在 Discord 发一条消息，日志里可以看到走的是 Model A；切到 Telegram 发一条消息，走的是 Model B。

但无论从哪个平台聊，agent 都认识你、记得你之前说过什么（因为读的是同一份记忆文件），性格设定也是一致的。

就像一个人，在家和在公司说话方式不一样，但他还是同一个人。

## 几个注意事项

**agentDir 一定不能共用。** 文档里明确说了，共用 agentDir 会导致认证和会话冲突。workspace 可以共用，agentDir 不行。

**第二个 agent 需要有自己的认证。** 如果两个 agent 用的是不同的模型提供商，需要分别配置各自的认证信息。可以把主 agent 的 `auth-profiles.json` 复制一份到新 agent 的 agentDir 下作为起点。

**Bindings 是按顺序匹配的。** 第一个匹配到的规则生效。如果你有更复杂的路由需求（比如 Discord 某个频道用特定模型），可以用更细粒度的 match 条件，包括 guild ID、peer ID 等。

**模型的选择看场景。** 不是贵的就好。有些模型擅长创意和聊天，有些擅长逻辑和代码。根据你在每个平台的主要用途来选。

## 适合什么人

- 同时在多个平台使用 OpenClaw 的人
- 想给不同场景配置不同模型但不想管理多套配置的人
- 想省钱的人——闲聊用便宜的模型，正事用好的模型

---

*这是 OpenClaw Tips 的第一篇。后续会分享更多实战经验，包括语音消息配置、定时任务、模型调优等。关注这个仓库不迷路。*
