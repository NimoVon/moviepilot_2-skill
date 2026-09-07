# MoviePilot MCP / HTTP API 接入

本文件说明如何用 HTTP REST 调用 MoviePilot 能力（比浏览器自动化稳定），以及如何注册为 MCP 服务。

## 端口与地址
- 前端 Web：`NGINX_PORT`，默认 `3000`
- 后端 API / MCP：`PORT`，默认 `3001`
- 反向代理/HTTPS 场景用对外域名，如 `https://moviepilot.example.com/api/v1/mcp`

## 认证
推荐用请求头 `X-API-KEY: <API_TOKEN>`。兜底用查询参数 `?apikey=<API_TOKEN>`（会出现在日志/截图，不推荐）。
- API_TOKEN 获取：设定 -> 系统 -> 基础设置 -> API 令牌；或启动日志 `logs/moviepilot.log` 搜 `API_TOKEN`。
- V2.0+ 要求 API_TOKEN ≥ 16 位复杂字符串，否则每次启动自动生成新值。

## MCP 端点
- 端点：`/api/v1/mcp`（完整示例 `http://127.0.0.1:3001/api/v1/mcp`）
- transport：`streamable-http`；客户端字段若是 `type` 则填 `http`。
- 支持协议版本：`2025-11-25`、`2025-06-18`、`2024-11-05`。
- 不要拼 `/sse`、`/messages` 等额外路径。

## 工具接口（REST 直调）
- `GET /api/v1/mcp/tools` — 列出全部工具（应以当前实例实际返回为准）
- `GET /api/v1/mcp/tools/{tool_name}` — 查看单个工具定义
- `GET /api/v1/mcp/tools/{tool_name}/schema` — 查看参数 Schema
- `POST /api/v1/mcp/tools/call` — 按 REST 直接调用工具

### 先验证可达性
```bash
curl -H "X-API-KEY: YOUR_API_TOKEN" \
  http://127.0.0.1:3001/api/v1/mcp/tools
```
返回 JSON 工具列表 => 网络、端口、认证 OK。返回 401 => API_TOKEN 缺失或不正确。

## 通用注册模板

推荐写法（请求头认证）：
```json
{
  "mcpServers": {
    "moviepilot": {
      "type": "http",
      "url": "http://127.0.0.1:3001/api/v1/mcp",
      "headers": { "X-API-KEY": "YOUR_API_TOKEN" }
    }
  }
}
```
兜底写法（查询参数认证）：
```json
{
  "mcpServers": {
    "moviepilot": {
      "type": "http",
      "url": "http://127.0.0.1:3001/api/v1/mcp?apikey=YOUR_API_TOKEN"
    }
  }
}
```

## 在 OpenClaw 中注册
优先 `transport: streamable-http`（不要留默认 sse）：
```bash
openclaw mcp set moviepilot '{
  "url": "http://127.0.0.1:3001/api/v1/mcp",
  "transport": "streamable-http",
  "headers": { "X-API-KEY": "YOUR_API_TOKEN" }
}'
openclaw mcp show moviepilot
```
注意：`openclaw mcp set` 只写入注册配置，不代表已建立连接。OpenClaw 与 MoviePilot 不在同机时把 `127.0.0.1` 改成真实地址。

## 在 Claude Code 注册
```bash
claude mcp add-json moviepilot '{"type":"http","url":"http://127.0.0.1:3001/api/v1/mcp","headers":{"X-API-KEY":"YOUR_API_TOKEN"}}'
```

## Web 登录（JWT 方式，可用作 API 认证替代）
```bash
curl -X POST http://127.0.0.1:3001/api/v1/login/access-token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=admin&password=YOUR_PASSWORD"
# 返回 {"access_token": "...", "token_type": "bearer"}
# 后续: curl -H "Authorization: Bearer <token>" ...
```

## 常见用途 curl 写法
- 列站点/站点数据：`GET /api/v1/site/userdata/latest`（与站点数据相关）
- 更多端点以实际 `/api/v1/openapi.json`（Swagger）为准。

## 安全注意
- 不要把 API_TOKEN 写进 URL、聊天、截图、git。
- 远程 Agent 访问前，先做好 HTTPS / 访问控制 / 网络隔离 / 反代鉴权，不要把带管理员级凭证的接口裸奔到公网。
