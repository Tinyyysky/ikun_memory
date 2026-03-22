# 2026-03-21 每日工作总结

## 📋 工作概览

今天是超级忙碌的一天，主要围绕**提示词盒子 App** 从 v4 升级到 v14+，经历了大量功能开发、bug修复、Android原生打包、状态栏适配等。

---

## 1. 安装的 SkillHub 技能

从 SkillHub（腾讯 skill 社区，API: lightmake.site）安装了 **14 个 coding 相关技能：

| 技能 | 用途 |
|------|------|
| code-runner | 代码运行 |
| coding-agent | 编程助手 |
| github | GitHub 集成 |
| github-cli-tool | GitHub CLI 工具 |
| git-commit-template | Git 提交模板 |
| gitlab-integration | GitLab 集成 |
| dotnet-expert | .NET 专家 |
| rust | Rust 开发 |
| swift | Swift 开发 |
| python | Python 开发 |
| json | JSON 处理 |
| opencode | OpenCode |
| tshogx-coding-agent | 编码助手 |
| azure-keyvault-certificates-rust | Azure KeyVault |
| api-design-doc | API 设计文档 |

> 这些是指导性 skill 文件（文档），不是工具插件。

---

## 2. 提供的服务

### 提示词盒子 App（全天，10+小时）

**v5 功能升级（13 项）：**
- 模板系统（6 预设 + 自定义）、标签组合筛选、Markdown 预览
- 增强导出（MD/批量/筛选导出）、键盘快捷键、钉选/置顶
- 版本历史（最多 5 版恢复）、自定义主题色（8 色）
- JSON 分享导出/导入、拖拽排序、空状态引导
- 卡片颜色标记（6 色）、复制历史（15 条）

**v5→v14 迭代修复（30+ 次提交）：**
- 附件图片保存 bug、图片预览、卡片大小自适应
- 闪屏修复、侧边栏精简、毛玻璃效果
- 复制逻辑调整（快捷复制 vs 计数副本）
- 搜索栏滚动隐藏 + 防抖 + 120Hz 性能优化
- Action Sheet 统一交互（iOS 风格底部菜单）
- 多选模式重构（4 宫格按钮模态框）
- 所有系统 prompt()/confirm() 替换为自定义模态框
- 图片压缩 → IndexedDB fallback 存储
- 图片堆叠滑动交互（v1.0.2b）

**Android 原生打包：**
- WebView 壳 + GitHub Actions CI 自动构建
- 本地 JDK 17 + Android build-tools 签名流水线
- 15+ 个 APK 版本（v1→v13→v1.0.2b）
- 自定义 APP 图标（5 种密度）

**状态栏适配（小米 14 HyperOS3，最头疼的部分）：**
- 经历 v10→v13 四个大版本
- fitsSystemWindows(true) + 固体色状态栏
- 小米私有 API 反射调用 setStatusBarDarkMode
- 深色/浅色模式动态切换
- 刘海屏 134px 状态栏 + 48px 导航栏适配

### 其他服务
- ClaWHub 网站访问排查（发现 API 在 lightmake.site）
- GitHub API 限制探索（workflow 文件 400/404 问题）
- ACP（Claude Code）子代理测试

---

## 3. 学到的新知识

1. **bash heredoc 写大文件的坑** — 特殊字符（JS 中的 $, `, \ 等）会提前截断 heredoc。以后写含代码的大文件必须用 `python3 << 'PYEOF'` 方案。

2. **Android WebView 不支持 `env(safe-area-inset-*)`** — 需要用 JS 注入 `getStatusBarHeight()` 手动计算。

3. **小米 HyperOS 深色模式** — 标准 Android API `SYSTEM_UI_FLAG_LIGHT_STATUS_BAR` 不生效，需用小米私有 API 反射调用 `Window.setStatusBarDarkMode()`。

4. **GitHub API 限制** — `.github/workflows/` 目录下的文件不能通过 Contents API PUT 写入（返回 400/404），可能有安全保护机制。

5. **双 Git 仓库工作流** — 飞书 Git 存代码 + GitHub 触发 Actions 构建，需要两个都推保持同步。

6. **GitHub API 推大文件** — 100KB+ base64 内容需要 120s 超时，curl 比 Python urllib 更稳定。

7. **Android APK 签名** — Android 14+ 不允许安装未签名 APK，需要 keytool + zipalign + apksigner 完整流程。

8. **SkillHub API 发现** — ClaWHub 和腾讯 SkillHub 共享后端（lightmake.site），搜索 API: `/api/skills?keyword=xxx`。

9. **fitsSystemWindows 的坑** — `false` 会导致布局错位（侧边栏文字太靠上等），`true` 是正确默认。

10. **图片存储策略** — localStorage 有 5MB 限制，大图片需用 IndexedDB fallback（>50KB 的附件提取到 IDB）。

---

## 4. 可改进的地方

1. **子代理可靠性** — 大任务交给子代理经常超时或不完整，主 agent 需要更好的 fallback 策略，不能依赖子代理完成关键路径。

2. **迭代速度 vs 质量** — 今天 30+ 次迭代中有太多"用户反馈→修→再反馈→再修"的循环。应该在第一版就做更全面的测试，特别是 CSS 优先级冲突（今天的多次 bug 源头）。

3. **CSS 优先级管理** — 多次出现重复/冲突的 CSS 规则导致显示异常。应该建立更好的 CSS 组织结构，避免后面规则覆盖前面。

4. **Android 状态栏适配** — 应该一开始就在真机上测试，而不是纯靠理论方案迭代。有小米14设备数据（134px状态栏）后才找到正确方案。

5. **文件写入方式** — 今天浪费了大量时间在 bash heredoc 截断问题上。应该从一开始就用 Python 写大文件，不要反复尝试 bash 方案。

6. **APK 签名自动化** — 目前每次构建后需要本地手动签名。如果能解决 workflow 文件的 API 写入问题，可以在 CI 中自动签名。

7. **知识沉淀** — 状态栏适配、GitHub API 限制等经验应该写入 TOOLS.md，避免未来重复踩坑。
