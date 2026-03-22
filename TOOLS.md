# TOOLS.md - Local Notes

Skills define _how_ tools work. This file is for _your_ specifics — the stuff that's unique to your setup.

## What Goes Here

Things like:

- Camera names and locations
- SSH hosts and aliases
- Preferred voices for TTS
- Speaker/room names
- Device nicknames
- Anything environment-specific

## Examples

```markdown
### Cameras

- living-room → Main area, 180° wide angle
- front-door → Entrance, motion-triggered

### SSH

- home-server → 192.168.1.100, user: admin

### TTS

- Preferred voice: "Nova" (warm, slightly British)
- Default speaker: Kitchen HomePod
```

## Why Separate?

Skills are shared. Your setup is yours. Keeping them apart means you can update skills without losing your notes, and share skills without leaking your infrastructure.

---

## 动画查询

- **yuc.wiki** (https://yuc.wiki)
  - 用途：按季度划分的新番列表（如 26 年 1 月新番）
  - 查新番、快览当季有哪些动画 → 用这个

- **bangumi** (https://bgm.tv)
  - 用途：动画番剧的详细信息、评分、吐槽
  - 查详细动画信息、剧情、评价 → 用这个

---

## Web Search

- **Tavily Search** - AI优化的网络搜索
  - API Key: `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`
  - 使用方式: 通过 `tavily-search` skill 执行搜索
  - 技能位置: `~/workspace/agent/workspace/skills/tavily-search`

Add whatever helps you do your job. This is your cheat sheet.
