# Agents.md Collection

A GitHub repository focused on collecting and showcasing agents.md files, providing structured AI agent instruction content for frontend websites.

## 📖 Project Overview

This repository collects `agents.md` files for various project types, each containing AI agent instructions tailored to specific project categories. All files use YAML front matter to store metadata, ensuring structured and easily parseable content.

## 🗂️ Repository Structure

```
agents_md_collection/
├── examples/                 # Various project type agents.md examples
│   ├── react-project.md     # React project example
│   └── ...                  # More examples
├── templates/               # Template files
│   └── agents-template.md   # agents.md template
├── CONTRIBUTING.md          # Contribution guidelines
└── README.md               # Project documentation
```

## 📋 File Format

Each agents.md file follows this format:

```markdown
---
name: "Project Name"
description: "Brief project description"
category: "Project Category"
author: "Author Name"
authorUrl: "Author Link (optional)"
tags: ["tag1", "tag2", "tag3"]
lastUpdated: "2024-05-30"
---

# Project Title

Specific agents.md content goes here...
```

## 🚀 Usage

1. **Browse Examples**: Check out various project examples in the `examples/` directory
2. **Use Templates**: Copy `templates/agents-template.md` as a starting point
3. **Contribute Content**: Refer to `CONTRIBUTING.md` to learn how to contribute new agents.md files

## 📊 Metadata Fields

| Field         | Type   | Required | Description                                                  |
| ------------- | ------ | -------- | ------------------------------------------------------------ |
| `name`        | String | ✅       | Project or template name                                     |
| `description` | String | ✅       | Brief project description                                    |
| `category`    | String | ✅       | Project category (e.g., Frontend Framework, Backend Service) |
| `author`      | String | ✅       | Author name                                                  |
| `authorUrl`   | String | ❌       | Author's GitHub or personal website link                     |
| `tags`        | Array  | ✅       | Related technology tags                                      |
| `lastUpdated` | String | ✅       | Last update date (YYYY-MM-DD format)                         |

## 🤝 Contributing

We welcome contributions of new agents.md files! Please check [CONTRIBUTING.md](CONTRIBUTING.md) for detailed contribution guidelines.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🔗 Related Links

- [🌐 Live Demo - AgentsMD.net](https://agentsmd.net) - **Explore all agents.md files with beautiful UI! Try it now →**
- [What is agents.md?](https://github.com/AugmentCode/agents.md)
- [AI Agent Best Practices](https://docs.augmentcode.com)

---

⭐ If this project helps you, please give us a Star!
