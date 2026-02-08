[根目录](../CLAUDE.md) > **web**

# Web 前端应用

> 最后更新: 2026-02-07 16:01:46

## 模块职责

Vue 3 实现的前端应用，提供:
- PC 端 AI 对话界面 (Element Plus)
- 移动端适配界面 (Vant)
- 管理后台界面
- AI 绘画/音乐/视频任务管理
- 用户中心与会员充值
- 深色/浅色主题切换

## 入口与启动

**主入口**: `src/main.js`

```javascript
// 启动流程:
// 1. 创建 Vue 应用实例
// 2. 注册 Element Plus (PC端)
// 3. 注册 Vant 组件 (移动端)
// 4. 配置 Pinia 状态管理
// 5. 配置 Vue Router
// 6. 挂载应用到 #app
```

**启动命令**:
```bash
npm install
npm run dev    # 开发模式 (热重载)
npm run build  # 生产构建
npm run lint   # 代码检查
```

## 对外接口

### 路由结构 (router.js)

| 路由 | 组件 | 说明 |
|------|------|------|
| `/` | Index.vue | 首页 |
| `/chat` | ChatPlus.vue | AI 对话 |
| `/mj` | ImageMj.vue | MidJourney 绘画 |
| `/sd` | ImageSd.vue | Stable Diffusion |
| `/dalle` | Dalle.vue | DALL-E 绘画 |
| `/suno` | Suno.vue | Suno 音乐 |
| `/luma` | Luma.vue | Luma 视频 |
| `/keling` | KeLing.vue | 可灵视频 |
| `/member` | Member.vue | 会员充值 |
| `/apps` | ChatApps.vue | 应用中心 |
| `/xmind` | MarkMap.vue | 思维导图 |
| `/admin/*` | admin/*.vue | 管理后台 |
| `/mobile/*` | mobile/*.vue | 移动端页面 |

### HTTP 请求封装 (utils/http.js)

```javascript
import { httpGet, httpPost } from '@/utils/http'

// GET 请求
httpGet('/api/user/profile').then(res => {...})

// POST 请求
httpPost('/api/chat/create', { title: '新对话' }).then(res => {...})
```

## 关键依赖与配置

### 主要依赖 (package.json)

| 依赖 | 版本 | 用途 |
|------|------|------|
| vue | ^3.2.13 | 核心框架 |
| vue-router | ^4.0.15 | 路由 |
| pinia | ^2.1.4 | 状态管理 |
| element-plus | ^2.4.0 | PC端 UI |
| vant | ^4.5.0 | 移动端 UI |
| axios | ^0.27.2 | HTTP 请求 |
| echarts | ^5.5.0 | 图表 |
| markdown-it | ^13.0.1 | Markdown 渲染 |
| highlight.js | ^11.7.0 | 代码高亮 |
| tailwindcss | ^3.4.17 | 样式 |

### 环境变量

```bash
# .env 或 .env.local
VUE_APP_API_HOST=http://localhost:5678  # API 地址
```

## 数据模型

### 状态管理 (store/)

| Store | 文件 | 说明 |
|-------|------|------|
| session | session.js | 用户 Token 管理 |
| system | system.js | 系统配置 |
| theme | theme.js | 主题切换 |
| sidebar | sidebar.js | 侧边栏状态 |
| cache | cache.js | 缓存管理 |
| tags | tags.js | 标签页管理 |
| sharedata | sharedata.js | 共享数据 |

### Session Store 示例

```javascript
// 获取/设置用户 Token
import { getUserToken, setUserToken, removeUserToken } from '@/store/session'

setUserToken('jwt-token-xxx')
const token = getUserToken()
```

## 目录结构

```
web/
├── src/
│   ├── main.js           # 入口文件
│   ├── App.vue           # 根组件
│   ├── router.js         # 路由配置
│   ├── views/            # 页面视图
│   │   ├── Home.vue      # 主布局
│   │   ├── ChatPlus.vue  # AI 对话
│   │   ├── ImageMj.vue   # MJ 绘画
│   │   ├── Suno.vue      # Suno 音乐
│   │   ├── admin/        # 管理后台页面
│   │   └── mobile/       # 移动端页面
│   ├── components/       # 公共组件
│   │   ├── ChatPrompt.vue
│   │   ├── ChatReply.vue
│   │   ├── LoginDialog.vue
│   │   ├── admin/        # 管理后台组件
│   │   ├── mobile/       # 移动端组件
│   │   └── ui/           # 基础 UI 组件
│   ├── store/            # Pinia 状态管理
│   ├── utils/            # 工具函数
│   │   ├── http.js       # HTTP 封装
│   │   ├── libs.js       # 通用工具
│   │   └── dialog.js     # 对话框工具
│   ├── assets/           # 静态资源
│   │   ├── css/          # 样式文件
│   │   └── iconfont/     # 图标字体
│   └── lib/              # 第三方库
│       └── wavtools/     # 音频处理
├── public/
├── package.json
├── vue.config.js
└── tailwind.config.js
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
themeStore.toggleTheme()  // 切换深色/浅色
```

### Q: 如何调用 API?

```javascript
import { httpGet, httpPost } from '@/utils/http'

// GET 请求
const res = await httpGet('/api/chat/list')

// POST 请求
const res = await httpPost('/api/chat/create', { data })
```

## 相关文件清单

| 类别 | 路径 |
|------|------|
| 入口 | src/main.js |
| 路由 | src/router.js |
| HTTP 封装 | src/utils/http.js |
| 对话页面 | src/views/ChatPlus.vue |
| 管理后台首页 | src/views/admin/Home.vue |
| 移动端首页 | src/views/mobile/Home.vue |
| 主题 Store | src/store/theme.js |
| Session Store | src/store/session.js |

---

## 变更记录 (Changelog)

### 2026-02-07 (初始化)
- 创建模块文档
- 识别 100+ Vue 文件, 25+ JS 文件
- 记录路由结构与状态管理
