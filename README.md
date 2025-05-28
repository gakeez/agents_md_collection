# Agents.md Collection

专注于收集和展示 agents.md 文件的 GitHub 仓库，为前端网站提供结构化的 AI 代理指令内容。

## 📖 项目简介

这个仓库收集各种项目类型的 `agents.md` 文件，每个文件都包含了针对特定项目类型的 AI 代理指令。所有文件都使用 YAML front matter 来存储元数据，确保内容的结构化和易于解析。

## 🗂️ 仓库结构

```
agents_md_collection/
├── examples/                 # 各种项目类型的 agents.md 示例
│   ├── react-project.md     # React 项目示例
│   └── ...                  # 更多示例
├── templates/               # 模板文件
│   └── agents-template.md   # agents.md 模板
├── CONTRIBUTING.md          # 贡献指南
└── README.md               # 项目说明
```

## 📋 文件格式

每个 agents.md 文件都采用以下格式：

```markdown
---
name: "项目名称"
description: "项目简短描述"
category: "项目分类"
author: "作者姓名"
authorUrl: "作者链接（可选）"
tags: ["标签1", "标签2", "标签3"]
lastUpdated: "2024-05-30"
---

# 项目标题

这里是具体的 agents.md 内容...
```

## 🚀 使用方法

1. **浏览示例**: 查看 `examples/` 目录中的各种项目示例
2. **使用模板**: 复制 `templates/agents-template.md` 作为起点
3. **贡献内容**: 参考 `CONTRIBUTING.md` 了解如何贡献新的 agents.md 文件

## 📊 元数据字段说明

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `name` | String | ✅ | 项目或模板的名称 |
| `description` | String | ✅ | 简短的项目描述 |
| `category` | String | ✅ | 项目分类（如：前端框架、后端服务等） |
| `author` | String | ✅ | 作者姓名 |
| `authorUrl` | String | ❌ | 作者的 GitHub 或个人网站链接 |
| `tags` | Array | ✅ | 相关技术标签 |
| `lastUpdated` | String | ✅ | 最后更新日期（YYYY-MM-DD 格式） |

## 🤝 贡献

欢迎贡献新的 agents.md 文件！请查看 [CONTRIBUTING.md](CONTRIBUTING.md) 了解详细的贡献指南。

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

## 🔗 相关链接

- [什么是 agents.md？](https://github.com/AugmentCode/agents.md)
- [AI 代理最佳实践](https://docs.augmentcode.com)

---

⭐ 如果这个项目对您有帮助，请给我们一个 Star！
