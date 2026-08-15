# SIKO AI 网关机器人（中文版）

一个部署在 **Cloudflare Workers + D1 数据库** 上的多供应商 AI 聊天网关 Telegram 机器人。管理员接入多个 AI 供应商（OpenAI 兼容接口 / Google Gemini），用户即可在 Telegram 里像用 ChatGPT 一样对话，具备额度、权限、人格、记忆、图片理解等完整运营能力。

> 由 **培哥** 制作并维护。

## 功能特性

- **多供应商接入**：支持 OpenAI 兼容接口与 Google Gemini 两类，添加时自动拉取模型列表（Groq / OpenRouter / DeepSeek / Mistral / OpenAI / NVIDIA / Cohere / Cerebras 等）。
- **AI 聊天**：文字对话 + 图片理解（多模态），带进度动画，长文本自动分段发送。
- **对话记忆**：每人保留最近 10 条上下文，可一键清空；单条内容超长自动截断，避免撑爆模型上下文。
- **人格系统**：管理员定义人格（系统提示词），用户可自由切换。
- **额度制**：每次对话扣 1 额度，定时任务每日自动重置。
- **用户管理**：查看、封禁/解封、增加额度、指定专属模型。
- **管理面板**：仪表盘、供应商增删改、默认模型、群发、私有/公开切换。

## 前置条件

- 一个 Telegram Bot（找 [@BotFather](https://t.me/BotFather) 创建，获取 Token）。
- 一个 Cloudflare 账号（Workers + D1）。
- 至少一个 AI 供应商的 API Key（见 [PROVIDERS.md](PROVIDERS.md)）。

## 环境变量 / 绑定

| 变量 | 必填 | 说明 |
|------|------|------|
| `BOT_TOKEN` | ✅ | BotFather 的机器人 Token |
| `ADMIN_ID` | ✅ | 管理员的 Telegram 数字 ID |
| `SETUP_SECRET` | 推荐 | 初始化 `/setup` 的强密钥；不设则回退用 `ADMIN_ID`（较弱） |
| `DB` (D1 binding) | ✅ | 绑定的 D1 数据库 |

> **安全建议**：供应商 API Key 会存入 D1（面板显示时做掩码）。对安全要求高的场景，建议改用 Cloudflare Secrets 管理敏感密钥。

## 部署步骤

1. 在 Cloudflare 新建 Worker，粘贴 `index.js`。
2. 创建并绑定一个 **D1 数据库**，绑定名为 `DB`。
3. 配置环境变量 `BOT_TOKEN`、`ADMIN_ID`、（推荐）`SETUP_SECRET`。
4. 部署后，浏览器访问初始化地址（建表 + 注册 Webhook）：
   ```
   https://<你的Worker域名>/setup?key=<SETUP_SECRET 或 ADMIN_ID>
   ```
5. 在 Telegram 中向机器人发送 `/start`，管理员用 `/admin` 打开管理面板，先添加一个供应商并选择默认模型。

## 使用方法

- 普通用户：`/start` → 直接发消息或图片即可对话；可切换人格、清空记忆、查看额度。
- 管理员：`/admin` → 添加供应商 → 选择默认模型 → 管理用户/人格/群发。

## 优点

- **Serverless 架构**：免运维、可扩展，D1 持久化数据。
- **供应商全动态**：不改代码即可接入新的兼容接口与模型。
- **容错完善**：Telegram 429 限流自动重试，数据库操作异常降级。
- **功能接近可商用**：额度、封禁、人格、记忆、群发、多模态齐全。

## 已修复的问题

- **UI 全中文化**：菜单、提示、报错、系统提示词、`/setup` 页面。
- **editMessageText 键盘 Bug**：原代码用 `editMessageText` 携带底部 reply 键盘（Telegram 不支持），已改为编辑失败时补发带菜单的消息。
- **长文本分段**：仅在最后一段附带键盘，避免每段刷屏。
- **初始化强密钥**：新增 `SETUP_SECRET`，避免用可猜的 `ADMIN_ID` 作为唯一凭证。
- **图片体积保护**：超过 5MB 的图片拒绝处理，避免 Workers 内存/请求体超限。
- **记忆长度保护**：单条记忆超 4000 字符自动截断，避免超出模型上下文。

## 已知限制与建议

- **假流式**：进度条为定时编辑动画，非真正 SSE 流式输出。
- **API Key 存 D1**：面板已掩码，但底层为明文，敏感场景建议用 Secrets。
- **记忆按条数截断**：未按 token 精确计算，极端超长历史仍可能触及模型上限。

## 供应商参考

常见供应商的 API Key 获取地址与 Base URL 见 [PROVIDERS.md](PROVIDERS.md)。

## 联系方式

- 频道：[@pgkj666](https://t.me/pgkj666)
- 联系机器人：[@pgkj666_bot](https://t.me/pgkj666_bot)

## 许可证

[MIT](LICENSE) © 2026 培哥
