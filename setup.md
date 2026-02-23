# 步骤一
安装openclaw：
npm install -g openclaw@latest

# 步骤二
配置openclaw
openclaw onboard --install-daemon
；
注意点：base_url要带上v1，如
https://www.dmxapi.cn/v1
也可以后面去改这个配置文件：vim ~/.openclaw/openclaw.json

# 步骤三
本地测试
openclaw tui进入会话窗口、测试本地能否调用模型

# 步骤四
接入飞书机器人bot
openclaw channels add
配置完成后：
```bash
openclaw gateway status      # 查看网关运行状态
openclaw gateway restart     # 重启网关以应用新配置
openclaw logs --follow       # 查看实时日志
```text

然后：
### 9.1.3 第一步：创建飞书应用

#### 1. 打开飞书开放平台

访问 [飞书开放平台](https://open.feishu.cn/app)，使用飞书账号登录。

> 💡 **Lark（国际版）**：请使用 https://open.larksuite.com/app，并在配置中设置 `domain: "lark"`。

#### 2. 创建应用

1. 点击 **创建企业自建应用**
2. 填写应用名称和描述
3. 选择应用图标

![飞书开放平台 - 创建企业自建应用](https://upload.maynor1024.live/file/1770734336224_image_1770734318.jpg)

#### 3. 获取应用凭证

在应用的 **凭证与基础信息** 页面，复制：
- **App ID**（格式如 `cli_xxx`）
- **App Secret**

❗ **重要**：请妥善保管 App Secret，不要分享给他人。

![飞书应用凭证 - App ID和App Secret](https://upload.maynor1024.live/file/1770734332380_image_1770734319.jpg)

#### 4. 配置应用权限

在 **权限管理** 页面，点击 **批量导入** 按钮，粘贴以下 JSON 配置一键导入所需权限：

```json
{
  "scopes": {
    "tenant": [
      "aily:file:read",
      "aily:file:write",
      "application:application.app_message_stats.overview:readonly",
      "application:application:self_manage",
      "application:bot.menu:write",
      "cardkit:card:write",
      "contact:user.employee_id:readonly",
      "corehr:file:download",
      "docs:document.content:read",
      "event:ip_list",
      "im:chat",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.members:bot_access",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.group_msg",
      "im:message.p2p_msg:readonly",
      "im:message:readonly",
      "im:message:send_as_bot",
      "im:resource",
      "sheets:spreadsheet",
      "wiki:wiki:readonly"
    ],
    "user": [
      "aily:file:read",
      "aily:file:write",
      "im:chat.access_event.bot_p2p_chat:read"
    ]
  }
}
```text
![飞书应用权限配置 - 批量导入JSON权限](https://upload.maynor1024.live/file/1770734343156_image_1770734320.jpg)

#### 5. 启用机器人能力

在 **应用能力** > **机器人** 页面：
1. 开启机器人能力
2. 配置机器人名称

![飞书机器人配置 - 启用机器人功能](https://upload.maynor1024.live/file/1770734349201_image_1770734321.jpg)

#### 6. 配置事件订阅

⚠️ **重要提醒**：在配置事件订阅前，请务必确保已完成以下步骤：
1. 运行 `openclaw channels add` 添加了 Feishu 渠道，并且已经添加了应用的appid和secret。
2. 网关处于启动状态（可通过 `openclaw gateway status` 检查状态）
3. 启动网关

```bash
# 安装并启动网关
openclaw gateway install

# 检查网关状态
openclaw gateway status

# 查看实时日志
openclaw logs --follow
```text

在 **事件订阅** 页面：

**步骤1：选择长连接模式**
1. 选择 **使用长连接接收事件**（WebSocket 模式）

**步骤2：添加事件**
2. 添加事件：`im.message.receive_v1`（接收消息）

**步骤3：配置必需权限（重要）**

在配置事件订阅的同时，请确保在 **权限管理** 页面已添加以下权限：

| 权限标识 | 权限名称 | 是否必需 | 说明 |
|---------|---------|---------|------|
| `im:message` | 获取与发送单聊、群组消息 | ✅ 必需 | 接收和发送消息 |
| `im:message:send_as_bot` | 以应用身份发消息 | ✅ 必需 | 以机器人身份回复 |
| `contact:contact.base:readonly` | 获取通讯录基本信息 | ✅ 必需 | 识别用户身份 |

> 💡 **为什么需要 `contact:contact.base:readonly` 权限？**
> 
> 这个权限用于获取用户的基本信息（如用户名、部门等），OpenClaw需要这些信息来：
> - ✅ 识别消息发送者
> - ✅ 实现访问控制（allowlist/denylist）
> - ✅ 提供个性化服务
> - ✅ 记录对话历史
> 
> ⚠️ **如果缺少此权限，机器人将无法正常响应消息！**

**配置截图示例**：

![飞书权限配置 - 通讯录权限](https://upload.maynor1024.live/file/1771065454975_image-20260214183727712.png)

⚠️ **注意**：如果网关未启动或渠道未添加，长连接设置将保存失败。

![飞书事件订阅 - 使用长连接接收消息](https://upload.maynor1024.live/file/1770734352151_image_1770734322.jpg)

**常见错误排查：**

如果遇到 "Gateway start blocked: set gateway.mode=local" 错误：
```bash
# 确保配置文件中设置了 gateway.mode
{
  "gateway": {
    "mode": "local"
  }
}
```text
如果遇到 "Gateway auth is set to token, but no token is configured" 错误：
```bash
# 方式1：在配置文件中设置 token
{
  "gateway": {
    "auth": {
      "mode": "token",
      "token": "your-secure-token"
    }
  }
}

# 方式2：使用环境变量
export OPENCLAW_GATEWAY_TOKEN="your-secure-token"
```text
#### 7. 发布应用

1. 在 **版本管理与发布** 页面创建版本
2. 提交审核并发布
3. 等待管理员审批（企业自建应用通常自动通过）

#### 2. 发送测试消息

在飞书中找到您创建的机器人，发送一条消息，例如："hi"。

**在日志中应该能看到：**
