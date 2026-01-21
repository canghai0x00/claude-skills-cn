---
name: docx
description: "全面的文档创建、编辑和分析功能，支持修订跟踪、注释、格式保留和文本提取。当Claude需要处理专业文档（.docx文件）以进行：(1)创建新文档，(2)修改或编辑内容，(3)使用修订跟踪，(4)添加注释，或任何其他文档任务时使用"
license: 专有。LICENSE.txt包含完整条款
---

# DOCX 创建、编辑和分析

## 概述

用户可能会要求你创建、编辑或分析.docx 文件的内容。.docx 文件本质上是包含 XML 文件和其他资源的 ZIP 压缩包，你可以读取或编辑它。针对不同任务，你有不同的工具和工作流程可用。

## 工作流程决策树

### 读取/分析内容

使用下面的"文本提取"或"原始 XML 访问"部分

### 创建新文档

使用"创建新 Word 文档"工作流程

### 编辑现有文档

- **自己的文档+简单更改**
  使用"基本 OOXML 编辑"工作流程

- **他人的文档**
  使用**"修订工作流程"**（推荐默认方式）

- **法律、学术、商业或政府文档**
  使用**"修订工作流程"**（必需）

## 读取和分析内容

### 文本提取

如果你只需要阅读文档的文本内容，应使用 pandoc 将文档转换为 markdown。Pandoc 提供了出色的文档结构保留支持，并可以显示修订跟踪：

```bash
# 转换文档为markdown并显示修订跟踪
pandoc --track-changes=all path-to-file.docx -o output.md
# 选项：--track-changes=accept/reject/all
```

### 原始 XML 访问

你需要原始 XML 访问以获取：注释、复杂格式、文档结构、嵌入媒体和元数据。对于任何这些功能，你需要解压文档并读取其原始 XML 内容。

#### 解压文件

`python ooxml/scripts/unpack.py <office_file> <output_directory>`

#### 关键文件结构

- `word/document.xml` - 主文档内容
- `word/comments.xml` - document.xml 中引用的注释
- `word/media/` - 嵌入的图像和媒体文件
- 修订跟踪使用`<w:ins>`（插入）和`<w:del>`（删除）标签

## 创建新 Word 文档

从头创建新 Word 文档时，使用**docx-js**，它允许你使用 JavaScript/TypeScript 创建 Word 文档。

### 工作流程

1. **必须 - 阅读整个文件**：完整阅读[`docx-js.md`](docx-js.md)（约 500 行）从头到尾。**读取此文件时切勿设置任何范围限制。**在进行文档创建之前，阅读完整文件内容以了解详细语法、关键格式规则和最佳实践。
2. 使用 Document、Paragraph、TextRun 组件创建 JavaScript/TypeScript 文件（你可以假设所有依赖项已安装，但如果未安装，请参阅下面的依赖项部分）
3. 使用 Packer.toBuffer()导出为.docx

## 编辑现有 Word 文档

编辑现有 Word 文档时，使用**Document 库**（一个用于 OOXML 操作的 Python 库）。该库自动处理基础设施设置并提供文档操作方法。对于复杂场景，你可以通过库直接访问底层 DOM。

### 工作流程

1. **必须 - 阅读整个文件**：完整阅读[`ooxml.md`](ooxml.md)（约 600 行）从头到尾。**读取此文件时切勿设置任何范围限制。**阅读完整文件内容以了解 Document 库 API 和直接编辑文档文件的 XML 模式。
2. 解压文档：`python ooxml/scripts/unpack.py <office_file> <output_directory>`
3. 使用 Document 库创建并运行 Python 脚本（参见 ooxml.md 中的"Document 库"部分）
4. 打包最终文档：`python ooxml/scripts/pack.py <input_directory> <office_file>`

Document 库既提供常见操作的高级方法，也提供复杂场景的直接 DOM 访问。

## 文档审阅的修订工作流程

此工作流程允许你在实现 OOXML 中的修改之前，使用 markdown 计划全面的修订跟踪更改。**关键**：要完成跟踪更改，你必须系统地实现所有更改。

**批处理策略**：将相关更改分组为 3-10 个更改的批次。这使调试变得可管理，同时保持效率。在进行下一批次之前测试每个批次。

**原则：最小化、精确编辑**
实现修订跟踪更改时，只标记实际发生变化的文本。重复未更改的文本会使编辑更难以审阅并显得不专业。将替换分解为：[未更改的文本] + [删除] + [插入] + [未更改的文本]。通过从原始运行中提取`<w:r>`元素并重用它，为未更改的文本保留原始运行的 RSID。

示例 - 在句子中将"30 天"更改为"60 天"：

```python
# 不好 - 替换整个句子
'<w:del><w:r><w:delText>期限为30天。</w:delText></w:r></w:del><w:ins><w:r><w:t>期限为60天。</w:t></w:r></w:ins>'

# 好 - 只标记发生变化的部分，为未更改的文本保留原始<w:r>
'<w:r w:rsidR="00AB12CD"><w:t>期限为</w:t></w:r><w:del><w:r><w:delText>30</w:delText></w:r></w:del><w:ins><w:r><w:t>60</w:t></w:r></w:ins><w:r w:rsidR="00AB12CD"><w:t>天。</w:t></w:r>'
```

### 修订跟踪工作流程

1. **获取 markdown 表示**：将文档转换为保留修订跟踪的 markdown：

   ```bash
   pandoc --track-changes=all path-to-file.docx -o current.md
   ```

2. **识别并分组更改**：审阅文档并确定所有需要的更改，将它们组织成逻辑批次：

   **定位方法**（用于在 XML 中查找更改）：

   - 节/标题编号（例如，"第 3.2 节"，"第 IV 条"）
   - 如果有编号则使用段落标识符
   - 带有独特周围文本的 Grep 模式
   - 文档结构（例如，"第一段"，"签名块"）
   - **不要使用 markdown 行号** - 它们不映射到 XML 结构

   **批次组织**（每批次分组 3-10 个相关更改）：

   - 按节：第"批次 1：第 2 节修订"，"批次 2：第 5 节更新"
   - 按类型："批次 1：日期更正"，"批次 2：方名称更改"
   - 按复杂度：从简单文本替换开始，然后处理复杂结构更改
   - 按顺序："批次 1：第 1-3 页"，"批次 2：第 4-6 页"

3. **阅读文档并解压**：

   - **必须 - 阅读整个文件**：完整阅读[`ooxml.md`](ooxml.md)（约 600 行）从头到尾。**读取此文件时切勿设置任何范围限制。**特别注意"Document 库"和"修订跟踪模式"部分。
   - **解压文档**：`python ooxml/scripts/unpack.py <file.docx> <dir>`
   - **注意建议的 RSID**：解压脚本将为你的修订跟踪更改建议一个 RSID。复制此 RSID 以在步骤 4b 中使用。

4. **分批实施更改**：将更改逻辑分组（按部分、按类型或按接近性）并在单个脚本中一起实现它们。这种方法：

   - 使调试更容易（更小的批次=更容易隔离错误）
   - 允许渐进式进展
   - 保持效率（3-10 个更改的批量大小效果良好）

   **建议的批次分组：**

   - 按文档部分（例如，"第 3 节更改"，"定义"，"终止条款"）
   - 按更改类型（例如，"日期更改"，"方名称更新"，"法律术语替换"）
   - 按接近性（例如，"第 1-3 页的更改"，"文档前半部分的更改"）

   对于每批相关更改：

   **a. 将文本映射到 XML**：在`word/document.xml`中 grep 文本，以验证文本如何在`<w:r>`元素之间分割。

   **b. 创建并运行脚本**：使用`get_node`查找节点，实现更改，然后`doc.save()`。参见 ooxml.md 中的**"Document 库"**部分以获取模式。

   **注意**：在编写脚本之前，请立即 grep `word/document.xml`以获取当前行号并验证文本内容。每次脚本运行后行号会改变。

5. **打包文档**：所有批次完成后，将解压缩的目录转换回.docx：

   ```bash
   python ooxml/scripts/pack.py unpacked reviewed-document.docx
   ```

6. **最终验证**：对完整文档进行全面检查：
   - 将最终文档转换为 markdown：
     ```bash
     pandoc --track-changes=all reviewed-document.docx -o verification.md
     ```
   - 验证所有更改是否已正确应用：
     ```bash
     grep "原始短语" verification.md  # 不应找到
     grep "替换短语" verification.md  # 应该找到
     ```
   - 检查是否引入了任何意外更改

## 将文档转换为图像

要直观分析 Word 文档，请使用两步过程将其转换为图像：

1. **将 DOCX 转换为 PDF**：

   ```bash
   soffice --headless --convert-to pdf document.docx
   ```

2. **将 PDF 页面转换为 JPEG 图像**：
   ```bash
   pdftoppm -jpeg -r 150 document.pdf page
   ```
   这将创建如`page-1.jpg`、`page-2.jpg`等文件。

选项：

- `-r 150`：将分辨率设置为 150 DPI（调整以平衡质量/大小）
- `-jpeg`：输出 JPEG 格式（如果首选 PNG，则使用`-png`）
- `-f N`：要转换的第一页（例如，`-f 2`从第 2 页开始）
- `-l N`：要转换的最后一页（例如，`-l 5`在第 5 页停止）
- `page`：输出文件的前缀

特定范围的示例：

```bash
pdftoppm -jpeg -r 150 -f 2 -l 5 document.pdf page  # 仅转换第2-5页
```

## 代码风格指南

**重要**：在生成 DOCX 操作代码时：

- 编写简洁的代码
- 避免冗长的变量名和冗余操作
- 避免不必要的打印语句

## 依赖项

所需依赖项（如果不可用，请安装）：

- **pandoc**：`sudo apt-get install pandoc`（用于文本提取）
- **docx**：`npm install -g docx`（用于创建新文档）
- **LibreOffice**：`sudo apt-get install libreoffice`（用于 PDF 转换）
- **Poppler**：`sudo apt-get install poppler-utils`（用于 pdftoppm 将 PDF 转换为图像）
- **defusedxml**：`pip install defusedxml`（用于安全 XML 解析）
