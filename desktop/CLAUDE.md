[根目录](../CLAUDE.md) > **desktop**

# Desktop 桌面客户端

> 最后更新: 2026-02-21 23:00:18

## 模块职责

基于 Electron 封装的桌面客户端应用，提供:
- 跨平台桌面应用 (Windows, macOS, Linux)
- 封装 Web 应用为原生窗口体验
- 启动加载动画 (loading 窗口)
- 窗口最大化自动切换

## 入口与启动

**主入口**: `index.js` (29 行)

```javascript
// 启动流程:
// 1. app.on('ready') 触发
// 2. 创建 loading 窗口 (400x300, 无边框)
// 3. 创建主窗口 (1x1, 隐藏)
// 4. 主窗口加载远程 URL (https://ai.r9it.com)
// 5. loading 窗口加载 loading.html
// 6. 隐藏应用菜单栏
// 7. 主窗口 did-finish-load 事件:
//    - 最大化主窗口
//    - 显示主窗口
//    - 关闭 loading 窗口
```

**启动命令**:
```bash
npm install
npm start       # electron . (开发运行)
npm run package # electron-builder (打包发布)
```

## 对外接口

桌面客户端本身不提供 API，作为 Web 应用的容器。

**默认加载 URL**: `https://ai.r9it.com`

如需修改，编辑 `index.js`:
```javascript
mainWindow.loadURL('https://your-domain.com');
```

## 关键依赖与配置

### 主要依赖 (package.json)

| 依赖 | 版本 | 用途 |
|------|------|------|
| electron | ^26.1.0 | Electron 框架 |
| electron-builder | ^24.6.4 | 打包工具 |

### 构建配置 (package.json build 字段)

```json
{
  "appId": "ai.r9it.com",
  "productName": "ChatGPT-Plus",
  "directories": { "output": "dist" },
  "files": ["index.js", "package.json"],
  "win":   { "target": "nsis",     "icon": "icons/logo.ico" },
  "mac":   { "target": "dmg",      "icon": "icons/logo.icns" },
  "linux": { "target": "AppImage", "icon": "icons/logo.png" }
}
```

## 目录结构

```
desktop/
├── index.js          # Electron 主进程入口 (29 行)
├── loading.html      # 加载动画页面
├── package.json      # 依赖、脚本与构建配置
├── icons/            # 应用图标
│   ├── logo.ico      # Windows 图标
│   ├── logo.icns     # macOS 图标
│   └── logo.png      # Linux 图标
└── .gitignore
```

**总文件数**: 3 个源文件 + 3 个图标

## 测试与质量

**测试状态**: 暂无自动化测试

package.json 中 test 脚本为占位:
```bash
npm test  # echo "Error: no test specified" && exit 1
```

## 常见问题 (FAQ)

### Q: 如何修改默认加载的网站?

编辑 `index.js`:
```javascript
mainWindow.loadURL('https://your-new-domain.com');
```

### Q: 如何修改应用名称和图标?

1. 替换 `icons/` 目录下的图标文件
2. 编辑 `package.json` 中的 `build.productName` 和 `build.appId`

### Q: 如何打包发布?

```bash
npm run package
# 输出到 dist/ 目录
# Windows: .exe (NSIS 安装包)
# macOS: .dmg
# Linux: .AppImage
```

### Q: 如何启用开发者工具?

在 `index.js` 中添加:
```javascript
mainWindow.webContents.openDevTools();
```

### Q: 如何修改窗口大小?

修改 `index.js` 中 BrowserWindow 的参数:
```javascript
const mainWindow = new BrowserWindow({
    width: 1200,
    height: 800,
});
// 默认行为: 初始 1x1 隐藏, 加载完成后 maximize()
```

## 相关文件清单

| 类别 | 路径 |
|------|------|
| 主入口 | index.js |
| 加载页 | loading.html |
| 配置 | package.json |
| Windows 图标 | icons/logo.ico |
| macOS 图标 | icons/logo.icns |
| Linux 图标 | icons/logo.png |

---

## 变更记录 (Changelog)

### 2026-02-21 (增量更新)
- 补充启动流程详细步骤
- 补充构建配置详情 (各平台打包格式)
- 补充窗口行为说明 (loading -> maximize)
- 确认模块无代码变更

### 2026-02-07 (初始化)
- 创建模块文档
- 记录 Electron 配置与打包信息
