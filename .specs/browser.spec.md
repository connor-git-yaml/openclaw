---
type: module-spec
version: 1.0
source: src/browser/
files_analyzed: 52
loc: 10475
last_updated: 2026-02-09
---

# Browser Spec

## 1. Intent (意图)
浏览器自动化模块，提供 Chrome DevTools Protocol (CDP) 集成：
- Chrome 浏览器启动和管理（独立 profile）
- CDP 连接和命令执行
- 截图、DOM 快照、页面导航
- Bridge server（浏览器扩展中继）
- 浏览器配置解析（profile、executable 路径）

## 2. Interface (接口定义)
- `launchChrome(config)` → `RunningChrome` — 启动 Chrome 实例
- `captureScreenshot(opts)` → `Buffer` — CDP 截图
- `normalizeCdpWsUrl(wsUrl, cdpUrl)` — 标准化 CDP WebSocket URL
- `isChromeReachable(cdpUrl)` — 检查 Chrome 是否可达
- `resolveBrowserExecutableForPlatform()` — 跨平台 Chrome 路径发现
- Bridge server 用于浏览器扩展与 Gateway 通信

## 3. Business Logic (业务逻辑)
- **Chrome 启动**: 发现可执行文件 → 创建独立 user-data-dir → 启用 CDP → 等待就绪
- **CDP 通信**: WebSocket 连接 → JSON 消息收发 → 命令超时管理
- **Profile 管理**: OpenClaw 专用 profile → 装饰标记 → 清理退出
- **截图**: Page.enable → 获取布局尺寸 → 截图（支持全页面）

## 4. Data Structures (数据结构)
- `RunningChrome` — 运行中的 Chrome 实例（pid, exe, userDataDir, cdpPort, proc）
- `ResolvedBrowserConfig` — 浏览器配置
- `ResolvedBrowserProfile` — profile 配置
- `BrowserExecutable` — 可执行文件信息

## 5. Constraints (约束)
- 仅支持 Chrome/Chromium
- 需要 CDP（--remote-debugging-port）
- 跨平台：Mac/Linux/Windows 不同的 Chrome 路径
- Profile 存储在 `~/.openclaw/browser/`

## 6. Edge Cases (边界条件)
- Chrome 未安装或不可达
- CDP 端口冲突
- Profile 非正常退出的恢复
- Loopback vs 远程 CDP URL 转换

## 7. Technical Debt (技术债务)
- CDP 通信是手动 WebSocket 实现，可考虑使用 puppeteer/playwright
- 文件数较多（52），部分可合并

## 8. Dependencies (依赖)
- **内部**: config, infra/ports, logging
- **外部**: ws
