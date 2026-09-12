# Tauri vs Electron 详细对比

## 1. 核心架构

**Electron**
- 内置完整的 Chromium 渲染引擎 + Node.js 运行时
- 每个应用都打包一份完整浏览器内核
- 前端(渲染进程)与后端(主进程)都用 JavaScript/TypeScript 编写

**Tauri**
- 使用操作系统自带的 WebView(Windows 上是 WebView2/Edge 内核,macOS 是 WKWebView/Safari 内核,Linux 是 WebKitGTK)
- 后端用 Rust 编写,前端可以是任意 Web 框架(React/Vue/Svelte 等)
- 前后端通过 IPC(命令系统)通信,类似 RPC 调用

## 2. 包体积

| | Electron | Tauri |
|---|---|---|
| 最小应用体积 | 通常 80-150MB+ | 通常 3-10MB |
| 原因 | 捆绑完整 Chromium + Node | 复用系统 WebView,只打包应用代码 |

这是 Tauri 最大的卖点之一,尤其对于分发和更新场景优势明显。

## 3. 内存与性能

- **Electron**:每个窗口本质是独立的 Chromium 实例,内存占用高(空应用常见 100-200MB+)
- **Tauri**:共享系统 WebView,内存占用显著更低(空应用常见 30-60MB),Rust 后端本身性能和内存效率也优于 Node.js

但要注意:Tauri 的性能优势主要体现在启动速度、内存、包体积上;渲染性能仍取决于系统 WebView 的实现,不同平台间可能有细微差异(尤其 Linux 的 WebKitGTK 有时功能滞后)。

## 4. 跨平台一致性

- **Electron**:因为捆绑固定版本 Chromium,所有平台渲染表现高度一致,几乎不用担心浏览器兼容性问题
- **Tauri**:依赖各平台系统 WebView,版本和特性可能不一致——比如 Windows 用户系统 WebView2 版本较老时,某些 CSS/JS 特性可能不支持,需要额外测试

这是 Tauri 的一个真实痛点,做严格 UI 还原的项目需要注意。

## 5. 开发语言与生态

**Electron**
- 全 JavaScript/TypeScript 技术栈,前端开发者上手成本极低
- 生态成熟,Node.js 生态包(npm)可直接用于主进程,能力(文件系统、原生模块等)非常齐全
- 社区庞大,踩过的坑基本都有解决方案

**Tauri**
- 后端需要写 Rust(即使只是调用系统 API),对纯前端背景的开发者有一定学习曲线
- 插件生态相对年轻,但增长很快(官方和社区插件覆盖文件系统、通知、更新、深度链接等常见需求)
- 如果团队愿意投入 Rust 学习成本,能获得更好的安全性和性能

## 6. 安全模型

- **Tauri**:默认更安全,采用"权限白名单"(allowlist/capabilities)机制,渲染进程默认无法访问系统 API,必须显式声明和调用 Rust 命令;沙箱边界更清晰
- **Electron**:早期版本默认开启 nodeIntegration 存在较大安全风险,现在官方推荐关闭 nodeIntegration + 启用 contextIsolation + 用 preload 脚本暴露有限 API,但需要开发者主动配置,容易配置疏忽

## 7. 成熟度与稳定性

- **Electron**:2013 年发布,久经考验,VS Code、Slack、Discord、Figma(部分)、WhatsApp Desktop 等大量知名应用都在用,长期维护、文档丰富、招聘市场认可度高
- **Tauri**:2019 年起步,2024 年发布 2.0 大版本(支持移动端 iOS/Android),发展迅速但相对年轻,大规模生产案例还在积累中,遇到冷门问题时可参考资料较少

## 8. 移动端支持

- **Electron**:仅支持桌面(Windows/macOS/Linux)
- **Tauri 2.0+**:已支持构建 iOS 和 Android 应用,是相对独特的优势,一套代码理论上可覆盖桌面+移动

## 9. 构建与打包体验

- **Electron**:electron-builder / electron-forge 成熟稳定,打包配置项丰富
- **Tauri**:自带 CLI 打包工具,依赖系统工具链(Rust toolchain、平台 SDK),Windows 上打包需要 MSVC 环境,首次配置略繁琐,但打包出的安装包(msi/dmg/AppImage 等)体积小很多

## 10. 调试与开发体验

- **Electron**:直接用 Chrome DevTools,调试体验和 Web 前端几乎一致,非常成熟
- **Tauri**:调试依赖系统 WebView 的开发者工具(不同平台调用方式略有差异),Rust 部分调试需要额外配置(如 `println!`、`log` crate 或 IDE 调试器),整体调试链路比 Electron 稍复杂

---

## 总结建议

| 场景 | 更适合 |
|---|---|
| 追求极致包体积/内存占用、愿意投入 Rust 学习成本、需要移动端 | **Tauri** |
| 团队全是前端背景、需要成熟稳定生态、依赖大量 Node 原生模块、要求跨平台渲染绝对一致 | **Electron** |
| 需要与已有 Node.js 后端服务深度集成 | **Electron** 更顺手 |
| 新项目、对安全模型要求高、长期维护考虑体积 | **Tauri** 更有前景 |
