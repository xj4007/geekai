[根目录](../CLAUDE.md) > **api**

# API 后端服务

> 最后更新: 2026-02-21 23:00:18

## 模块职责

Go 语言实现的后端 API 服务，提供:
- RESTful API 接口 (Gin 框架)
- WebSocket 实时通信 (对话流式输出)
- OpenAI Realtime API 中继 (语音对话)
- 多 AI 模型集成 (OpenAI, Claude, DeepSeek 等)
- AI 绘画服务 (MidJourney, Stable Diffusion, DALL-E)
- AI 音乐/视频生成 (Suno, Luma, 可灵)
- AI 提示词生成 (歌词、绘画指令、视频指令)
- 用户认证与授权 (JWT，用户端 + 管理端双 Token)
- 支付集成 (支付宝、微信、虎皮椒、GeekPay)
- 文件存储 (本地、Minio、七牛、阿里云 OSS)
- 短信服务 (阿里云、短信宝)
- 邮件服务 (SMTP)
- 定时任务 (XXL-JOB)
- 数据库自动迁移 (MigrationService)
- 图片缩略图生成 (静态资源中间件)
- IP 地理定位 (ip2region，嵌入式 xdb)

## 入口与启动

**主入口**: `main.go` (596 行)

```go
// 启动流程:
// 1. 读取 CONFIG_FILE 环境变量 (默认 config.toml)
// 2. 加载 TOML 配置 -> types.AppConfig
// 3. 初始化 FX 依赖注入容器
// 4. 连接 MySQL (GORM) + Redis + LevelDB
// 5. 创建 ip2region Searcher (嵌入式 xdb)
// 6. 注册所有 Handler (fx.Provide) 和路由 (fx.Invoke)
// 7. 注册所有 Service 并启动异步任务 (MJ/SD/DALL-E/Suno/Video)
// 8. 执行数据库自动迁移 (MigrationService)
// 9. 加载系统配置 (从 DB chatgpt_configs 表)
// 10. 启动 HTTP 服务 (默认 0.0.0.0:5678)
// 11. 监听 SIGINT/SIGTERM 信号优雅退出
```

**启动命令**:
```bash
# 开发模式
APP_DEBUG=true go run main.go

# 指定配置文件
CONFIG_FILE=config.toml go run main.go

# 编译构建
make amd64  # Linux AMD64
make arm64  # Linux ARM64
# 输出到 bin/geekai-linux
```

**Docker 构建**: 多阶段构建 (golang:1.21-alpine -> alpine:3.18)，监听 5678 端口。

## 对外接口

### API 路由结构 (完整列表)

| 路由前缀 | 用途 | 认证 | Handler |
|----------|------|------|---------|
| `/api/user/` | 用户注册/登录/个人中心 | 部分 | `UserHandler` |
| `/api/chat/` | 对话管理 (列表/历史/TTS) | 部分 | `ChatHandler` |
| `/api/app/` | 应用/角色管理 | 部分 | `ChatRoleHandler` |
| `/api/app/type/` | 应用分类 | 不需要 | `ChatAppTypeHandler` |
| `/api/model/` | AI 模型列表 | 不需要 | `ChatModelHandler` |
| `/api/mj/` | MidJourney 绘画 | 需要 | `MidJourneyHandler` |
| `/api/sd/` | Stable Diffusion | 需要 | `SdJobHandler` |
| `/api/dall/` | DALL-E 绘画 | 需要 | `DallJobHandler` |
| `/api/suno/` | Suno 音乐 | 部分 | `SunoHandler` |
| `/api/video/` | 视频生成 (Luma/可灵) | 需要 | `VideoHandler` |
| `/api/payment/` | 支付 | 部分 | `PaymentHandler` |
| `/api/order/` | 订单查询 | 需要 | `OrderHandler` |
| `/api/product/` | 产品列表 | 不需要 | `ProductHandler` |
| `/api/invite/` | 邀请推广 | 需要 | `InviteHandler` |
| `/api/redeem/` | 兑换码 | 需要 | `RedeemHandler` |
| `/api/powerLog/` | 算力日志 | 需要 | `PowerLogHandler` |
| `/api/menu/` | 菜单列表 | 不需要 | `MenuHandler` |
| `/api/function/` | 函数服务 (微博/早报/WebSearch) | 不需要 | `FunctionHandler` |
| `/api/prompt/` | AI 提示词生成 | 需要 | `PromptHandler` |
| `/api/config/` | 系统配置 | 不需要 | `ConfigHandler` |
| `/api/sms/` | 短信验证码 | 不需要 | `SmsHandler` |
| `/api/captcha/` | 图形验证码 | 不需要 | `CaptchaHandler` |
| `/api/upload` | 文件上传/下载 | 需要 | `NetHandler` |
| `/api/markMap/gen` | 思维导图生成 | 需要 | `MarkMapHandler` |
| `/api/ws` | WebSocket 对话 | 需要 | `WebsocketHandler` |
| `/api/realtime` | Realtime 语音对话 | 需要 | `RealtimeHandler` |
| `/api/test/` | 测试 (SSE) | 不需要 | `TestHandler` |
| `/api/admin/` | 管理后台 (全部路由) | 管理员认证 | `admin.*Handler` |

### 管理后台 API 路由

| 路由前缀 | 用途 | Handler |
|----------|------|---------|
| `/api/admin/` (根) | 管理员登录/管理 | `admin.ManagerHandler` |
| `/api/admin/config/` | 系统配置管理 | `admin.ConfigHandler` |
| `/api/admin/apikey/` | API Key 管理 | `admin.ApiKeyHandler` |
| `/api/admin/user/` | 用户管理 | `admin.UserHandler` |
| `/api/admin/role/` | 应用/角色管理 | `admin.ChatAppHandler` |
| `/api/admin/model/` | AI 模型管理 | `admin.ChatModelHandler` |
| `/api/admin/product/` | 产品管理 | `admin.ProductHandler` |
| `/api/admin/order/` | 订单管理 | `admin.OrderHandler` |
| `/api/admin/redeem/` | 兑换码管理 | `admin.RedeemHandler` |
| `/api/admin/dashboard/` | 仪表盘统计 | `admin.DashboardHandler` |
| `/api/admin/chat/` | 对话记录管理 | `admin.ChatHandler` |
| `/api/admin/powerLog/` | 算力日志管理 | `admin.PowerLogHandler` |
| `/api/admin/menu/` | 菜单管理 | `admin.MenuHandler` |
| `/api/admin/function/` | 函数管理 | `admin.FunctionHandler` |
| `/api/admin/image/` | 绘图记录管理 | `admin.ImageHandler` |
| `/api/admin/media/` | 音视频记录管理 | `admin.MediaHandler` |
| `/api/admin/app/type/` | 应用分类管理 | `admin.ChatAppTypeHandler` |
| `/api/admin/upload` | 管理后台上传 | `admin.UploadHandler` |

## 关键依赖与配置

### 主要依赖 (go.mod)

| 依赖 | 版本 | 用途 |
|------|------|------|
| github.com/gin-gonic/gin | v1.9.1 | Web 框架 |
| gorm.io/gorm | v1.25.1 | ORM |
| gorm.io/driver/mysql | v1.4.7 | MySQL 驱动 |
| github.com/go-redis/redis/v8 | v8.11.5 | Redis 客户端 |
| go.uber.org/fx | v1.19.3 | 依赖注入 |
| github.com/golang-jwt/jwt/v5 | v5.0.0 | JWT 认证 |
| github.com/gorilla/websocket | v1.5.0 | WebSocket |
| github.com/sashabaranov/go-openai | v1.38.1 | OpenAI SDK |
| github.com/syndtr/goleveldb | v1.0.0 | LevelDB 本地存储 |
| github.com/imroc/req/v3 | v3.37.2 | HTTP 客户端 |
| github.com/go-pay/gopay | v1.5.101 | 支付 SDK |
| github.com/go-rod/rod | v0.116.2 | 浏览器自动化 |
| github.com/aliyun/aliyun-oss-go-sdk | v2.2.9 | 阿里云 OSS |
| github.com/minio/minio-go/v7 | v7.0.62 | Minio 对象存储 |
| github.com/qiniu/go-sdk/v7 | v7.17.1 | 七牛云 SDK |
| github.com/lionsoul2014/ip2region | - | IP 地理定位 |
| github.com/pkoukk/tiktoken-go | v0.1.1 | Token 计算 |
| github.com/xxl-job/xxl-job-executor-go | v1.2.0 | 定时任务执行器 |
| github.com/microcosm-cc/bluemonday | v1.0.26 | HTML 安全过滤 |
| github.com/shopspring/decimal | v1.3.1 | 精确十进制计算 |
| github.com/google/go-tika | v0.3.1 | 文档解析客户端 |

### 配置文件结构 (config.sample.toml)

```toml
Listen = "0.0.0.0:5678"           # 监听地址
MysqlDns = "..."                   # MySQL 连接串 (geekai_plus 数据库)
ProxyURL = ""                      # HTTP 代理地址 (用于海外 API)
StaticDir = "./static"             # 静态资源目录
StaticUrl = "/static"              # 静态资源 URL 前缀
TikaHost = "http://tika:9998"     # Apache Tika 服务地址

[Session]                          # 用户端 JWT 配置
  SecretKey = "..."
  MaxAge = 86400

[AdminSession]                     # 管理端 JWT 配置 (独立密钥)

[Redis]                            # Redis 配置
  Host = "localhost"
  Port = 6379
  Password = ""
  DB = 0

[ApiConfig]                        # 第三方插件 API (函数服务)
  ApiURL = "https://sapi.geekai.me"

[SMS]                              # 短信配置
  Active = "Ali"                   # Ali / Bao
  [SMS.Ali]                        # 阿里云短信
  [SMS.Bao]                        # 短信宝

[OSS]                              # 对象存储配置
  Active = "local"                 # local / Minio / QiNiu / AliYun
  [OSS.Local]
  [OSS.Minio]
  [OSS.QiNiu]
  [OSS.AliYun]

[SmtpConfig]                       # 邮件 SMTP 配置

[XXLConfig]                        # XXL-JOB 定时任务配置
  Enabled = false

[AlipayConfig]                     # 支付宝
  Enabled = false
[WechatPayConfig]                  # 微信支付
  Enabled = false
[HuPiPayConfig]                    # 虎皮椒
  Enabled = false
[GeekPayConfig]                    # GeekPay (易支付)
  Enabled = true
  Methods = ["alipay", "wxpay", "qqpay", "jdpay", "douyin", "paypal"]
```

## 数据模型

### 核心模型 (store/model/) - 24 个

| 模型 | 文件 | 表名 | 说明 |
|------|------|------|------|
| User | user.go | chatgpt_users | 用户信息 (用户名/手机/邮箱/算力/VIP/OpenID) |
| AdminUser | admin_user.go | - | 管理员用户 |
| UserLoginLog | user_login_log.go | chatgpt_user_login_logs | 登录日志 (IP/地址) |
| ChatItem | chat_item.go | - | 对话会话 |
| ChatMessage | chat_history.go | - | 对话消息 |
| ChatModel | chat_model.go | - | AI 模型配置 |
| ChatRole | chat_role.go | - | 角色/应用 |
| AppType | app_type.go | - | 应用分类 |
| Config | config.go | chatgpt_configs | 系统配置 (JSON 存储) |
| Menu | menu.go | - | 前端菜单 |
| Function | function.go | - | 函数/插件 |
| Order | order.go | - | 订单 |
| Product | product.go | - | 充值产品 |
| PowerLog | power_log.go | - | 算力消费日志 |
| ApiKey | api_key.go | - | API 密钥 |
| Redeem | redeem.go | - | 兑换码 |
| InviteCode | invite_code.go | - | 邀请码 |
| InviteLog | invite_log.go | - | 邀请记录 |
| File | file.go | - | 上传文件 |
| MidJourneyJob | mj_job.go | - | MJ 任务 |
| SdJob | sd_job.go | - | SD 任务 |
| DallJob | dalle_job.go | - | DALL-E 任务 |
| SunoJob | suno_job.go | - | Suno 音乐任务 |
| VideoJob | video_job.go | chatgpt_video_jobs | 视频任务 (Luma/可灵) |

### VO 视图对象 (store/vo/) - 24 个

每个 Model 都有对应的 VO，用于 API 响应数据序列化，基类为 `BaseVo` (Id + CreatedAt + UpdatedAt)。

### 用户模型字段

```go
type User struct {
    Id, Username, Mobile, Email, Nickname, Password, Avatar, Salt string
    Power int          // 剩余算力
    ExpiredTime int64  // VIP 过期时间
    Status bool        // 账号状态
    Vip bool           // 是否 VIP
    ChatConfig string  // 聊天配置 (JSON)
    ChatRoles string   // 已选角色 (JSON)
    ChatModels string  // 已选模型 (JSON)
    LastLoginAt int64
    LastLoginIp string
    OpenId string      // 第三方登录 ID
    Platform string    // 登录平台
}
```

## 中间件链

`app_server.go` 中注册的中间件执行顺序:

1. **corsMiddleware** - CORS 跨域处理，动态设置 `Access-Control-Allow-Origin`
2. **staticResourceMiddleware** - 静态资源处理，支持 `?imageView2` 缩略图生成
3. **authorizeMiddleware** - JWT 认证，区分用户端 (`Authorization`) 和管理端 (`Admin-Authorization`)，WebSocket 通过子协议传递 Token
4. **parameterHandlerMiddleware** - 参数清洗，GET/POST JSON 字符串值去除两端空格
5. **errorHandler** - 全局异常恢复 (panic recover)

## 目录结构

```
api/
├── main.go                    # 入口 (路由注册、FX 容器)
├── config.sample.toml         # 配置模板
├── Makefile                   # 构建脚本 (amd64/arm64)
├── Dockerfile                 # Docker 多阶段构建
├── fresh.conf                 # 热重载配置
├── res/                       # 嵌入资源
│   ├── ip2region.xdb          # IP 地理数据库
│   └── certs/                 # 证书文件
├── core/                      # 核心框架 (13 文件)
│   ├── app_server.go          # HTTP 服务器 + 中间件
│   ├── config.go              # 配置加载/保存
│   └── types/                 # 类型定义
│       ├── config.go          # AppConfig + SystemConfig + 支付配置
│       ├── chat.go            # 聊天相关类型
│       ├── client.go          # 客户端类型
│       ├── session.go         # 会话类型
│       ├── order.go           # 订单类型
│       ├── function.go        # 函数类型
│       ├── task.go            # 异步任务类型
│       ├── oss.go             # OSS 类型
│       ├── sms.go             # SMS 类型
│       ├── web.go             # Web 通用类型
│       └── locked_map.go      # 线程安全 Map
├── handler/                   # 控制器层 (30 文件)
│   ├── base_handler.go        # 基础 Handler (获取登录用户等)
│   ├── user_handler.go        # 用户 (746 行)
│   ├── chat_handler.go        # 对话 (587 行)
│   ├── chat_openai_handler.go # OpenAI 集成 (253 行)
│   ├── mj_handler.go          # MidJourney (432 行)
│   ├── payment_handler.go     # 支付 (453 行)
│   ├── function_handler.go    # 函数服务 (348 行)
│   ├── video_handler.go       # 视频 (340 行)
│   ├── suno_handler.go        # Suno (325 行)
│   ├── realtime_handler.go    # Realtime 语音 (221 行)
│   ├── prompt_handler.go      # AI 提示词 (167 行)
│   ├── ws_handler.go          # WebSocket (152 行)
│   └── admin/                 # 管理后台 Handler (16 文件)
│       ├── admin_handler.go   # 管理员认证 (289 行)
│       ├── user_handler.go    # 用户管理 (356 行)
│       ├── chat_handler.go    # 对话管理 (269 行)
│       └── ...
├── service/                   # 业务服务层 (22 文件)
│   ├── user_service.go        # 用户服务
│   ├── ws_service.go          # WebSocket 服务
│   ├── migration_service.go   # 数据库迁移
│   ├── license_service.go     # License 服务
│   ├── captcha_service.go     # 验证码服务
│   ├── smtp_sms_service.go    # 邮件服务
│   ├── snowflake.go           # 雪花 ID 生成器
│   ├── xxl_job_service.go     # XXL-JOB 执行器
│   ├── crawler/service.go     # 爬虫服务
│   ├── mj/                    # MidJourney (service + client)
│   ├── sd/service.go          # Stable Diffusion
│   ├── dalle/service.go       # DALL-E
│   ├── suno/service.go        # Suno 音乐 (440 行)
│   ├── video/video.go         # 视频生成 (661 行)
│   ├── oss/                   # 文件存储 (6 文件)
│   │   ├── uploader.go        # 接口定义
│   │   ├── uploader_manager.go
│   │   ├── localstorage.go
│   │   ├── minio_oss.go
│   │   ├── qiniu_oss.go
│   │   └── aliyun_oss.go
│   ├── payment/               # 支付 (5 文件)
│   │   ├── types.go
│   │   ├── alipay_service.go
│   │   ├── wepay_service.go
│   │   ├── hupipay_serive.go
│   │   └── geekpay_service.go
│   └── sms/                   # 短信 (4 文件)
│       ├── service.go
│       ├── service_manager.go
│       ├── aliyun.go
│       └── bao.go
├── store/                     # 数据存储层 (52 文件)
│   ├── mysql.go               # MySQL 连接
│   ├── redis.go               # Redis 连接
│   ├── redis_queue.go         # Redis 队列封装
│   ├── leveldb.go             # LevelDB 封装
│   ├── model/                 # GORM 模型 (24 文件)
│   └── vo/                    # 视图对象 (24 文件)
├── utils/                     # 工具函数 (8 文件)
│   ├── common.go              # 通用工具 (JSON/Int/Sha256)
│   ├── crypto.go              # 加密工具
│   ├── file.go                # 文件操作
│   ├── net.go                 # 网络工具
│   ├── openai.go              # OpenAI 工具
│   ├── strings.go             # 字符串工具
│   ├── upload.go              # 上传工具
│   └── resp/response.go       # 统一响应封装
├── logger/logger.go           # 日志 (zap + lumberjack)
└── test/                      # 测试 (3 文件)
    ├── crawler_test.go
    ├── test.go
    └── run_crawler_test.sh
```

## 测试与质量

**测试文件**: `test/crawler_test.go` (214 行), `test/test.go` (55 行辅助)

```bash
# 运行测试
cd api
go test ./...

# 运行特定测试
go test -v ./test/...

# 运行爬虫测试
bash test/run_crawler_test.sh
```

**代码质量**:
- 使用 Go 标准格式化
- FX 依赖注入确保可测试性
- 中间件处理跨域、认证、异常恢复
- 参数自动清洗 (trim)
- 统一响应格式 (`utils/resp`)

## 常见问题 (FAQ)

### Q: 如何添加新的 API 接口?

1. 在 `handler/` 创建 Handler 结构体，嵌入 `BaseHandler`
2. 在 `main.go` 中使用 `fx.Provide(handler.NewXxxHandler)` 注册
3. 在 `main.go` 中使用 `fx.Invoke` 注册路由组

### Q: 如何添加新的 AI 模型支持?

1. 在 `service/` 创建对应服务
2. 实现模型调用逻辑
3. 在 `main.go` 中用 FX 注册
4. 在后台管理添加模型配置

### Q: 如何添加新的支付方式?

1. 在 `service/payment/` 创建新的支付服务文件
2. 实现支付创建和回调通知逻辑
3. 在 `core/types/config.go` 添加配置结构体
4. 在 `handler/payment_handler.go` 添加对应路由
5. 在 `main.go` 注册

### Q: 如何添加新的数据模型?

1. 在 `store/model/` 创建模型文件 (实现 `TableName()`)
2. 在 `store/vo/` 创建视图对象
3. 在 `service/migration_service.go` 的 `Migrate()` 中添加 `&model.NewModel{}`
4. 程序启动时自动迁移数据表

### Q: 如何配置 OSS?

编辑 `config.toml` 中的 `[OSS]` 节:
- `Active = "local"` 使用本地存储
- `Active = "Minio"` 使用 Minio
- `Active = "QiNiu"` 使用七牛云
- `Active = "AliYun"` 使用阿里云 OSS

## 相关文件清单

| 类别 | 路径 |
|------|------|
| 入口 | main.go |
| 配置模板 | config.sample.toml |
| 构建脚本 | Makefile |
| Docker 构建 | Dockerfile |
| 核心服务 | core/app_server.go |
| 配置类型 | core/types/config.go |
| 用户 Handler | handler/user_handler.go |
| 聊天 Handler | handler/chat_handler.go |
| OpenAI 集成 | handler/chat_openai_handler.go |
| 支付 Handler | handler/payment_handler.go |
| Realtime 语音 | handler/realtime_handler.go |
| AI 提示词 | handler/prompt_handler.go |
| WebSocket | handler/ws_handler.go |
| 支付服务 | service/payment/ |
| OSS 服务 | service/oss/ |
| 视频服务 | service/video/video.go |
| 数据库迁移 | service/migration_service.go |
| 数据模型 | store/model/ |
| 视图对象 | store/vo/ |
| 响应封装 | utils/resp/response.go |

---

## 变更记录 (Changelog)

### 2026-02-21 (增量更新)
- 补充完整的 API 路由列表 (用户端 + 管理端)
- 补充 VO (View Object) 层说明
- 补充中间件链详情
- 补充 MigrationService 自动迁移机制
- 补充 RealtimeHandler (语音对话) 和 PromptHandler (AI 提示词)
- 补充 Docker 构建信息
- 更新文件统计: 158 Go 文件
- 目录结构增加详细行数标注

### 2026-02-07 (初始化)
- 创建模块文档
- 识别 130+ Go 文件
- 记录核心架构与 API 路由
