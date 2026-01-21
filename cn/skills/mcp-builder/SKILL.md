---
name: mcp-builder
description: 创建高质量MCP（模型上下文协议）服务器的指南，使大型语言模型能够通过精心设计的工具与外部服务交互。在构建MCP服务器以集成外部API或服务时使用，无论是使用Python（FastMCP）还是Node/TypeScript（MCP SDK）。
license: 完整条款见LICENSE.txt
---

# MCP 服务器开发指南

## 概述

创建 MCP（模型上下文协议）服务器，使大型语言模型能够通过精心设计的工具与外部服务交互。MCP 服务器的质量取决于它如何有效地使大型语言模型完成现实世界的任务。

---

# 流程

## 🚀 高级工作流程

创建高质量的 MCP 服务器涉及四个主要阶段：

### 阶段 1：深入研究和规划

#### 1.1 理解现代 MCP 设计

**API 覆盖 vs. 工作流工具：**
平衡全面的 API 端点覆盖与专门的工作流工具。工作流工具对特定任务更便捷，而全面覆盖则给予代理组合操作的灵活性。性能因客户端而异——某些客户端受益于可以组合基本工具的代码执行，而其他客户端则使用高级工作流表现更好。在不确定时，优先考虑全面的 API 覆盖。

**工具命名和可发现性：**
清晰、描述性的工具名称帮助代理快速找到正确的工具。使用一致的前缀（例如，`github_create_issue`，`github_list_repos`）和面向动作的命名。

**上下文管理：**
代理受益于简洁的工具描述和过滤/分页结果的能力。设计返回聚焦、相关数据的工具。一些客户端支持代码执行，这有助于代理高效过滤和处理数据。

**可操作的错误消息：**
错误消息应该通过具体建议和后续步骤引导代理找到解决方案。

#### 1.2 研究 MCP 协议文档

**浏览 MCP 规范：**

从网站地图开始查找相关页面：`https://modelcontextprotocol.io/sitemap.xml`

然后获取带有`.md`后缀的特定页面以获取 markdown 格式（例如，`https://modelcontextprotocol.io/specification/draft.md`）。

关键页面包括：

- 规范概述和架构
- 传输机制（可流式 HTTP，stdio）
- 工具、资源和提示定义

#### 1.3 研究框架文档

**推荐技术栈：**

- **语言**：TypeScript（高质量 SDK 支持，在许多执行环境中兼容性好，如 MCPB。此外，AI 模型善于生成 TypeScript 代码，受益于其广泛使用、静态类型和良好的代码检查工具）
- **传输**：用于远程服务器的可流式 HTTP，使用无状态 JSON（相比有状态会话和流式响应，更容易扩展和维护）。本地服务器使用 stdio。

**加载框架文档：**

- **MCP 最佳实践**：[📋 查看最佳实践](./reference/mcp_best_practices.md) - 核心指南

**对于 TypeScript（推荐）：**

- **TypeScript SDK**：使用 WebFetch 加载`https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`
- [⚡ TypeScript 指南](./reference/node_mcp_server.md) - TypeScript 模式和示例

**对于 Python：**

- **Python SDK**：使用 WebFetch 加载`https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`
- [🐍 Python 指南](./reference/python_mcp_server.md) - Python 模式和示例

#### 1.4 规划你的实现

**了解 API：**
查看服务的 API 文档以识别关键端点、认证需求和数据模型。根据需要使用网络搜索和 WebFetch。

**工具选择：**
优先考虑全面的 API 覆盖。列出要实现的端点，从最常用的操作开始。

---

### 阶段 2：实现

#### 2.1 设置项目结构

参见特定语言的指南进行项目设置：

- [⚡ TypeScript 指南](./reference/node_mcp_server.md) - 项目结构，package.json，tsconfig.json
- [🐍 Python 指南](./reference/python_mcp_server.md) - 模块组织，依赖项

#### 2.2 实现核心基础设施

创建共享实用工具：

- 带认证的 API 客户端
- 错误处理助手
- 响应格式化（JSON/Markdown）
- 分页支持

#### 2.3 实现工具

对每个工具：

**输入模式：**

- 使用 Zod（TypeScript）或 Pydantic（Python）
- 包括约束和清晰描述
- 在字段描述中添加示例

**输出模式：**

- 在可能的情况下定义`outputSchema`以获取结构化数据
- 在工具响应中使用`structuredContent`（TypeScript SDK 功能）
- 帮助客户端理解和处理工具输出

**工具描述：**

- 功能的简明摘要
- 参数描述
- 返回类型模式

**实现：**

- 对 I/O 操作使用 async/await
- 适当的错误处理，提供可操作的消息
- 在适用的情况下支持分页
- 使用现代 SDK 时返回文本内容和结构化数据

**注解：**

- `readOnlyHint`：true/false
- `destructiveHint`：true/false
- `idempotentHint`：true/false
- `openWorldHint`：true/false

---

### 阶段 3：审查和测试

#### 3.1 代码质量

审查以下方面：

- 无重复代码（DRY 原则）
- 一致的错误处理
- 完全类型覆盖
- 清晰的工具描述

#### 3.2 构建和测试

**TypeScript：**

- 运行`npm run build`验证编译
- 使用 MCP 检查器测试：`npx @modelcontextprotocol/inspector`

**Python：**

- 验证语法：`python -m py_compile your_server.py`
- 使用 MCP 检查器测试

参见特定语言的指南获取详细测试方法和质量检查表。

---

### 阶段 4：创建评估

实现 MCP 服务器后，创建全面评估以测试其有效性。

**加载[✅ 评估指南](./reference/evaluation.md)获取完整评估指南。**

#### 4.1 理解评估目的

使用评估来测试大型语言模型是否能有效使用你的 MCP 服务器回答现实、复杂的问题。

#### 4.2 创建 10 个评估问题

要创建有效评估，请按照评估指南中概述的流程：

1. **工具检查**：列出可用工具并了解它们的功能
2. **内容探索**：使用只读操作探索可用数据
3. **问题生成**：创建 10 个复杂、现实的问题
4. **答案验证**：自己解决每个问题以验证答案

#### 4.3 评估要求

确保每个问题：

- **独立**：不依赖其他问题
- **只读**：只需要非破坏性操作
- **复杂**：需要多个工具调用和深入探索
- **现实**：基于人类关心的真实用例
- **可验证**：有单一、明确的答案，可通过字符串比较验证
- **稳定**：答案不会随时间变化

#### 4.4 输出格式

创建具有以下结构的 XML 文件：

```xml
<evaluation>
  <qa_pair>
    <question>查找关于带有动物代号的AI模型发布的讨论。一个模型需要特定的安全名称，格式为ASL-X。为名字以斑点野猫命名的模型确定的数字X是多少？</question>
    <answer>3</answer>
  </qa_pair>
<!-- 更多qa_pairs... -->
</evaluation>
```

---

# 参考文件

## 📚 文档库

在开发过程中根据需要加载这些资源：

### 核心 MCP 文档（首先加载）

- **MCP 协议**：从`https://modelcontextprotocol.io/sitemap.xml`的网站地图开始，然后获取带有`.md`后缀的特定页面
- [📋 MCP 最佳实践](./reference/mcp_best_practices.md) - 通用 MCP 指南，包括：
  - 服务器和工具命名约定
  - 响应格式指南（JSON vs Markdown）
  - 分页最佳实践
  - 传输选择（可流式 HTTP vs stdio）
  - 安全和错误处理标准

### SDK 文档（在阶段 1/2 期间加载）

- **Python SDK**：从`https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`获取
- **TypeScript SDK**：从`https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`获取

### 特定语言实现指南（在阶段 2 期间加载）

- [🐍 Python 实现指南](./reference/python_mcp_server.md) - 完整 Python/FastMCP 指南，包含：

  - 服务器初始化模式
  - Pydantic 模型示例
  - 使用`@mcp.tool`注册工具
  - 完整工作示例
  - 质量检查表

- [⚡ TypeScript 实现指南](./reference/node_mcp_server.md) - 完整 TypeScript 指南，包含：
  - 项目结构
  - Zod 模式模式
  - 使用`server.registerTool`注册工具
  - 完整工作示例
  - 质量检查表

### 评估指南（在阶段 4 期间加载）

- [✅ 评估指南](./reference/evaluation.md) - 完整评估创建指南，包含：
  - 问题创建指南
  - 答案验证策略
  - XML 格式规范
  - 示例问题和答案
  - 使用提供的脚本运行评估
