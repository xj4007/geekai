[根目录](../CLAUDE.md) > **web**

# Web 前端应用

> 最后更新: 2026-02-21 23:00:18

## 模块职责

Vue 3 实现的前端应用，提供:
- PC 端 AI 对话界面 (Element Plus)
- 移动端适配界面 (Vant)
- 管理后台界面 (Element Plus)
- AI 绘画任务管理 (MJ/SD/DALL-E)
- AI 音乐创作 (Suno)
- AI 视频创作 (Luma/可灵)
- 思维导图 (MarkMap)
- 实时语音对话 (Realtime Conversation)
- 用户中心与会员充值
- 深色/浅色主题切换
- 第三方 OAuth 登录

## 入口与启动

**主入口**: `src/main.js` (124 行)

```javascript
// 启动流程:
// 1. 创建 Vue 应用实例
// 2. 注册 Pinia 状态管理
// 3. 初始化主题 (data-theme 属性)
// 4. 注册 Vant 组件 (约 40+ 组件，逐个注册)
// 5. 注册 Element Plus
// 6. 配置 Vue Router
// 7. 挂载应用到 #app
```

**启动命令**:
```bash
npm install      # 安装依赖
npm run dev      # 开发模式 (vue-cli-service serve, 端口 8888)
npm run build    # 生产构建 (输出到 dist/)
npm run lint     # ESLint 代码检查
```

**开发代理**: `vue.config.js` 中配置了 `/static/upload/` 路径代理到 `VUE_APP_API_HOST`。

## 对外接口

### 路由结构 (router.js) -- 完整列表

**PC 端主路由** (嵌套在 `/home` 布局下):

| 路由 | 组件 | 说明 |
|------|------|------|
| `/` | Index.vue | 首页 (落地页) |
| `/chat` | ChatPlus.vue (1525行) | AI 对话 |
| `/chat/:id` | ChatPlus.vue | 指定对话 |
| `/mj` | ImageMj.vue (1354行) | MidJourney 绘画 |
| `/sd` | ImageSd.vue (758行) | Stable Diffusion |
| `/dalle` | Dalle.vue (591行) | DALL-E 绘画 |
| `/suno` | Suno.vue (652行) | Suno 音乐创作 |
| `/luma` | Luma.vue (341行) | Luma 视频 |
| `/keling` | KeLing.vue (662行) | 可灵视频 |
| `/member` | Member.vue | 会员充值 |
| `/apps` | ChatApps.vue | 应用中心 |
| `/images-wall` | ImagesWall.vue (412行) | 作品展示 |
| `/invite` | Invitation.vue | 推广计划 |
| `/powerLog` | PowerLog.vue | 消费日志 |
| `/xmind` | MarkMap.vue | 思维导图 |
| `/song/:id` | Song.vue | 音乐播放 |
| `/external` | ExternalPage.vue | 外部链接 |

**独立路由** (无嵌套布局):

| 路由 | 组件 | 说明 |
|------|------|------|
| `/login` | Login.vue | 用户登录 |
| `/login/callback` | LoginCallback.vue | OAuth 回调 |
| `/register` | Register.vue | 用户注册 |
| `/resetpassword` | Resetpassword.vue | 重置密码 |
| `/chat/export` | ChatExport.vue | 对话导出 |
| `/payReturn` | PayReturn.vue | 支付回调 |
| `/test` | Test.vue | 测试页面 |
| `/:all(.*)` | 404.vue | 404 页面 |

**管理后台路由** (`/admin/*`):

| 路由 | 组件 | 说明 |
|------|------|------|
| `/admin/login` | admin/Login.vue | 管理员登录 |
| `/admin/dashboard` | admin/Dashboard.vue | 仪表盘 |
| `/admin/system` | admin/SysConfig.vue (621行) | 系统设置 |
| `/admin/user` | admin/Users.vue (460行) | 用户管理 |
| `/admin/app` | admin/Apps.vue (458行) | 应用管理 |
| `/admin/app/type` | admin/AppType.vue | 应用分类 |
| `/admin/apikey` | admin/ApiKey.vue | API Key 管理 |
| `/admin/chat/model` | admin/ChatModel.vue (439行) | 模型管理 |
| `/admin/product` | admin/Product.vue | 产品管理 |
| `/admin/order` | admin/Order.vue | 订单管理 |
| `/admin/redeem` | admin/Redeem.vue | 兑换码管理 |
| `/admin/loginLog` | admin/LoginLog.vue | 登录日志 |
| `/admin/functions` | admin/Functions.vue | 函数管理 |
| `/admin/chats` | admin/records/ChatList.vue | 对话记录 |
| `/admin/images` | admin/records/ImageList.vue | 绘图记录 |
| `/admin/medias` | admin/records/Medias.vue | 音视频记录 |
| `/admin/powerLog` | admin/PowerLog.vue | 算力日志 |
| `/admin/manger` | admin/Manager.vue | 管理员管理 |

**移动端路由** (`/mobile/*`):

| 路由 | 组件 | 说明 |
|------|------|------|
| `/mobile/login` | mobile/Login.vue | 移动端登录 |
| `/mobile/index` | mobile/Index.vue (333行) | 首页 |
| `/mobile/chat` | mobile/ChatList.vue (281行) | 对话列表 |
| `/mobile/chat/session` | mobile/ChatSession.vue (606行) | 对话会话 |
| `/mobile/chat/export` | mobile/ChatExport.vue | 对话导出 |
| `/mobile/image` | mobile/Image.vue | 绘图入口 |
| `/mobile/apps` | mobile/Apps.vue (268行) | 应用中心 |
| `/mobile/profile` | mobile/Profile.vue (393行) | 个人中心 |
| `/mobile/imgWall` | mobile/pages/ImgWall.vue | 作品墙 |

### HTTP 请求封装 (utils/http.js)

```javascript
import { httpGet, httpPost, httpDownload, httpPostDownload } from '@/utils/http'

// GET 请求
httpGet('/api/user/profile').then(res => {...})

// POST 请求
httpPost('/api/chat/create', { title: '新对话' }).then(res => {...})

// 文件下载 (GET)
httpDownload('/api/download?path=xxx').then(res => {...})

// 文件下载 (POST)
httpPostDownload('/api/admin/redeem/export', data).then(res => {...})
```

**Axios 配置**:
- `baseURL`: `process.env.VUE_APP_API_HOST`
- `timeout`: 180000ms (3分钟)
- `withCredentials`: true
- 请求头自动携带: `Authorization` (用户 Token) + `Admin-Authorization` (管理员 Token)
- 401 响应自动清除 Token

## 关键依赖与配置

### 主要依赖 (package.json)

| 依赖 | 版本 | 用途 |
|------|------|------|
| vue | ^3.2.13 | 核心框架 |
| vue-router | ^4.0.15 | 路由 |
| pinia | ^2.1.4 | 状态管理 |
| element-plus | ^2.4.0 | PC端 UI |
| @element-plus/icons-vue | ^2.3.1 | 图标 |
| vant | ^4.5.0 | 移动端 UI |
| axios | ^0.27.2 | HTTP 请求 |
| echarts | ^5.5.0 | 图表 (管理后台) |
| markdown-it | ^13.0.1 | Markdown 渲染 |
| markdown-it-mathjax3 | ^4.3.2 | LaTeX 数学公式 |
| md-editor-v3 | ^2.2.1 | Markdown 编辑器 |
| highlight.js | ^11.7.0 | 代码高亮 |
| markmap-view/lib/common/toolbar | ^0.16-0.17 | 思维导图 |
| three | ^0.128.0 | 3D 渲染 |
| sortablejs | ^1.15.0 | 拖拽排序 |
| clipboard | ^2.0.11 | 剪贴板 |
| qrcode | ^1.5.3 | 二维码 |
| compressorjs | ^1.2.1 | 图片压缩 |
| animate.css | ^4.1.1 | CSS 动画 |
| vue-waterfall-plugin-next | ^2.6.5 | 瀑布流布局 |
| @better-scroll/core | ^2.5.1 | 移动端滚动 |
| lodash | ^4.17.21 | 工具库 |

### 开发依赖

| 依赖 | 版本 | 用途 |
|------|------|------|
| @vue/cli-service | ~5.0.0 | 构建工具 |
| tailwindcss | ^3.4.17 | 原子 CSS |
| stylus/stylus-loader | ^0.58.1 / ^7.0.0 | CSS 预处理 |
| webpack | ^5.90.3 | 打包器 |
| eslint + eslint-plugin-vue | ^7.32 + ^8.0.3 | 代码检查 |

### 环境变量

```bash
# .env 或 .env.local
VUE_APP_API_HOST=http://localhost:5678  # 后端 API 地址
```

### 构建配置 (vue.config.js)

- `lintOnSave: false` - 关闭保存时 ESLint
- `productionSourceMap: false` - 生产禁用 Source Map
- 路径别名: `@` -> `src/`
- 开发端口: 8888
- 静态资源代理: `/static/upload/` -> API 服务

## 数据模型

### 状态管理 (store/) - 7 个 Store

| Store | 文件 | 说明 |
|-------|------|------|
| session | session.js | 用户 Token + 管理员 Token 管理 |
| system | system.js | 系统配置缓存 |
| theme | theme.js | 主题切换 (dark/light) |
| sidebar | sidebar.js | 侧边栏折叠状态 |
| cache | cache.js | 通用缓存 |
| tags | tags.js | 管理后台标签页 |
| sharedata | sharedata.js | 组件间共享数据 |

### Session Store 使用

```javascript
import { getUserToken, setUserToken, removeUserToken } from '@/store/session'
import { getAdminToken, setAdminToken, removeAdminToken } from '@/store/session'

setUserToken('jwt-token-xxx')
const token = getUserToken()
```

### 主题切换

```javascript
import { useThemeStore } from '@/store/theme'
const themeStore = useThemeStore()
// 初始化时: document.documentElement.setAttribute("data-theme", themeStore.theme)
```

## 目录结构

```
web/
├── src/
│   ├── main.js               # 入口 (124 行)
│   ├── App.vue                # 根组件 (178 行)
│   ├── router.js              # 路由配置 (362 行)
│   ├── views/                 # 页面视图 (28 文件)
│   │   ├── Index.vue          # 首页落地页
│   │   ├── Home.vue           # 主布局
│   │   ├── ChatPlus.vue       # AI 对话 (1525 行, 最大组件)
│   │   ├── ImageMj.vue        # MJ 绘画 (1354 行)
│   │   ├── ImageSd.vue        # SD 绘画 (758 行)
│   │   ├── Dalle.vue          # DALL-E (591 行)
│   │   ├── Suno.vue           # 音乐创作 (652 行)
│   │   ├── KeLing.vue         # 可灵视频 (662 行)
│   │   ├── Luma.vue           # Luma 视频 (341 行)
│   │   ├── MarkMap.vue        # 思维导图 (303 行)
│   │   ├── admin/             # 管理后台 (18 文件)
│   │   │   ├── Home.vue       # 管理后台布局
│   │   │   ├── Dashboard.vue  # 仪表盘
│   │   │   ├── SysConfig.vue  # 系统配置 (621 行)
│   │   │   ├── Users.vue      # 用户管理 (460 行)
│   │   │   └── records/       # 记录管理 (3 文件)
│   │   └── mobile/            # 移动端 (13 文件)
│   │       ├── Home.vue       # 移动端布局
│   │       ├── ChatSession.vue # 对话 (606 行)
│   │       ├── Index.vue      # 首页 (333 行)
│   │       └── pages/         # 子页面
│   ├── components/            # 公共组件 (48 文件)
│   │   ├── ChatPrompt.vue     # 聊天输入 (478 行)
│   │   ├── ChatReply.vue      # 聊天回复 (537 行)
│   │   ├── LoginDialog.vue    # 登录弹窗 (453 行)
│   │   ├── CaptchaPlus.vue    # 验证码 (348 行)
│   │   ├── RealtimeConversation.vue # 实时语音 (338 行)
│   │   ├── SlideCaptcha.vue   # 滑动验证 (306 行)
│   │   ├── MusicPlayer.vue    # 音乐播放器 (282 行)
│   │   ├── admin/             # 管理后台组件 (3 文件)
│   │   ├── mobile/            # 移动端组件 (3 文件)
│   │   └── ui/                # 基础 UI 组件 (7 文件)
│   │       ├── BlackDialog.vue
│   │       ├── BlackInput.vue
│   │       ├── BlackSelect.vue
│   │       ├── BlackSwitch.vue
│   │       ├── Generating.vue
│   │       ├── ItemsInput.vue
│   │       └── RippleButton.vue
│   ├── store/                 # Pinia 状态管理 (7 文件)
│   ├── utils/                 # 工具函数 (7 文件)
│   │   ├── http.js            # HTTP 封装 (100 行)
│   │   ├── libs.js            # 通用工具
│   │   ├── dialog.js          # 对话框工具
│   │   ├── validate.js        # 验证工具
│   │   ├── conversation_config.js # 对话配置
│   │   ├── wav_player.js      # WAV 播放器
│   │   └── wav_renderer.js    # WAV 渲染器
│   ├── assets/                # 静态资源
│   │   ├── css/               # 样式 (32 styl + 18 css)
│   │   │   ├── common.styl    # 公共样式
│   │   │   ├── theme-dark.styl # 深色主题
│   │   │   ├── theme-light.styl # 浅色主题
│   │   │   ├── tailwind.css   # Tailwind 入口
│   │   │   └── mobile/        # 移动端样式
│   │   └── iconfont/          # 图标字体
│   └── lib/                   # 第三方库
│       └── wavtools/          # 音频处理工具 (7 文件)
├── public/                    # 公共静态资源
├── package.json               # 依赖 + ESLint 配置
├── vue.config.js              # Vue CLI 配置
├── tailwind.config.js         # Tailwind 配置
├── postcss.config.js          # PostCSS 配置
├── babel.config.js            # Babel 配置
├── nginx.conf                 # Nginx 配置 (Docker 内用)
└── Dockerfile                 # Docker 多阶段构建
```

## 测试与质量

**测试状态**: 暂无自动化测试

**代码检查**:
```bash
npm run lint
```

**ESLint 配置** (package.json):
- extends: `plugin:vue/vue3-essential`, `eslint:recommended`
- parser: `@babel/eslint-parser`
- lintOnSave: false (开发时关闭)

## 常见问题 (FAQ)

### Q: 如何添加新页面?

1. 在 `src/views/` 创建 Vue 组件
2. 在 `src/router.js` 添加路由配置
3. 如需侧边栏显示，配合后台菜单管理

### Q: 如何添加新组件?

1. 在 `src/components/` 创建组件
2. 在需要的页面中 import 使用
3. 公共组件可在 `main.js` 全局注册

### Q: 如何切换主题?

```javascript
import { useThemeStore } from '@/store/theme'
const themeStore = useThemeStore()
// 主题通过 data-theme 属性控制
// 样式定义在 theme-dark.styl 和 theme-light.styl
```

### Q: 如何调用 API?

```javascript
import { httpGet, httpPost } from '@/utils/http'

// GET 请求 (自动携带 Authorization 和 Admin-Authorization)
const res = await httpGet('/api/chat/list')

// POST 请求
const res = await httpPost('/api/chat/create', { data })
```

### Q: 如何添加移动端页面?

1. 在 `src/views/mobile/` 创建组件
2. 在 `router.js` 的 mobile 子路由中添加
3. 使用 Vant 组件而非 Element Plus
4. 样式放在 `assets/css/mobile/`

### Q: 如何添加管理后台页面?

1. 在 `src/views/admin/` 创建组件
2. 在 `router.js` 的 admin 子路由中添加
3. 使用 Element Plus 组件
4. 管理后台布局在 `admin/Home.vue`

## 相关文件清单

| 类别 | 路径 |
|------|------|
| 入口 | src/main.js |
| 路由 | src/router.js |
| HTTP 封装 | src/utils/http.js |
| 对话页面 | src/views/ChatPlus.vue |
| MJ 绘画 | src/views/ImageMj.vue |
| 可灵视频 | src/views/KeLing.vue |
| 音乐创作 | src/views/Suno.vue |
| 管理后台首页 | src/views/admin/Home.vue |
| 系统配置 | src/views/admin/SysConfig.vue |
| 移动端首页 | src/views/mobile/Home.vue |
| 主题 Store | src/store/theme.js |
| Session Store | src/store/session.js |
| Vue 配置 | vue.config.js |
| Nginx 配置 | nginx.conf |
| Docker 构建 | Dockerfile |

---

## 变更记录 (Changelog)

### 2026-02-21 (增量更新)
- 补充完整路由列表 (PC + 管理后台 + 移动端 共 50+ 路由)
- 补充 HTTP 工具 4 个导出函数 (httpGet/httpPost/httpDownload/httpPostDownload)
- 补充 Axios 配置详情 (baseURL/timeout/interceptors)
- 补充组件行数标注 (最大: ChatPlus.vue 1525 行)
- 补充 Stylus 样式文件统计 (32 styl + 18 css)
- 补充 wavtools 音频处理库
- 补充 RealtimeConversation 语音对话组件
- 更新文件统计: 106 Vue + 25 JS

### 2026-02-07 (初始化)
- 创建模块文档
- 识别 100+ Vue 文件, 25+ JS 文件
- 记录路由结构与状态管理
