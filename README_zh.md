<p align="center">
  <a href="README.md">English</a> | <b>中文</b>
</p>

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://github.com/CloakHQ/CloakBrowser/raw/main/images/logo.png">
    <img alt="CloakBrowser" src="https://github.com/CloakHQ/CloakBrowser/raw/main/images/logo.png" width="400">
  </picture>
</p>

# CloakBrowser — 隐形 Chromium 浏览器

**通过所有机器人检测测试的隐身浏览器**。在 Chromium C++ 源码层面打了 58 个指纹补丁，是 Playwright 的即插即用替代品。30/30 测试全部通过。

> Fork 自 [CloakHQ/CloakBrowser](https://github.com/CloakHQ/CloakBrowser)
> 🌐 [官网](https://cloakbrowser.dev/)

---

## 目录

- [为什么需要 CloakBrowser](#为什么需要-cloakbrowser)
- [核心特性](#核心特性)
- [快速开始](#快速开始)
- [安装](#安装)
- [API 参考](#api-参考)
  - [浏览器启动](#浏览器启动)
  - [启动上下文](#启动上下文)
  - [持久化配置](#持久化配置)
  - [工具函数](#工具函数)
- [CLI 命令](#cli-命令)
- [拟人化行为](#拟人化行为)
- [Docker 部署](#docker-部署)
  - [cloakserve — CDP 多路复用器](#cloakserve--cdp-多路复用器)
- [框架集成](#框架集成)
- [平台支持](#平台支持)
- [测试结果](#测试结果)
- [许可说明](#许可说明)
- [常见问题](#常见问题)

---

## 为什么需要 CloakBrowser

现有的无头浏览器反检测方案大多存在问题：

| 方案 | 问题 |
|------|------|
| `playwright-stealth`、`undetected-chromedriver` | 通过 JS 注入修改指纹，每次 Chrome 更新就失效 |
| 指纹浏览器（Multilogin/GoLogin/AdsPower） | 商业闭源，成本高 |

**CloakBrowser 的不同之处：** 在 **Chromium C++ 源码级别**打了 58 个补丁，直接编译进浏览器二进制文件。它伪装成真正的 Google Chrome，在底层通过所有检测。

---

## 核心特性

- **58 个源码级补丁** — Canvas、WebGL、Audio、Fonts、GPU、Screen、WebRTC 等指纹全覆盖
- **即插即用** — Playwright/Puppeteer 的替代品，修改一行代码即可
- **拟人化操作** — 贝塞尔曲线鼠标移动、逐字模拟键盘输入、自然滚动
- **CDP 多路复用** — 单端口多指纹实例，每个连接可独立设置时区/语言/代理
- **Docker 支持** — 预装浏览器，开箱即用
- **双语言** — Python + TypeScript 完整支持
- **自动更新** — 首次使用自动下载浏览器（~200MB），支持版本检测

---

## 快速开始

```python
from cloakbrowser import launch

browser = launch(headless=False)
page = browser.new_page()
page.goto("https://bot.sannysoft.com")
page.screenshot(path="result.png")
browser.close()
```

拟人化模式（鼠标/键盘/滚动更像真人）：

```python
from cloakbrowser import launch

browser = launch(humanize=True)
page = browser.new_page()
page.goto("https://example.com")
page.click("#submit")  # 贝塞尔曲线鼠标移动
page.fill("#search", "hello")  # 逐字输入，偶尔打错
browser.close()
```

---

## 安装

### Python

```bash
pip install cloakbrowser
```

### Node.js

```bash
npm install cloakbrowser
```

首次导入时会自动下载 Chromium 浏览器到 `~/.cloakbrowser/`。

---

## API 参考

### 浏览器启动

#### `launch(headless=False, **kwargs)` / `launch_async()`

启动隐身浏览器，返回 Playwright `Browser` 实例。

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `headless` | bool | `False` | 是否无头模式 |
| `humanize` | bool | `False` | 启用拟人化行为 |
| `human_config` | dict | `None` | 拟人化参数配置 |
| `proxy` | dict/str | `None` | 代理配置（支持 HTTP/SOCKS5） |
| `extension_paths` | list | `None` | Chrome 扩展路径列表 |
| `geoip` | bool | `False` | 根据代理 IP 自动设置时区和语言 |
| `disable_web_security` | bool | `False` | 禁用 Web 安全策略 |
| `extra_args` | list | `None` | 额外的 Chrome CLI 参数 |

```python
browser = launch(
    humanize=True,
    proxy={"server": "http://user:pass@host:port"},
    geoip=True,
)
```

---

### 启动上下文

#### `launch_context(**kwargs)` / `launch_context_async()`

一步完成浏览器启动 + 上下文创建，支持直接设置 viewport/UA/时区/语言。

```python
context, browser = launch_context(
    viewport={"width": 1920, "height": 1080},
    user_agent="Mozilla/5.0 ...",
    timezone_id="Asia/Shanghai",
    locale="zh-CN",
)
page = context.new_page()
```

---

### 持久化配置

#### `launch_persistent_context(user_data_dir, **kwargs)` / `launch_persistent_context_async()`

使用持久化用户数据目录启动，保持登录态、Cookie、LocalStorage 跨会话。

```python
context = launch_persistent_context("./my-profile")
page = context.new_page()
# 登录...
context.close()

# 下次启动，登录态还在
context = launch_persistent_context("./my-profile")
```

---

### 工具函数

| 函数 | 说明 |
|------|------|
| `ensure_binary()` | 下载并缓存 Chromium 浏览器 |
| `clear_cache()` | 清除缓存的浏览器文件 |
| `binary_info()` | 获取浏览器版本、路径、平台信息 |
| `check_for_update()` | 检查是否有更新的浏览器版本 |

---

## CLI 命令

```bash
# 下载浏览器（首次使用）
python -m cloakbrowser install

# 查看版本信息
python -m cloakbrowser info

# 检查更新
python -m cloakbrowser update

# 清除缓存
python -m cloakbrowser clear-cache
```

---

## 拟人化行为

`humanize=True` 启用三层真人模拟：

| 模块 | 行为 |
|------|------|
| **鼠标** | 贝塞尔曲线轨迹移动，非直线；click/hover/dblclick 均有自然轨迹 |
| **键盘** | 逐字输入，间隔随机延迟（30~120ms）；偶尔打错字并自动修正 |
| **滚动** | 加速→巡航→减速三段式滚动，符合人眼追踪规律 |

### 预设

```python
# 默认模式（平衡速度和拟真度）
browser = launch(humanize=True)

# "认真"模式（更慢但更像人）
browser = launch(humanize=True, human_config="careful")
```

### 自定义参数

```python
from cloakbrowser.human import HumanConfig

config = HumanConfig(
    typing_delay=(40, 100),      # 打字延迟范围 ms
    mistype_chance=0.02,         # 打错字概率 2%
    mouse_speed=0.8,             # 鼠标速度 0~1
    scroll_step=50,              # 滚动步长 px
)
browser = launch(humanize=True, human_config=config)
```

---

## Docker 部署

预构建镜像，含完整的 Chromium + Xvfb 虚拟显示：

```bash
# 运行隐身测试
docker run --rm cloakhq/cloakbrowser cloaktest

# 启动 CDP 服务器
docker run -p 9222:9222 cloakhq/cloakbrowser cloakserve
```

### cloakserve — CDP 多路复用器

单端口管理多个浏览器实例，每个连接独立指纹：

```bash
docker run -p 9222:9222 cloakhq/cloakbrowser cloakserve
```

通过 WebSocket 连接，支持查询参数设置指纹：

```
ws://localhost:9222/devtools?fingerprint=myseed&timezone=Asia/Shanghai&locale=zh-CN
```

支持的参数：

| 参数 | 说明 |
|------|------|
| `fingerprint` | 指纹种子（连接复用，相同种子共享实例） |
| `timezone` | 时区（如 `Asia/Shanghai`） |
| `locale` | 语言（如 `zh-CN`） |
| `platform` | 操作系统（`linux`、`macos`、`windows`） |
| `proxy` | 代理地址 |
| `gpu-vendor` | GPU 厂商 |
| `screen-width` / `screen-height` | 屏幕分辨率 |
| `hardware-concurrency` | CPU 核心数 |
| `device-memory` | 设备内存 GB |

内置健康检查：`GET http://localhost:9222/` 返回进程状态 JSON。

---

## 框架集成

CloakBrowser 可与以下框架无缝集成：

| 框架 | 连接方式 | 文档 |
|------|---------|------|
| **Playwright** | 直接替代（一行代码修改） | — |
| **Puppeteer** | 直接替代 | `examples/integrations/` |
| **browser-use** | CDP 端口 9242 | `examples/integrations/browser_use_example.py` |
| **Crawl4AI** | CDP 端口 9243 | `examples/integrations/crawl4ai_example.py` |
| **Crawlee** | 自定义 Plugin 替换 Chromium | `examples/integrations/crawlee_example.py` |
| **Scrapling** | CDP WebSocket URL | `examples/integrations/scrapling_example.py` |
| **Selenium** | ChromeDriver 指定二进制路径 | `examples/integrations/selenium_example.py` |
| **LangChain** | 自定义 Document Loader | `examples/integrations/langchain_loader.py` |
| **Stagehand** | npm 包 `stagehand` + 自定义 launch | `js/examples/stagehand.ts` |
| **AgentBrowser** | 环境变量指定路径 | `examples/integrations/agent_browser.sh` |
| **AWS Lambda** | 容器镜像运行 | `examples/integrations/aws_lambda/` |

---

## 平台支持

| 平台 | 架构 | Chromium 版本 | 补丁数 | 状态 |
|------|------|--------------|--------|------|
| Linux | x86_64 | 146 | 58 | ✅ 最新 |
| Linux | arm64 (RPi/Graviton) | 146 | 58 | ✅ 最新 |
| macOS | arm64 (Apple Silicon) | 145 | 26 | ✅ |
| macOS | x86_64 (Intel) | 145 | 26 | ✅ |
| Windows | x86_64 | 146 | 58 | ✅ 最新 |

Nix/NixOS 支持：提供 `flake.nix`，可直接 `nix run`。

---

## 测试结果

与普通 Playwright 的对比：

| 检测项 | Playwright | CloakBrowser |
|--------|-----------|-------------|
| reCAPTCHA v3 得分 | 0.1（机器人） | **0.9（人类）** |
| Cloudflare Turnstile | ❌ 失败 | ✅ 通过 |
| FingerprintJS | ❌ 检测到 | ✅ 通过 |
| BrowserScan | ❌ 检测到 | ✅ 正常 |
| bot.incolumitas.com | 13 项失败 | **1 项失败** |
| deviceandbrowserinfo.com | 6 项异常 | **0 项异常** |
| navigator.webdriver | `true` | **`false`**（源码补丁） |
| TLS 指纹 | 不匹配 | **与 Chrome 一致** |

---

## 许可说明

CloakBrowser 采用双重许可：

**包装代码（Python/JS 库）：** MIT 许可
- 可自由使用、修改、分发

**编译的 Chromium 二进制文件：** Binary License
- **个人和商业免费** — 无需付费
- 不可重新分发、转售、逆向工程
- 内部使用（Docker、CI 等）允许
- OEM/SaaS 嵌入分发需要商业许可
- 基于 ungoogled-chromium，无遥测
- 联系 `cloakhq@pm.me` 获取 OEM/SaaS 许可

---

## 常见问题

**Q: 和 undetected-chromedriver 有什么区别？**

undetected-chromedriver 通过 JS 注入修改指纹，CloakBrowser 在 Chromium C++ 源码层修改，更底层、更稳定、更隐蔽。

**Q: 首次运行需要下载什么？**

首次 `import cloakbrowser` 时会自动下载 Chromium 浏览器（~200MB）到 `~/.cloakbrowser/`。

**Q: 支持 Selenium 吗？**

支持。参考 `examples/integrations/selenium_example.py`。

**Q: 生产环境推荐怎么用？**

推荐使用 Docker 部署 `cloakhq/cloakbrowser` + `cloakserve` 做 CDP 多路复用，每连接独立指纹。

**Q: reCAPTCHA v3 得分低怎么办？**

- 使用 `humanize=True`
- 配置高质量住宅代理
- 使用持久化用户目录（`launch_persistent_context`）
- 安装 Linux 字体包（Kasada/Akamai 检测字体指纹）

---

<div align="center">

**CloakBrowser** — 让自动化浏览器隐形

</div>
