# Contributing Guide

Thank you for your interest in the Agents.md Collection project! We welcome all forms of contributions.

## 🎯 Contribution Types

- Add new agents.md file examples
- Improve existing examples
- Fix documentation errors
- Suggest improvements

## 📝 Adding New agents.md Files

### 1. File Naming Convention

- Use lowercase letters and hyphens: `project-type.md`
- File names should clearly describe the project type
- Examples: `react-project.md`, `python-fastapi.md`, `nodejs-express.md`

### 2. File Structure Requirements

Each agents.md file must include the following structure:

```markdown
---
name: "Project Name"
description: "Brief project description (under 100 words)"
category: "Project Category"
author: "Your Name"
authorUrl: "https://github.com/yourusername" # Optional
tags: ["tag1", "tag2", "tag3"]
lastUpdated: "2024-05-30"
---

# Project Title

## Project Overview

Brief description of the project's purpose and functionality...

## Tech Stack

- Technology 1
- Technology 2

## Project Structure

Describe the recommended project structure...

## Development Guidelines

Detailed development principles...

## Example Code

Provide relevant code examples...
```

### 3. Metadata Guidelines

#### Required Fields:

- **name**: Clear project or template name
- **description**: 50-100 word brief description
- **category**: Choose appropriate category (see category list below)
- **author**: Your real name or GitHub username
- **tags**: 3-6 relevant technology tags
- **lastUpdated**: Current date (YYYY-MM-DD format)

#### Optional Fields:

- **authorUrl**: Your GitHub homepage or personal website

#### Recommended Categories:

- `Frontend Framework` - React, Vue, Angular, etc.
- `Backend Service` - Express, FastAPI, Spring Boot, etc.
- `Mobile Development` - React Native, Flutter, etc.
- `Desktop Application` - Electron, Tauri, etc.
- `Data Science` - Python data analysis, machine learning, etc.
- `DevOps` - Docker, Kubernetes, CI/CD, etc.
- `Game Development` - Unity, Godot, etc.
- `Blockchain` - Web3, smart contracts, etc.

### 4. Content Quality Requirements

- **Practicality**: Content should be helpful for actual project development
- **Completeness**: Include sufficient details and examples
- **Accuracy**: Ensure technical information is correct
- **Clarity**: Use clear language and good formatting

## 🔄 Submission Process

1. **Fork the repository**
2. **Create a new branch**: `git checkout -b add-new-agent-example`
3. **Add your file**: Add your agents.md file to the `examples/` directory
4. **Test and validate**: Ensure YAML front matter format is correct
5. **Commit changes**:
   ```bash
   git add examples/your-project.md
   git commit -m "Add: [Project Type] agents.md example"
   ```
6. **Push branch**: `git push origin add-new-agent-example`
7. **Create Pull Request**

## ✅ Pull Request Checklist

Before submitting a PR, ensure:

- [ ] File name follows naming convention
- [ ] YAML front matter format is correct and includes all required fields
- [ ] Content structure is clear and includes necessary sections
- [ ] Technical information is accurate
- [ ] Language is clear and understandable
- [ ] Practical example code is provided (if applicable)

## 🐛 Reporting Issues

If you find any issues, please:

1. Check if there's already a related Issue
2. Create a new Issue including:
   - Detailed description of the problem
   - Steps to reproduce (if applicable)
   - Expected behavior
   - Related files or links

## 💡 Improvement Suggestions

We welcome any improvement suggestions! Please share through:

- Create an Issue to discuss new features
- Participate in discussions on existing Issues
- Submit improvement Pull Requests directly

## 📞 Contact Us

If you have any questions, you can:

- Create a GitHub Issue
- Send email to: [your-email@example.com]
- Discuss in GitHub Discussions

Thank you for your contribution! 🎉
