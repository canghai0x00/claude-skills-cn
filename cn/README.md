> **注意：** 本仓库包含 Anthropic 对 Claude 技能的实现。有关 Agent Skills 标准的信息，请参阅[agentskills.io](http://agentskills.io)。

# 技能

技能是 Claude 动态加载的指令、脚本和资源文件夹，用于提高特定任务的性能。技能教会 Claude 如何以可重复的方式完成特定任务，无论是按照公司品牌指南创建文档、使用组织特定工作流分析数据，还是自动化个人任务。

更多信息，请查看：

- [什么是技能？](https://support.claude.com/en/articles/12512176-what-are-skills)
- [在 Claude 中使用技能](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
- [如何创建自定义技能](https://support.claude.com/en/articles/12512198-creating-custom-skills)
- [使用 Agent Skills 为代理配备现实世界能力](https://anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)

# 关于本仓库

本仓库包含展示 Claude 技能系统可能性的各种技能。这些技能涵盖从创意应用（艺术、音乐、设计）到技术任务（测试 Web 应用、MCP 服务器生成）再到企业工作流程（通信、品牌等）。

每个技能都包含在其自己的文件夹中，带有一个`SKILL.md`文件，其中包含 Claude 使用的指令和元数据。浏览这些技能可以获取创建自己技能的灵感，或了解不同的模式和方法。

本仓库中的许多技能都是开源的（Apache 2.0 许可）。我们还包含了支持[Claude 文档功能](https://www.anthropic.com/news/create-files)的文档创建和编辑技能，位于[`skills/docx`](./skills/docx)、[`skills/pdf`](./skills/pdf)、[`skills/pptx`](./skills/pptx)和[`skills/xlsx`](./skills/xlsx)子文件夹中。这些技能是源代码可用的，而非开源的，但我们希望与开发者分享这些作为更复杂技能的参考，这些技能在生产 AI 应用中被积极使用。

## 免责声明

**这些技能仅供演示和教育目的。** 虽然这些功能中的一些可能在 Claude 中可用，但您从 Claude 获得的实现和行为可能与这些技能中展示的不同。这些技能旨在说明模式和可能性。在依赖它们执行关键任务之前，请始终在您自己的环境中彻底测试技能。

# 技能集合

- [./skills](./skills)：创意与设计、开发与技术、企业与通信以及文档技能的示例
- [./spec](./spec)：Agent Skills 规范
- [./template](./template)：技能模板

# 在 Claude Code、Claude.ai 和 API 中尝试

## Claude Code

您可以通过在 Claude Code 中运行以下命令将此仓库注册为 Claude Code Plugin marketplace：

```
/plugin marketplace add anthropics/skills
```

然后，安装特定的技能集：

1. 选择`浏览并安装插件`
2. 选择`anthropic-agent-skills`
3. 选择`document-skills`或`example-skills`
4. 选择`立即安装`

或者，直接通过以下命令安装任一插件：

```
/plugin install document-skills@anthropic-agent-skills
/plugin install example-skills@anthropic-agent-skills
```

安装插件后，只需提及它即可使用该技能。例如，如果您从 marketplace 安装了`document-skills`插件，可以要求 Claude Code 执行类似操作："使用 PDF 技能从`path/to/some-file.pdf`提取表单字段"

## Claude.ai

Claude.ai 的付费计划中已经提供了这些示例技能。

要使用此仓库中的任何技能或上传自定义技能，请按照[在 Claude 中使用技能](https://support.claude.com/en/articles/12512180-using-skills-in-claude#h_a4222fa77b)中的说明操作。

## Claude API

您可以通过 Claude API 使用 Anthropic 预构建的技能，并上传自定义技能。详见[Skills API 快速入门](https://docs.claude.com/en/api/skills-guide#creating-a-skill)。

# 创建基本技能

创建技能非常简单 - 只需一个包含 YAML 前置元数据和指令的`SKILL.md`文件的文件夹。您可以使用本仓库中的**template-skill**作为起点：

```markdown
---
name: my-skill-name
description: 关于此技能功能和使用时机的清晰描述
---

# 我的技能名称

[在这里添加 Claude 在此技能激活时将遵循的指令]

## 示例

- 示例用法 1
- 示例用法 2

## 指南

- 指南 1
- 指南 2
```

前置元数据只需两个字段：

- `name` - 技能的唯一标识符（小写，连字符代替空格）
- `description` - 关于技能功能和使用时机的完整描述

下面的 markdown 内容包含 Claude 将遵循的指令、示例和指南。有关更多详细信息，请参阅[如何创建自定义技能](https://support.claude.com/en/articles/12512198-creating-custom-skills)。

# 合作伙伴技能

技能是教 Claude 如何更好地使用特定软件的绝佳方式。当我们看到来自合作伙伴的优秀示例技能时，我们可能会在此处突出显示其中一些：

- **Notion** - [Claude 的 Notion 技能](https://www.notion.so/notiondevs/Notion-Skills-for-Claude-28da4445d27180c7af1df7d8615723d0)
