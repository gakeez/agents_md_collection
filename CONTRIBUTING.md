# 贡献指南

感谢您对 Agents.md Collection 项目的关注！我们欢迎所有形式的贡献。

## 🎯 贡献类型

- 添加新的 agents.md 文件示例
- 改进现有的示例
- 修复文档错误
- 提出改进建议

## 📝 添加新的 agents.md 文件

### 1. 文件命名规范

- 使用小写字母和连字符：`project-type.md`
- 文件名应该清晰描述项目类型
- 例如：`react-project.md`、`python-fastapi.md`、`nodejs-express.md`

### 2. 文件结构要求

每个 agents.md 文件必须包含以下结构：

```markdown
---
name: "项目名称"
description: "简短的项目描述（不超过100字）"
category: "项目分类"
author: "您的姓名"
authorUrl: "https://github.com/yourusername"  # 可选
tags: ["tag1", "tag2", "tag3"]
lastUpdated: "2024-05-30"
---

# 项目标题

## 项目概述
简要描述项目的目的和功能...

## 技术栈
- 技术1
- 技术2

## 项目结构
描述推荐的项目结构...

## 开发规范
详细的开发指导原则...

## 示例代码
提供相关的代码示例...
```

### 3. 元数据填写指南

#### 必填字段：
- **name**: 项目或模板的清晰名称
- **description**: 50-100字的简短描述
- **category**: 选择合适的分类（见下方分类列表）
- **author**: 您的真实姓名或 GitHub 用户名
- **tags**: 3-6个相关技术标签
- **lastUpdated**: 当前日期（YYYY-MM-DD 格式）

#### 可选字段：
- **authorUrl**: 您的 GitHub 主页或个人网站

#### 推荐分类：
- `前端框架` - React、Vue、Angular 等
- `后端服务` - Express、FastAPI、Spring Boot 等
- `移动开发` - React Native、Flutter 等
- `桌面应用` - Electron、Tauri 等
- `数据科学` - Python 数据分析、机器学习等
- `DevOps` - Docker、Kubernetes、CI/CD 等
- `游戏开发` - Unity、Godot 等
- `区块链` - Web3、智能合约等

### 4. 内容质量要求

- **实用性**: 内容应该对实际项目开发有帮助
- **完整性**: 包含足够的细节和示例
- **准确性**: 确保技术信息的正确性
- **清晰性**: 使用清晰的语言和良好的格式

## 🔄 提交流程

1. **Fork 仓库**
2. **创建新分支**: `git checkout -b add-new-agent-example`
3. **添加文件**: 在 `examples/` 目录下添加您的 agents.md 文件
4. **测试验证**: 确保 YAML front matter 格式正确
5. **提交更改**: 
   ```bash
   git add examples/your-project.md
   git commit -m "Add: [项目类型] agents.md example"
   ```
6. **推送分支**: `git push origin add-new-agent-example`
7. **创建 Pull Request**

## ✅ Pull Request 检查清单

在提交 PR 之前，请确保：

- [ ] 文件名符合命名规范
- [ ] YAML front matter 格式正确且包含所有必填字段
- [ ] 内容结构清晰，包含必要的章节
- [ ] 技术信息准确无误
- [ ] 语言表达清晰易懂
- [ ] 提供了实用的示例代码（如适用）

## 🐛 报告问题

如果您发现任何问题，请：

1. 检查是否已有相关 Issue
2. 创建新的 Issue，包含：
   - 问题的详细描述
   - 重现步骤（如适用）
   - 期望的行为
   - 相关的文件或链接

## 💡 改进建议

我们欢迎任何改进建议！请通过以下方式分享：

- 创建 Issue 讨论新功能
- 在现有 Issue 中参与讨论
- 直接提交改进的 Pull Request

## 📞 联系我们

如果您有任何问题，可以：

- 创建 GitHub Issue
- 发送邮件至：[your-email@example.com]
- 在 GitHub Discussions 中讨论

感谢您的贡献！🎉
