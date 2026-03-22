# MEMORY.md - 长期记忆

## 动画查询网站

- **yuc.wiki** (https://yuc.wiki) - 按季度划分的新番列表，查新番用
- **bangumi** (https://bgm.tv) - 动画详细信息、评分、吐槽

## 已安装的热门技能 (2026-03-13)

- **self-improving** - 自我反思+自我学习+记忆系统
- **news-aggregator-skill** - 新闻聚合（8大来源）
- **daily-trending** - 今日热榜
- **github-trending** - GitHub 热门
- **hackernews** - Hacker News
- **stock-analysis** - 股票/加密货币分析
- **tavily-search** - 联网搜索

## 定时任务

- 每日 23:30 自动工作总结（写入 memory/YYYY-MM-DD.md）
- 每日 8:00 科技新闻推送（已设置，2026-03-14 开始执行）

## 用户偏好

- 用户943826 使用 MiniMax-M2.5 模型
- 对科技新闻、动画、游戏相关内容感兴趣
- 喜欢有锐评、有观点的内容

## 提示词盒子 App (2026-03-21 最终状态)

- **文件位置**: `/home/gem/workspace/agent/workspace/prompt-box-app/index.html`
- **Android 项目**: `prompt-box-android/`（GitHub: Tinyyysky/prompt_box，Actions 自动构建）
- **当前版本**: v1.0.2b（堆叠图片滑动交互）
- **功能**: 13 个 v5 功能 + Action Sheet + 多选模态框 + IndexedDB fallback + 图片堆叠
- **交互逻辑**: 点击=编辑，长按=Action Sheet，多选=底部模态框4宫格
- **模态框统一**: 所有弹窗用 `asOL` + `as` + `asBody`，不用系统 prompt()/confirm()

### Android 打包经验
- 双 Git：飞书 Git (apaas-force-git) 存代码，GitHub (Tinyyysky/prompt_box) 触 Actions
- GitHub API 对 `.github/workflows/` 文件 PUT 返回 400/404，需手动网页操作
- GitHub token: ghp_***REDACTED***
- 本地签名流水线: `/tmp/local_jdk` (JDK 17) + `/tmp/local_sdk/build-tools/34.0.0`
- APP 图标: Pillow resize 5 种密度 (mdpi 48 / hdpi 72 / xhdpi 96 / xxhdpi 144 / xxxhdpi 192)

### 状态栏适配经验
- `fitsSystemWindows(true)` 是正确方案，false 会导致布局错位
- **小米 HyperOS 深色模式**: 标准 API 不生效，需小米私有 API 反射调用 `Window.setStatusBarDarkMode()`
- `env(safe-area-inset-*)` Android WebView 不支持，用 JS 注入 `getStatusBarHeight()`
- 小米14 刘海屏: 状态栏 134px，导航栏 48px
- `position:fixed` 的元素不受 fitsSystemWindows 影响，需手动 padding-top 补偿

### 关键教训 (2026-03-21)
- ⚠️ **写含代码的大文件不要用 bash heredoc**，特殊字符会截断。用 `python3 << 'PYEOF'` 分段写入
- GitHub API 推 100KB+ base64 需要 120s 超时，curl 比 Python urllib 更稳定
- CSS 优先级冲突是反复 bug 源头，应避免重复规则
- 子代理处理大任务可能不完整，主 agent 需兜底

## 解决问题流程（用户指定）

当遇到无法独立解决的问题时：
1. 去 ClaWHub 搜索是否有对应 Skill
2. ClaWHub 没有 → 去 GitHub 找开源项目
3. 使用前必须判断安全性（代码是否可信、是否有恶意行为）

## MiMo TTS 语音合成 (2026-03-20 安装)

- **Skill**: xiaomi-mimo-tts（GitHub: jazzqi/openclaw-mimo-tts）
- **位置**: ~/.openclaw/skills/mimo-tts/
- **API Key**: 已配置在 ~/.openclaw/skills/mimo-tts/.env
- **运行方式**: `export XIAOMI_API_KEY=$(cat ~/.openclaw/skills/mimo-tts/.env | cut -d= -f2) && node scripts/smart/mimo_tts_smart.js "文本" output.ogg`
- **飞书发语音**: ogg 文件需放到 workspace 目录，再用 message(asVoice=true) 发送
- **支持功能**: 情感检测、方言（东北话/四川话/台湾腔/粤语等）、悄悄话、夹子音、唱歌
- **API**: api.xiaomimimo.com/v1/chat/completions, model: mimo-v2-tts

## X Search (ClaWHub 上有)

- ClaWHub 上有 X Search skill（by @Jaaneek），但需要 XAI_API_KEY
- 用户表示不想弄 API key
- 目前可通过 miaoda-studio-cli search-summary 搜索网上关于 X 的讨论内容（非实时推文流）

## 用户偏好补充 (2026-03-20)

- 用户943826 的小米 API Key 已获得（MiMo TTS 用）
- 偏好苹果风格 UI：毛玻璃（glassmorphism）、半透明、模糊背景、柔和阴影、圆角
- 服务器无 Java / Android SDK，无法编译 APK；PWA 是可行替代方案
- 提示词盒子 App 已完成（HTML+CSS+JS 一体文件），支持 PWA 安装到安卓桌面
  - 文件位置：/home/gem/workspace/agent/workspace/prompt-box-app/index.html
  - 经历 7 个版本迭代（v1→v4），核心功能完善
  - **v5 已完成 (2026-03-21)**：700行/60KB，13个新功能全部实现
  - ⚠️ 写大 HTML 文件不要用 bash heredoc，特殊字符会截断。用 Python 脚本 `python3 << 'PYEOF'` 分段写入
  - **Android 化 (2026-03-21)**：WebView 壳 + GitHub Actions CI 构建，本地 zipalign+apksigner 签名
  - **状态栏适配经验 (v13 最终)**：
    - `env(safe-area-inset-*)` Android WebView 不支持，需用 JS 注入 `getStatusBarHeight()`
    - `fitsSystemWindows(true)` 是正确方案 — 系统自动处理子 view 位置
    - **⚠️ `fitsSystemWindows(false)` 会导致布局错位（侧边栏文字太靠上等）**
    - **小米 HyperOS 深色模式**：标准 `SYSTEM_UI_FLAG_LIGHT_STATUS_BAR` 不生效，需用小米私有 API：
      - `Window.class.getMethod("setStatusBarDarkMode", boolean.class)` 反射调用
      - 配合固体色状态栏 `setStatusBarColor(#1C1C1E)` — 小米能基于色块自动处理图标颜色
    - `position:fixed` 的元素（侧边栏等）不受 `fitsSystemWindows` 影响，需要在 JS 注入时用 `padding-top:max(50px, statusBarHeight)` 手动补偿
    - GitHub API 对 `.github/workflows/` 下文件 PUT 返回 400/404，源码文件正常
    - 本地签名流水线：/tmp/local_jdk (JDK 17) + /tmp/local_sdk/build-tools/34.0.0 → zipalign → apksigner
    - **双 Git 仓库**：飞书 Git (apaas-force-git) 存代码，GitHub (Tinyyysky/prompt_box) 触 Actions 构建。需要两个都推
    - GitHub token: ghp_***REDACTED***
    - GitHub API 推大文件（100KB+ base64）需要 120s 超时
    - 文件中中文字符 vs unicode escape 要注意匹配方式
  - **v1.0.5 (2026-03-22)**：clipboard 兼容 + 错误处理加固
    - **⚠️ Android WebView `navigator.clipboard` 不可用**（无 HTTPS），调用 `.writeText()` 抛 TypeError，不是 promise reject
    - 解决方案：`clipCopy()` 函数，fallback 到 `document.execCommand('copy')`
    - **Service Worker 缓存陷阱**：SW 缓存旧版 HTML，用户看到的是缓存版本。必须 bump SW cache version 才能触发更新
    - **GitHub Contents API**：`curl -d @file` 可能 400，改 `--data-binary @file` 解决
  - **交互逻辑（v14.2+ 最终）**：点击=编辑，长按=Action Sheet，多选=底部模态框4宫格
  - **模态框统一**：所有弹窗用 `asOL` + `as` + `asBody`，不要用系统 prompt()/confirm()
  - **APP 图标制作经验**：Pillow resize + 5 种密度（mdpi 48/hdpi 72/xhdpi 96/xxhdpi 144/xxxhdpi 192），放 res/mipmap-*/ic_launcher.png
  - **设备数据**：小米 14 (HyperOS3) 刘海屏状态栏 134px，导航栏 48px
  - **GitHub API 限制**：workflow 文件无法通过 API 修改，需手动在网页操作
- 子代理处理大任务时可能不完整，需主 agent 兜底完成
- 科技新闻推送 cron 任务需排查 798 错误


## 提示词盒子项目 (2026-03-20~22) — 经验教训

- ⚠️ 以后项目必须加入自动化 UI 测试，不能让用户帮忙测 bug
- CSS 变量未定义时引用不会报错、静默失效，需注意
- `\u00D7` 等 JS Unicode 转义不能在 HTML 正文用，必须用 HTML 实体 `&#215;`
- Android WebView 不支持 `env(safe-area-inset-*)`，需 JS 注入状态栏高度
- Android WebView 键盘是覆盖模式，position:fixed 不自动抬升，需 translateY 补偿
- 小米 HyperOS 深色模式需反射调用 Window.setStatusBarDarkMode()
- GitHub API 对 .github/workflows/ 文件 PUT 返回 400/404
- GitHub API 推 100KB+ base64 需要 120s 超时
- 写含代码的大文件不要用 bash heredoc，特殊字符会截断，用 python3
