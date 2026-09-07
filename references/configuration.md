# MoviePilot 配置与环境变量

> 结构化配置（下载器、媒体服务器、通知渠道、存储目录、过滤规则、工作流）优先在前端对应页面配置，不建议写环境变量。环境变量更适合：部署参数、密钥兜底、数据库/缓存连接、低层安全开关、首次启动前必须生效项、前端暂无入口的高级项。

## 关键配置项速查

| 变量 | 说明 | 默认/取值 |
|---|---|---|
| `NGINX_PORT` | WEB 前端端口 | 3000 |
| `PORT` | API 服务端口 | 3001 |
| `SSL_NGINX_PORT` | HTTPS 端口 | 443 |
| `API_TOKEN` | API 密钥，MCP 认证密钥 | V1 默认 moviepilot；V2.0+ 需 ≥16 位复杂字符串，不满足则每次启动自动生成新值 |
| `SUPERUSER` | 超级管理员用户名 | V1/V2 默认 admin；V3 初始化页创建 |
| `SUPERUSER_PASSWORD` | 超级管理员初始密码 | V2.8+ 可预置；V3 默认空，初始化页提交后存哈希 |
| `PUID` / `PGID` | 运行程序用户 uid/gid | Docker |
| `UMASK` | 掩码权限 | 000 |
| `CONFIG_DIR` | 配置文件目录 | Docker/套件 `/config`；macOS `~/Library/Application Support/MoviePilot` |
| `START_NOGOSU` | 不切换容器内用户启动，缓解无根容器权限问题（需完整参数，仅 Docker） | false |
| `AUTH_SITE` | 认证站点，认证通过才能用站点功能；多个用 `,` 分隔 | — |
| `PROXY_HOST` | 网络代理，访问 TMDB/更新/连 Github 用。格式 `http(s)://ip:port`、`socks5://user:pass@host:port` | — |
| `PLUGIN_MARKET` | 插件市场仓库地址，仅 Github main 分支，多个用 `,` 分隔 | — |
| `TZ` | 时区 | 容器/进程时区 |
| `DOCKER_CLIENT_API` | V1/V2 内建重启接口 | `tcp://127.0.0.1:38379`；V3 不用（由容器内 supervisor 托管） |
| `CACHE_BACKEND_URL` | 外部缓存（Redis）连接串 | `redis://:password@host:port`；用 Redis 可减主程序内存并迁移本地缓存 |
| `DB_POSTGRESQL_PORT` | PostgreSQL 端口 | 5432 |
| `OCR_HOST` | OCR 识别服务器 | `http(s)://ip:port` |
| `DOWNLOAD_TMPEXT` | 下载器临时文件后缀 | `['.!qb', '.part']` |
| `LLM_SUPPORT_IMAGE_INPUT` | 智能助手图片输入 | true |
| `LLM_SUPPORT_AUDIO_INPUT` | 语音识别输入 | false |
| `LLM_SUPPORT_AUDIO_OUTPUT` | 语音输出 | false |

## 建议用前端配置的项（不要写环境变量）
- 下载器、媒体服务器、通知渠道、存储目录、过滤规则、工作流。

## Docker / 挂载要点
- 下载目录根目录需映射到容器，且与下载器使用的目录一致，否则文件存到容器内导致异常。
- 日志文件路径：`/config` 对应映射目录下 `logs/moviepilot.log`。
- 管理员密码/API_TOKEN 可在此日志搜 `超级管理员` / `API_TOKEN` 定位。
- 首次启动（V3）：Web 初始化页创建管理员用户名、密码、API Key。
