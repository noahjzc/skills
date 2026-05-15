# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个 Claude Code 插件仓库，聚合个人 Skills。插件通过 `/plugin marketplace add` 安装。

## 项目结构

- `.claude-plugin/plugin.json` — 插件清单
- `skills/` — 所有 skill 存放目录，每个 skill 是一个子目录，包含 `SKILL.md`
- SKILL.md 使用 YAML frontmatter（name、description）+ Markdown 指令体

## 新增 Skill

1. 在 `skills/` 下创建以 skill 名称命名的目录
2. 编写 `SKILL.md`，包含 frontmatter 和指令内容
3. 如有参考文档放 `references/`，辅助脚本放 `scripts/`
4. 更新 README.md 的 Skills 列表表格

## 开发命令

此项目为纯 Markdown/配置项目，无构建或测试命令。可通过 `claude --plugin-dir .` 本地测试插件效果。
