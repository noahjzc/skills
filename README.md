# Skills

个人 Claude Code Skills 插件仓库。

## Skills 列表

| Skill | 说明 |
| --- | --- |
| `china-regions` | 从民政部 API 获取或更新全国省市区行政区划数据，生成 Ant Design Cascader 格式的 JSON 文件 |

## 安装

```bash
# 添加为插件源
/plugin marketplace add https://github.com/noahjzc/skills

# 安装插件
/plugin install noah-skills
```

### 本地测试

```bash
claude --plugin-dir /path/to/skills
```

## 项目结构

```
.
├── .claude-plugin/
│   └── plugin.json        # 插件清单
├── skills/                 # Skill 目录
│   └── <skill-name>/
│       ├── SKILL.md        # Skill 定义（YAML frontmatter + Markdown）
│       ├── references/     # 参考文档（可选）
│       ├── scripts/        # 辅助脚本（可选）
│       └── evals/          # 评估标准（可选）
├── CLAUDE.md
└── README.md
```

## 新增 Skill

在 `skills/` 下创建新目录，添加 `SKILL.md` 文件。确保：

- `description` 字段明确描述触发条件
- 指令内容精确、无歧义
- SKILL.md 保持 500 行以内，详细内容放 `references/`

## License

MIT License
