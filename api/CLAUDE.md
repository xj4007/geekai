[根目录](../CLAUDE.md) > **api**

# API 后端服务

> 最后更新: 2026-02-07 16:01:46

## 模块职责

Go 语言实现的后端 API 服务，提供:
- RESTful API 接口
- WebSocket 实时通信
- 多 AI 模型集成 (OpenAI, Claude, DeepSeek 等)
- AI 绘画服务 (MidJourney, SD, DALL-E)
- AI 音乐/视频生成 (Suno, Luma, 可灵)
- 用户认证与授权 (JWT)
- 支付集成 (支付宝、微信、虎皮椒等)
- 文件存储 (本地、Minio、七牛、阿里云 OSS)

## 入口与启动

**主入口**: `main.go`

```go
// 启动流程:
// 1. 加载配置文件 (config.toml)
// 2. 初始化 FX 依赖注入容器
// 3. 连接 MySQL、Redis、LevelDB
// 4. 注册所有 Handler 和 Service
// 5. 启动 HTTP 服务
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
```

## 对外接口

### API 路由结构

| 路由前缀 | 用途 | 认证 |
|----------|------|------|
| `/api/user/` | 用户相关 | 部分需要 |
| `/api/chat/` | 对话功能 | 需要 |
| `/api/app/` | 应用/角色 | 部分需要 |
| `/api/mj/` | MidJourney 绘画 | 需要 |
| `/api/sd/` | Stable Diffusion | 需要 |
| `/api/dall/` | DALL-E 绘画 | 需要 |
| `/api/suno/` | Suno 音乐 | 需要 |
| `/api/video/` | 视频生成 | 需要 |
| `/api/payment/` | 支付相关 | 部分需要 |
| `/api/admin/` | 管理后台 | 管理员认证 |
| `/api/ws` | WebSocket | 需要 |
| `/api/realtime` | 实时语音 | 需要 |

### 核心 API 示例

```
POST /api/user/login      # 用户登录
POST /api/user/register   # 用户注册
GET  /api/chat/list       # 对话列表
POST /api/mj/image        # MJ 生图
POST /api/payment/doPay   # 发起支付
```

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

### 配置文件结构 (config.toml)

```toml
Listen = "0.0.0.0:5678"           # 监听地址
MysqlDns = "..."                   # MySQL 连接串
ProxyURL = ""                      # 代理地址
StaticDir = "./static"             # 静态资源目录

[Session]                          # JWT 配置
  SecretKey = "..."
  MaxAge = 86400

[Redis]                            # Redis 配置
  Host = "localhost"
  Port = 6379

[OSS]                              # 存储配置
  Active = "local"                 # local/minio/qiniu/aliyun

[SMS]                              # 短信配置
  Active = "Ali"

[AlipayConfig]                     # 支付宝配置
[WechatPayConfig]                  # 微信支付配置
[HuPiPayConfig]                    # 虎皮椒配置
[GeekPayConfig]                    # GEEK 支付配置
```

## 数据模型

### 核心模型 (store/model/)

| 模型 | 文件 | 表名 | 说明 |
|------|------|------|------|
| User | user.go | chatgpt_users | 用户信息 |
| ChatItem | chat_item.go | - | 对话会话 |
| ChatMessage | chat_history.go | - | 对话消息 |
| ChatModel | chat_model.go | - | AI 模型配置 |
| ChatRole | chat_role.go | - | 角色/应用 |
| Order | order.go | - | 订单 |
| Product | product.go | - | 充值产品 |
| ApiKey | api_key.go | - | API 密钥 |
| MidJourneyJob | mj_job.go | - | MJ 任务 |
| SdJob | sd_job.go | - | SD 任务 |
| DallJob | dalle_job.go | - | DALL-E 任务 |
| SunoJob | suno_job.go | - | Suno 任务 |
| VideoJob | video_job.go | - | 视频任务 |

### 用户模型示例

```go
type User struct {
    Id          uint      `gorm:"primaryKey"`
    Username    string    `gorm:"uniqueIndex"`
    Mobile      string
    Email       string
    Password    string
    Power       int       // 剩余算力
    Vip         bool      // 是否会员
    ExpiredTime int64     // 会员过期时间
    Status      bool
}
```

## 目录结构

```
api/
├── main.go              # 入口文件
├── config.sample.toml   # 配置模板
├── Makefile             # 构建脚本
├── core/                # 核心框架
│   ├── app_server.go    # HTTP 服务器
│   ├── config.go        # 配置加载
│   └── types/           # 类型定义
├── handler/             # 控制器层
│   ├── admin/           # 管理后台 Handler
│   ├── user_handler.go
│   ├── chat_handler.go
│   └── ...
├── service/             # 业务服务层
│   ├── mj/              # MidJourney 服务
│   ├── sd/              # Stable Diffusion 服务
│   ├── dalle/           # DALL-E 服务
│   ├── suno/            # Suno 音乐服务
│   ├── video/           # 视频生成服务
│   ├── oss/             # 文件存储服务
│   ├── payment/         # 支付服务
│   ├── sms/             # 短信服务
│   └── ...
├── store/               # 数据存储层
│   ├── model/           # GORM 模型
│   ├── mysql.go         # MySQL 连接
│   ├── redis.go         # Redis 连接
│   └── leveldb.go       # LevelDB
├── utils/               # 工具函数
│   ├── common.go
│   ├── crypto.go
│   ├── file.go
│   └── resp/            # 响应封装
├── logger/              # 日志
└── test/                # 测试
```

## 测试与质量

**测试文件**: `test/crawler_test.go`

```bash
# 运行测试
cd api
go test ./...

# 运行特定测试
go test -v ./test/...
```

**代码质量**:
- 使用 Go 标准格式化
- 依赖注入确保可测试性
- 中间件处理跨域、认证、异常

## 常见问题 (FAQ)

### Q: 如何添加新的 API 接口?

1. 在 `handler/` 创建 Handler 文件
2. 在 `main.go` 中使用 `fx.Provide` 注册
3. 在 `main.go` 中使用 `fx.Invoke` 注册路由

### Q: 如何添加新的 AI 模型支持?

1. 在 `service/` 创建对应服务
2. 实现模型调用逻辑
3. 在后台管理添加模型配置

### Q: 如何配置支付?

编辑 `config.toml` 中的支付配置节:
- `[AlipayConfig]` - 支付宝
- `[WechatPayConfig]` - 微信支付
- `[HuPiPayConfig]` - 虎皮椒
- `[GeekPayConfig]` - GEEK 支付

## 相关文件清单

| 类别 | 路径 |
|------|------|
| 入口 | main.go |
| 配置模板 | config.sample.toml |
| 构建脚本 | Makefile |
| 核心服务 | core/app_server.go |
| 用户 Handler | handler/user_handler.go |
| 聊天 Handler | handler/chat_handler.go |
| OpenAI 集成 | handler/chat_openai_handler.go |
| 支付服务 | service/payment/ |
| 数据模型 | store/model/ |

---

## 变更记录 (Changelog)

### 2026-02-07 (初始化)
- 创建模块文档
- 识别 130+ Go 文件
- 记录核心架构与 API 路由
