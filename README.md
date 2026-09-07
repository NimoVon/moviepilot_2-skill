# MoviePilot Skill

一个用于在 OpenClaw（及支持远程 HTTP MCP 的第三方 Agent）中操作和运维 **MoviePilot** 的 AgentSkill。

MoviePilot 是基于 NAStool 二次开发的媒体自动化管理工具（NAS / Docker），本技能封装了它的核心能力：PT 站点管理、媒体搜索、订阅、下载、文件整理、插件、通知、故障诊断，以及通过内置 **MCP Server** 直接调用的方式。

> 官方文档：https://wiki.movie-pilot.org
>
> （注：该域名解析到 198.18.x.x 保留段，`web_fetch` 会被 SSRF 拦截；需要时用 `curl` 抓取。）

## 技能结构

```
moviepilot/
├── SKILL.md                    # 主入口：架构、认证、核心能力导航
└── references/
    ├── api.md                  # MCP / HTTP API 接入、认证、工具端点、curl / OpenClaw 注册
    ├── site.md                 # 站点管理（字段、CookieCloud、连接状态、优先级）
    ├── operations.md           # 搜索 / 订阅 / 下载 / 整理 / 插件 / 通知 / doctor 诊断
    └── configuration.md        # 关键环境变量速查表
```

## 认证方式

- **Web 用户名/密码登录**：`POST /api/v1/login/access-token`（OAuth2 表单），返回 JWT
- **API_TOKEN（推荐，也是 MCP 认证密钥）**：请求头 `X-API-KEY: <API_TOKEN>` 或查询参数 `?apikey=<API_TOKEN>`，`设定 -> 系统 -> API 令牌` 处管理

## MCP / API 接入

MoviePilot 内置 MCP Server，无需额外部署：

- 端点：`http://<host>:3001/api/v1/mcp`（transport `streamable-http`）
- 工具接口（REST 直调）：
  - `GET /api/v1/mcp/tools` — 列出全部工具
  - `GET /api/v1/mcp/tools/{tool_name}/schema` — 查看参数 Schema
  - `POST /api/v1/mcp/tools/call` — 直接调用工具（入参用 `tool_name`，不是 `name`）

OpenClaw 注册示例：

```bash
openclaw mcp set moviepilot '{
  "url": "http://127.0.0.1:3001/api/v1/mcp",
  "transport": "streamable-http",
  "headers": { "X-API-KEY": "YOUR_API_TOKEN" }
}'
```

## 常用能力

- **查状态**：`query_sites`、`query_site_userdata`、`query_downloaders`、`query_download_tasks`
- **搜索 / 下载**：`search_media`、`search_torrents`、`get_search_results`、`add_download_tasks`
- **订阅**：`add_subscribe`、`query_subscribes`、`update_subscribe`
- **整理 / 诊断**：`query_transfer_history`、`query_doctor_report`、`list_directory`
- **插件 / 设置**：`query_installed_plugins`、`update_plugin_config`、`query_system_settings`

## 安全提示

- `query_sites` 会带出各站 **Cookie / RSS passkey**（等于站点账号凭证），只在受控环境使用、勿外泄。
- API_TOKEN 是管理员级凭证，注意保密，不要写进 URL、日志或截图。
- 远程访问前请做好 HTTPS / 访问控制 / 网络隔离。
