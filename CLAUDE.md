# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

NovelKit 是一个 AI 辅助小说写作工具包，提供 40+ 个 `/novel.xxx` 格式的 Slash 命令，用于在 AI 环境（如 Cursor）中进行小说创作。这是一个"元项目"——主要开发内容是编写命令文件、模板文件和脚本文件，而非传统的应用程序代码。

## 常用命令

### 构建
```bash
# 构建指定 AI 环境和平台
python build_novelkit.py cursor linux   # Linux 版本
python build_novelkit.py cursor win     # Windows 版本

# 构建所有支持的 AI 环境
python build_novelkit.py all
```

构建产物位于 `dist/{ai}-{platform}/` 目录。

### 测试
```bash
# 初始化测试项目
novel-kit init test-project --ai cursor

# 在 Cursor 中测试命令
/novel.your.command
```

### 发布
```bash
# 1. 更新 pyproject.toml 中的版本号
# 2. 提交更改
# 3. 创建并推送 tag
git tag v0.0.x && git push origin v0.0.x
```

GitHub Actions 会自动读取 `build-config.json`，构建所有 AI 环境并发布到 Release。

## 架构

### 核心组件

```
novel-kit/
├── commands/              # AI 命令定义（Markdown + YAML Front Matter）
├── templates/             # 模板文件（生成内容的结构）
├── scripts/               # 自动化脚本
│   ├── bash/             # Linux/Mac 脚本
│   └── powershell/       # Windows 脚本
├── memory/               # 项目配置（config.json）
├── src/novel_kit_cli/    # CLI 工具（Python）
└── build_novelkit.py     # 构建脚本
```

### 构建流程

`build_novelkit.py` 将源文件转换为发布包：
1. 复制 `memory/config.json` → `.novelkit/memory/config.json`
2. 复制 `templates/` → `.novelkit/templates/`
3. 复制平台特定脚本 → `.novelkit/scripts/`
4. 转换命令文件：`commands/*.md` → `.cursor/commands/novel.*.md`

命令文件名转换规则：
- `writer-new.md` → `novel.writer.new.md`
- `novel-setup.md` → `novel.setup.md`

### 命令文件格式

```markdown
---
description: 命令描述
scripts:
  sh: .novelkit/scripts/bash/manager.sh action "$ARGUMENTS"
  ps: .novelkit/scripts/powershell/manager.ps1 -Action Action -Json "$ARGUMENTS"
---

## User Input

```text
$ARGUMENTS
```

[命令的交互流程和 AI 行为定义...]
```

### 脚本规范

脚本需要：
- 查找项目根目录（通过 `.novelkit/` 或 `.git/`）
- 读取/更新 `memory/config.json`
- 使用模板生成文件

## 支持的 AI 环境

在 `build-config.json` 中配置，当前支持 18 个 AI 环境：claude, gemini, copilot, cursor-agent, cursor, qwen, opencode, windsurf, codex, kilocode, auggie, roo, codebuddy, qoder, amp, shai, q, bob。

`AI_ENV_CONFIG`（在 `build_novelkit.py` 和 `src/novel_kit_cli/__init__.py` 中）定义了每个 AI 环境的目录结构和格式配置。

## 提交规范

使用约定式提交：
```
feat(command): 添加新命令
fix(script): 修复脚本错误
docs: 更新文档
```
