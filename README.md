# grokbot_skill

Penguin lee 的 Grok Bot 用户 skill 仓库。每周一例程同步；也可按需即时推送。

## Skills

- `grokbot-feishu-bridge` — 把 Grok Bot 接到飞书群聊（共享 CLI、OAuth、webhook 例程、本地转发；适合每 bot 一个私有群）。
- `grokbot-feishu-per-app` — 每个 Grok Bot 独立飞书应用（扫码建应用、webhook、Python WS 转发；适合同群多 bot @ 协作）。
- `freetoken-bots` — 发现、验证并维护 OpenRouter 上的零价模型，供 Pi 子 agent（Muse、Grokbot、Dot）使用。改编自 [limin112/min-skill](https://github.com/limin112/min-skill/tree/main/skills/FreeToken-Bots)（v0.2.0），保留来源说明。

密钥、租户 ID、webhook URL 等不要写进 skill 文件。
