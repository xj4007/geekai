[根目录](../CLAUDE.md) > **desktop**

# Desktop 桌面客户端

> 最后更新: 2026-02-07 16:01:46

## 模块职责

基于 Electron 封装的桌面客户端应用，提供:
- 跨平台桌面应用 (Windows, macOS, Linux)
- 封装 Web 应用为原生体验
- 启动加载动画

## 入口与启动

**主入口**: `index.js`

```javascript
// 启动流程:
// 1. 显示 loading 窗口
// 2. 后台加载主应用 URL
// 3. 加载完成后隐藏 loading，显示主窗口
// 4. 最大化主窗口
```

**启动命令**:
```bash
npm install
npm start       # 开发运行
npm run package # 打包发布
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

### 构建配置

```json
{
  "build": {
    "appId": "ai.r9it.com",
    "productName": "ChatGPT-Plus",
    "win": {
      "target": "nsis",
      "icon": "icons/logo.ico"
    },
    "mac": {
      "target": "dmg",
      "icon": "icons/logo.icns"
    },
    "linux": {
      "target": "AppImage",
      "icon": "icons/logo.png"
    }
  }
}
```

## 目录结构

```
desktop/
├── index.js          # Electron 主进程入口
├── loading.html      # 加载页面
├── package.json      # 依赖与构建配置
├── icons/            # 应用图标
│   ├── logo.ico      # Windows 图标
│   ├── logo.icns     # macOS 图标
│   └── logo.png      # Linux 图标
└── .gitignore
```

## 测试与质量

**测试状态**: 暂无自动化测试

## 常见问题 (FAQ)

### Q: 如何修改默认加载的网站?

编辑 `index.js`:
```javascript
mainWindow.loadURL('https://your-new-domain.com');
```

### Q: 如何修改应用名称和图标?

1. 替换 `icons/` 目录下的图标文件
2. 编辑 `package.json` 中的 `build.productName`

### Q: 如何打包发布?

```bash
npm run package
# 输出到 dist/ 目录
```

### Q: 如何启用开发者工具?

在 `index.js` 中添加:
```javascript
mainWindow.webContents.openDevTools();
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

### 2026-02-07 (初始化)
- 创建模块文档
- 记录 Electron 配置与打包信息
