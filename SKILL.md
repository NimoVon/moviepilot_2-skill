---
name: moviepilot
description: 操作和运维 MoviePilot（媒体自动化管理 NAS 应用，含 PT 站点管理、搜索、订阅、下载、文件整理、插件、通知、诊断）。当用户要查看/添加/维护 PT 站点状态、搜索或下载影视/音乐资源、管理订阅、查看下载器连接、配置通知或插件、排查 MoviePilot 问题，或用 MCP/HTTP API 调用 MoviePilot 能力时使用。涵盖 V2/V3 的 Web UI 路由、API/MCP 端点、认证方式与环境变量。
---

# MoviePilot

## Overview

MoviePilot 是基于 NAStool 二次开发的媒体自动化管理工具，运行于 NAS/Docker。核心能力：PT 站点管理、媒体搜索、订阅自动检索、下载管理、文件整理、插件市场、消息通知、AI 智能助手，并内置标准 MCP Server 供第三方 Agent 直接调用。

**官方文档**：https://wiki.movie-pilot.org（Wiki.js 站，web_fetch 会因解析到 198.18.x.x 保留段被 SSRF 拦截，改用 curl 抓取）

**实例上下文（本机）**：
- 前端 Web UI：`http://192.168.31.2:3000`
- 后端 API / MCP：`http://192.168.31.2:3001`
- 当前版本：V2.15.3（注意文档区分 V1/V2/V3 行为差异）
- 已登录用户：`admin`（管理员，昵称 mpv2）

## 认证方式（两种）

1. **Web 用户名/密码登录**：`POST /api/v1/login/access-token`，OAuth2 表单（`username`/`password`），返回 JWT `access_token`，后续 API 用 `Authorization: Bearer <token>`。
2. **API_TOKEN**：管理员级凭证，也是 MCP 认证密钥。V2.0+ 要求 16 位以上复杂字符串。MCP/API 用请求头 `X-API-KEY: <API_TOKEN>` 或查询参数 `?apikey=<API_TOKEN>`。
   - 获取路径：设定 -> 系统 -> 基础设置 -> API 令牌，或在启动日志 `logs/moviepilot.log` 中搜 `API_TOKEN`。

> ⚠️ 不要把密码/API_TOKEN 明文写入技能文件或聊天日志。凭据只在使用时按需提供。

## MCP / API 接入（本技能最常用的调用方式）

MoviePilot 已内置 MCP Server，无需额外部署。直接用 HTTP REST 调用工具即可（比浏览器自动化更稳）：
- MCP 端点：`http://<host>:3001/api/v1/mcp`（transport `streamable-http`，不要拼 `/sse`、`/messages`）
- `GET /api/v1/mcp/tools` — 列出全部工具
- `GET /api/v1/mcp/tools/{tool_name}/schema` — 查看某工具参数 Schema
- `POST /api/v1/mcp/tools/call` — 按 REST 直接调用工具

详细接入与 curl 示例见 `references/api.md`；在 OpenClaw 中注册见 `openclaw mcp set moviepilot '{...}'`。

## 核心能力与入口

### 1. 站点管理（PT/资源站）
- Web 路由：`http://<host>:3000/#/site`
- 站点卡片两指标：↑ 上传量 / ↓ 下载量；卡片右上角图标 = 连接状态（绿=正常且快、黄=可连通但 >5s 慢、红=无法连接/超时）。
- 站点优先级 = 展示顺序（数值越小越优先）。
- 添加站点字段：站点地址、优先级、RSS 地址、超时时间、站点 Cookie、Authorization 请求头、API Key、User-Agent、流控、代理、浏览器仿真。
- 推荐用 CookieCloud 批量同步 Cookie + 定时更新。
- 详见 `references/site.md`。

### 2. 搜索媒体/资源
- 聚合搜索：Web 点搜索图标或 `Ctrl+K`；支持媒体信息 / 人物 / 站点资源 / 音乐。
- 精确匹配 vs 模糊搜索（`设定->搜索` 可设媒体数据源、搜索站点、过滤规则）。
- 详见 `references/operations.md`。

### 3. 订阅（自动检索/追更）
- 添加订阅后约 3 分钟内做一次全站搜索补存量；后台任务定期 `订阅刷新` 检出增量。
- 订阅模式：`自动`（内置爬虫）或 `站点RSS`（用站点 RSS 链接）。
- 洗版（升级画质）/ 订阅搜索机制见 `references/operations.md`。

### 4. 下载与文件整理
- 下载目录需与下载器一致，Docker 需映射到容器。
- 自动整理：下载器监控（默认 5 分钟）或目录监控（实时，勿用于网盘目录）。
- 音乐整理走 `艺术家/专辑 (年份)/曲序 - 歌曲.扩展名`。
- 详见 `references/operations.md`。

### 5. 插件 / 通知 / 智能助手
- 插件市场需配置 `PLUGIN_MARKET`（Github main 分支）与 Github Token。
- 通知渠道：微信/Telegram/Slack/SynologyChat/VoceChat，配好后支持远程交互（发片名搜/下载/订阅）。
- 智能助手可在 AI_RECOMMEND_ENABLED 时做 AI 推荐，并承担远程交互。
- 详见 `references/operations.md`。

### 6. 故障诊断（doctor）
- Web/后端/插件全不可用时用 `moviepilot doctor`（离线只读诊断）。
- 用法：`moviepilot doctor [--json|--fix|--deep]`，Docker 内 `docker exec -it <c> moviepilot doctor`。
- 详见 `references/operations.md`。

## 关键环境变量 / 配置

常用：`API_TOKEN`、`PORT`(3001)、`NGINX_PORT`(3000)、`PUID`/`PGID`/`UMASK`、`SUPERUSER`/`SUPERUSER_PASSWORD`、`AUTH_SITE`、`PROXY_HOST`、`PLUGIN_MARKET`、`TZ`、`CACHE_BACKEND_URL`(Redis)、`DB_POSTGRESQL_PORT`。

> 结构化配置（下载器/媒体服务器/通知/存储/过滤规则/工作流）建议在前端页面配置，不写环境变量。完整说明见 `references/configuration.md`。

## 使用提示

- 优先用 MCP/API 完成查询与操作（列站点、搜资源、看下载任务、触发同步），比浏览器自动化稳定。
- Web UI 路由会随版本微调（如站点页 `#/site`）；进不了页面再回落 API 或 `doctor`。
- 写操作（添加站点、启停服务、改配置、删订阅等）遵守工作区「先展示、用户确认再执行」的二次确认规则。
