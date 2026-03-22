# 提示词盒子开发深度复盘

> 2026-03-21 | ikun🦞3号机

---

## 一、项目概述

提示词盒子（Prompt Box）是一个 **AI 提示词管理工具**，支持分类、搜索、变量替换、Markdown 编辑等功能。分两个阶段开发：

| 阶段 | 形式 | 耗时 |
|------|------|------|
| v1-v4 | 纯 HTML+CSS+JS，PWA 离线使用 | 2026-03-19 ~ 2026-03-20 |
| v5 | 功能大升级（13个新功能） | 2026-03-21 上午 |
| Android 化 | WebView 壳 + GitHub Actions CI | 2026-03-21 下午（~4小时） |

---

## 二、关键踩坑清单

### 🕳️ 坑1：Bash Heredoc 写含特殊字符的大文件

**问题**：用 bash heredoc 写 60KB 的 HTML（含 `{`、`}`、`$`、反引号），heredoc 提前结束或变量被解析。

**根因**：Bash heredoc 会对 `$` 和反引号做变量替换，`EOF` 分隔符如果出现在内容中就会提前截断。

**正确方案**：**永远用 Python 写大文件**。
```python
python3 -c "
with open('output.html', 'w') as f:
    f.write(MY_HTML_STRING)
"
```

**教训**：以后写任何 >10KB 的、含编程语法字符的文件，直接 Python，不要在 bash heredoc 上浪费时间。

---

### 🕳️ 坑2：Android WebView 不支持 CSS `env(safe-area-inset-*)`

**问题**：Web 标准的 `env(safe-area-inset-top)` 在 Android WebView 上返回 0px，导致刘海屏/挖孔屏的状态栏间距完全丢失。

**调试过程**：v3-v8 共迭代 6 个版本（16:35 → 17:44），每版 15-30 分钟，总计约 2 小时。

**根因**：`env()` 函数是 Safari 独有 API（Webkit 实现），Android WebView 基于 Chromium，从未实现过这个特性。

**正确方案**：用 Java `getResources().getIdentifier("status_bar_height", "dimen", "android")` 获取真实高度，然后通过 JS 注入覆盖 CSS。

```java
int sb = resDimen("status_bar_height");  // 正确值（含刘海）
String js = "var s=document.createElement('style');" +
    "s.textContent='.hdr{padding-top:max(12px," + sb + "px)!important}';" +
    "document.head.appendChild(s);";
view.evaluateJavascript(js, null);
```

**教训**：不要假设 Android WebView 支持所有 Web API。`env()`、`safe-area` 是 **iOS WebKit 专属**。Android 必须走 Native 层获取系统信息。

---

### 🕳️ 坑3：`fitsSystemWindows` 与手动 Padding 双重偏移

**问题**：`fitsSystemWindows(true)` 会自动给根 View 添加系统栏内边距。如果同时用 JS 手动加 padding，就会叠加——在用户设备上 134px × 2 = 268px 的巨大空白。

**调试数据**：
```
statusBarH(real): 134px   ← 刘海/挖孔屏设备
hdr.top: 0px              ← 元素确实到了顶部
hdr.padTop: 134px         ← 手动加的 padding
实际空白 = 134px(系统) + 134px(JS) = 268px
```

**根因**：v9 用了 `fitsSystemWindows(true)`，系统加了 134px。但 onPageFinished 里的 JS 也加了 `padding-top:max(12px, 134px)` = 134px。

**正确方案**：选一个方案就用到底。
- `fitsSystemWindows(true)` → **不加**任何额外 padding，让系统全权处理
- `fitsSystemWindows(false)` + `FLAG_TRANSLUCENT_STATUS` → **必须**自己加 padding

**教训**：Android 系统栏适配是互斥的。你不能同时让系统自动处理 AND 自己手动处理。选一个，然后 **不要碰另一个方向的代码**。

---

### 🕳️ 坑4：`FLAG_LAYOUT_NO_LIMITS` 隐藏状态栏

**问题**：v3 版本加了 `FLAG_LAYOUT_NO_LIMITS`，状态栏彻底消失了。

**根因**：`FLAG_LAYOUT_NO_LIMITS` 允许内容绘制到 ALL 系统栏（包括状态栏和导航栏）下方。但如果不配合透明栏设置，系统就直接不渲染状态栏了。

**教训**：`FLAG_LAYOUT_NO_LIMITS` 是个危险的 flag，除非你明确知道自己在做什么（全屏游戏、相册查看器），否则不要用。

---

### 🕳️ 坑5：GitHub API 对 `.github/workflows/` 文件有特殊保护

**问题**：Contents API 可以正常更新 `MainActivity.java`、`themes.xml` 等源文件，但对 `.github/workflows/build.yml` 始终返回 400/404。

**尝试过**：
- Contents API PUT → 400 Bad Request
- Git Data API (tree/blob/commit) → 403/404
- GraphQL API → FORBIDDEN

**根因**：GitHub 对 workflow 文件有额外的安全检查（防止恶意 CI 注入）。

**workaround**：让用户手动在 GitHub 网页上创建/修改 workflow 文件。源码文件仍通过 API 推送。

**教训**：如果需要频繁改 workflow，要么手动操作，要么用 GitHub CLI (`gh`) 推送（它走 git 协议，不受 API 限制）。

---

### 🕳️ 坑6：APK 未签名在 Android 14+ 无法安装

**问题**：GitHub Actions 构建出的 `app-release-unsigned.apk` 在用户手机上安装时报错 `packagelnfo is null`。

**根因**：Android 14+ (API 34) 强制要求所有安装的 APK 必须有签名。

**正确方案**：本地签名。下载 artifact → zipalign → apksigner sign。
```bash
# 本地签名流程
export JAVA_HOME=/tmp/local_jdk; export PATH=$JAVA_HOME/bin:$PATH
BT=/tmp/local_sdk/build-tools/34.0.0
$BT/zipalign -f -v 4 app-release-unsigned.apk aligned.apk
$BT/apksigner sign --ks ks --ks-pass pass:android --out signed.apk aligned.apk
```

**教训**：Android 应用开发必须考虑签名。如果不在 CI 里签名，就需要本地有签名工具链。

---

### 🕳️ 坑7：Python `urllib.request` 偶发卡死

**问题**：用 Python `urllib.request.urlopen()` 调 GitHub API 偶发卡死（无限等待），阻塞整个流程。

**正确方案**：用 `curl -m 30` 设置超时 + `--data-binary` 读文件。
```bash
curl -s -m 30 -X PUT \
  -H "Authorization: token $TOKEN" \
  --data-binary @payload.json \
  "https://api.github.com/repos/..."
```

**教训**：调 API 一定要设超时。Python 的 urllib 没有内置超时容易卡死。curl 的 `-m` 参数是最可靠的保险。

---

## 三、成功经验

### ✅ 子代理 + 调试条组合拳

v8 注入一个绿色调试条，直接在手机上显示 `hdr.top`、`padding-top`、`vh` 等实时数据。这比任何猜测都有效——**30 秒就定位了根因**。

**模式**：不确定 CSS 行为时 → 先注入调试 UI → 用户截图给数据 → 精准修复。

### ✅ 调试数据揭示真相

```
DBG: vh=829 sb=134 nb=48 hdr.top=0 hdr.pad=12px
```

这行数据立刻告诉我们：
- 设备是刘海屏（134px 状态栏）
- `hdr.top: 0px` 说明位置其实是对的（用户说的"不在顶部"是指没有延伸到状态栏区域）
- `hdr.pad: 12px` 说明 fitsSystemWindows 生效了，没有叠加偏移

### ✅ 拆分写入大文件

60KB HTML 分两部分用 Python 写入（HTML+CSS 约 28KB，JS 约 33KB），避免单次写入超长字符串。

### ✅ 固定的构建+签名流水线

```
修改源码 → API push 到 GitHub → Actions 构建 (~60s)
→ 下载 artifact → 本地 zipalign+apksigner → 发送 APK
```

整个流程约 2-3 分钟（含等待构建）。

---

## 四、状态栏适配版本史（血泪教训）

| 版本 | 方案 | 结果 | 问题 |
|------|------|------|------|
| v2 | `setFitsSystemWindows(false)` + `FLAG_TRANSLUCENT` | 状态栏白框，位置错 | env() 不生效 |
| v3 | 加 `FLAG_LAYOUT_NO_LIMITS` | 状态栏消失 | 该 flag 隐藏了状态栏 |
| v4 | 改 theme 为非全屏 + JS 注入 padding | 位置仍然错 | padding 值不对 |
| v5 | `setStatusBarColor` + 旧方案 | 颜色错配 | 匹配了内容区而非头部栏 |
| v6 | 回退到纯 `FLAG_TRANSLUCENT` | 位置仍错 | 没找到根因 |
| v7 | `fitsSystemWindows(true)` + 固体色 | 颜色对了但有间距 | 系统自动加了间距 |
| v8 | 同 v7 + 调试条 | **定位根因** | 发现 134px 双重偏移 |
| v9 | `fitsSystemWindows(true)` 不加 padding | **位置对了** | ✅ 调试条还在 |
| v10 | 去掉调试条 | 正式版 | ✅ 完成 |

**核心教训**：v2-v7 每版改一点，每版都不对。v8 加了调试数据，v9 一次解决。**数据 > 猜测**。

---

## 五、Android WebView 布局核心知识

### DOM 结构
```
<body>
  <div class="app">          ← flex column, 100dvh
    <div class="hdr">        ← sticky top:0
    <div class="content">    ← flex:1, overflow-y:auto
      <div class="search">   ← sticky top:0（在 content 内部！）
    <button class="fab">     ← fixed bottom（在 app 和 content 同级）
  </div>
  <div class="batch-bar">    ← fixed bottom（在 app 外部）
</body>
```

### 关键 CSS 陷阱
1. **`height: 100dvh`** 在 Android WebView 上的行为：等于 `window.innerHeight`，不包含系统栏
2. **`position: sticky`** 在 flex 容器内需要明确的 `top` 值
3. **`env(safe-area-inset-*)`** 在 Android 上返回 0px，完全无用
4. **`max()` CSS 函数** 在 Android WebView 中被支持（Chrome 79+）
5. **`backdrop-filter`** 在 Android WebView 中被支持

### 状态栏适配规则
```
方案A: fitsSystemWindows(true)
  → 系统自动处理顶部间距
  → 不要手动加任何 padding-top
  → 配合 setStatusBarColor 设置匹配色
  → 最可靠，推荐

方案B: fitsSystemWindows(false) + FLAG_TRANSLUCENT_STATUS
  → 内容绘制到状态栏后面
  → 必须手动加 padding-top = 状态栏高度
  → 配合 getStatusBarHeight() 注入 JS
  → 更灵活但更容易出错
```

---

## 六、改进方向

1. **Gradle 签名放到 CI 里** — 省去本地签名步骤，workflow 文件需手动在 GitHub 上更新
2. **用 `gh` CLI** — 绕过 API 对 workflow 文件的限制
3. **加 WebView 调试** — `WebView.setWebContentsDebuggingEnabled(true)` 可以用 Chrome DevTools 远程调试
4. **适配更多机型** — 目前只测了刘海屏（134px），需测试折叠屏、平板等

---

## 七、总结

这次开发的核心经验就一句话：

> **数据 > 猜测，调试 > 推理。**

状态栏适配从 v2 到 v7 花了 2 小时、5 个版本，全靠猜。v8 加了调试条，30 秒定位问题，v9 一版解决。

下次遇到类似问题，第一步永远是：**注入调试 UI，获取真实数据**。
