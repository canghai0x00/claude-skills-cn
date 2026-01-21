# Office Open XML 技术参考：全面解析与更新

# Office Open XML 技术参考

**重要提示：开始前请通读本文档。** 本文档涵盖以下内容：

- [技术规范](#技术规范) - 模式合规规则和验证要求

- [文档内容格式](#文档内容格式) - 标题、列表、表格、格式设置等对应的 XML 格式

- [文档库（Python）](#文档库python) - 用于 OOXML 操作的推荐方案，含自动基础设施搭建

- [修订跟踪（红线标注）](#修订跟踪红线标注) - 实现修订跟踪的 XML 格式

## 技术规范

### 模式合规性

- `<w:pPr>` 中的元素顺序：`<w:pStyle>`、`<w:numPr>`、`<w:spacing>`、`<w:ind>`、`<w:jc>`

- 空白字符：为包含前导/尾随空格的 `<w:t>` 元素添加 `xml:space='preserve'` 属性

- 统一码（Unicode）：对 ASCII 内容中的特殊字符进行转义处理：

  - 双引号 `"` 转为 `&#8220;`

  - 字符编码参考：弯引号 `""` 转为 `&#8220;&#8221;`，撇号 `'` 转为 `&#8217;`，破折号 `—` 转为 `&#8212;`

- 修订跟踪：在 `<w:r>` 元素外部使用带 `w:author="Claude"` 属性的 `<w:del>` 和 `<w:ins>` 标签

  - **关键要求**：`<w:ins>` 必须以 `</w:ins>` 闭合，`<w:del>` 必须以 `</w:del>` 闭合——严禁混用闭合标签

  - RSID 必须为 8 位十六进制数：使用如 `00AB1234` 格式的值（仅允许 0-9、A-F 字符）

  - trackRevisions 放置位置：在 settings.xml 中，将 `<w:trackRevisions/>` 添加在 `<w:proofState>` 之后

- 图片：将图片添加至 `word/media/` 目录，在 `document.xml` 中引用，并设置尺寸以防止内容溢出

## 文档内容格式

### 基本结构

```XML

<w:p>
  <w:r><w:t>文本内容</w:t></w:r>
</w:p>
```

### 标题与样式

```XML

<w:p>
  <w:pPr>
    <w:pStyle w:val="Title"/>
    <w:jc w:val="center"/>
  </w:pPr>
  <w:r><w:t>文档标题</w:t></w:r>
</w:p>

<w:p>
  <w:pPr><w:pStyle w:val="Heading2"/></w:pPr>
  <w:r><w:t>章节标题</w:t></w:r>
</w:p>
```

### 文本格式

```XML

<!-- 加粗 -->
<w:r><w:rPr><w:b/><w:bCs/></w:rPr><w:t>Bold</w:t></w:r>
<!-- 斜体 -->
<w:r><w:rPr><w:i/><w:iCs/></w:rPr><w:t>Italic</w:t></w:r>
<!-- 下划线 -->
<w:r><w:rPr><w:u w:val="single"/></w:rPr><w:t>Underlined</w:t></w:r>
<!-- 高亮 -->
<w:r><w:rPr><w:highlight w:val="yellow"/></w:rPr><w:t>Highlighted</w:t></w:r>
```

### 列表

```XML

<!-- 编号列表 -->
<w:p>
  <w:pPr>
    <w:pStyle w:val="ListParagraph"/>
    <w:numPr><w:ilvl w:val="0"/><w:numId w:val="1"/></w:numPr>
    <w:spacing w:before="240"/>
  </w:pPr>
  <w:r><w:t>First item</w:t></w:r>
</w:p>

<!-- 重新开始编号列表（从1开始）- 使用不同的numId -->
<w:p>
  <w:pPr>
    <w:pStyle w:val="ListParagraph"/>
    <w:numPr><w:ilvl w:val="0"/><w:numId w:val="2"/></w:numPr>
    <w:spacing w:before="240"/>
  </w:pPr>
  <w:r><w:t>New list item 1</w:t></w:r>
</w:p>

<!-- 项目符号列表（二级） -->
<w:p>
  <w:pPr>
    <w:pStyle w:val="ListParagraph"/>
    <w:numPr><w:ilvl w:val="1"/><w:numId w:val="1"/></w:numPr>
    <w:spacing w:before="240"/>
    <w:ind w:left="900"/>
  </w:pPr>
  <w:r><w:t>Bullet item</w:t></w:r>
</w:p>
```

### 表格

```XML

<w:tbl>
  <w:tblPr>
    <w:tblStyle w:val="TableGrid"/>
    <w:tblW w:w="0" w:type="auto"/>
  </w:tblPr>
  <w:tblGrid>
    <w:gridCol w:w="4675"/><w:gridCol w:w="4675"/>
  </w:tblGrid>
  <w:tr>
    <w:tc>
      <w:tcPr><w:tcW w:w="4675" w:type="dxa"/></w:tcPr>
      <w:p><w:r><w:t>Cell 1</w:t></w:r></w:p>
    </w:tc>
    <w:tc>
      <w:tcPr><w:tcW w:w="4675" w:type="dxa"/></w:tcPr>
      <w:p><w:r><w:t>Cell 2</w:t></w:r></w:p>
    </w:tc>
  </w:tr>
</w:tbl>
```

### 布局

```XML

<!-- 新章节前插入分页符（通用格式） -->
<w:p>
  <w:r>
    <w:br w:type="page"/>
  </w:r>
</w:p>
<w:p>
  <w:pPr>
    <w:pStyle w:val="Heading1"/>
  </w:pPr>
  <w:r>
    <w:t>New Section Title</w:t>
  </w:r>
</w:p>

<!-- 居中段落 -->
<w:p>
  <w:pPr>
    <w:spacing w:before="240" w:after="0"/>
    <w:jc w:val="center"/>
  </w:pPr>
  <w:r><w:t>Centered text</w:t></w:r>
</w:p>

<!-- 字体更改 - 段落级别（应用于所有文本段） -->
<w:p>
  <w:pPr>
    <w:rPr><w:rFonts w:ascii="Courier New" w:hAnsi="Courier New"/></w:rPr>
  </w:pPr>
  <w:r><w:t>Monospace text</w:t></w:r>
</w:p>

<!-- 字体更改 - 文本段级别（仅适用于本段文本） -->
<w:p>
  <w:r>
    <w:rPr><w:rFonts w:ascii="Courier New" w:hAnsi="Courier New"/></w:rPr>
    <w:t>This text is Courier New</w:t>
  </w:r>
  <w:r><w:t> and this text uses default font</w:t></w:r>
</w:p>
```

## 文件更新

添加内容时，需更新以下文件：

**`word/_rels/document.xml.rels`** **：**

```XML

<Relationship Id="rId1" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/numbering" Target="numbering.xml"/>
<Relationship Id="rId5" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/image" Target="media/image1.png"/>
```

**`[Content_Types].xml`** **：**

```XML

<Default Extension="png" ContentType="image/png"/>
<Override PartName="/word/numbering.xml" ContentType="application/vnd.openxmlformats-officedocument.wordprocessingml.numbering+xml"/>
```

### 图片

**关键要求**：计算图片尺寸以防止页面溢出，并保持宽高比。

```XML

<!-- 最小必要结构 -->
<w:p>
  <w:r>
    <w:drawing>
      <wp:inline>
        <wp:extent cx="2743200" cy="1828800"/>
        <wp:docPr id="1" name="Picture 1"/>
        <a:graphic xmlns:a="http://schemas.openxmlformats.org/drawingml/2006/main">
          <a:graphicData uri="http://schemas.openxmlformats.org/drawingml/2006/picture">
            <pic:pic xmlns:pic="http://schemas.openxmlformats.org/drawingml/2006/picture">
              <pic:nvPicPr>
                <pic:cNvPr id="0" name="image1.png"/>
                <pic:cNvPicPr/>
              </pic:nvPicPr>
              <pic:blipFill>
                <a:blip r:embed="rId5"/>
                <!-- 添加以按宽高比拉伸填充 -->
                <a:stretch>
                  <a:fillRect/>
                </a:stretch>
              </pic:blipFill>
              <pic:spPr>
                <a:xfrm>
                  <a:ext cx="2743200" cy="1828800"/>
                </a:xfrm>
                <a:prstGeom prst="rect"/>
              </pic:spPr>
            </pic:pic>
          </a:graphicData>
        </a:graphic>
      </wp:inline>
    </w:drawing>
  </w:r>
</w:p>
```

### 链接（超链接）

**重要提示**：所有超链接（内部和外部）都要求在 styles.xml 中定义 Hyperlink 样式。若无此样式，链接将显示为普通文本，而非蓝色带下划线的可点击链接。

**外部链接：**

```XML

<!-- 在 document.xml 中 -->
<w:hyperlink r:id="rId5">
  <w:r>
    <w:rPr><w:rStyle w:val="Hyperlink"/></w:rPr>
    <w:t>Link Text</w:t>
  </w:r>
</w:hyperlink>

<!-- 在 word/_rels/document.xml.rels 中 -->
<Relationship Id="rId5" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/hyperlink"
              Target="https://www.example.com/" TargetMode="External"/>
```

**内部链接：**

```XML

<!-- 链接到书签 -->
<w:hyperlink w:anchor="myBookmark">
  <w:r>
    <w:rPr><w:rStyle w:val="Hyperlink"/></w:rPr>
    <w:t>Link Text</w:t>
  </w:r>
</w:hyperlink>

<!-- 书签目标 -->
<w:bookmarkStart w:id="0" w:name="myBookmark"/>
<w:r><w:t>Target content</w:t></w:r>
<w:bookmarkEnd w:id="0"/>
```

**超链接样式（需在 styles.xml 中配置）：**

```XML

<w:style w:type="character" w:styleId="Hyperlink">
  <w:name w:val="Hyperlink"/>
  <w:basedOn w:val="DefaultParagraphFont"/>
  <w:uiPriority w:val="99"/>
  <w:unhideWhenUsed/>
  <w:rPr>
    <w:color w:val="467886" w:themeColor="hyperlink"/>
    <w:u w:val="single"/>
  </w:rPr>
</w:style>
```

## 文档库（Python）

所有修订跟踪和批注操作均使用 `scripts/document.py` 中的 Document 类。该类会自动处理基础设施搭建（people.xml、RSID、settings.xml、批注文件、关系文件、内容类型）。仅在库不支持的复杂场景下，才直接进行 XML 操作。

### 统一码与实体处理：

- **搜索**：实体表示法和 Unicode 字符均可生效——`contains="&#8220;Company"` 和 `contains="\u201cCompany"` 可找到相同文本

- **替换**：可使用实体（`&#8220;`）或 Unicode（`\u201c`）——两者均可生效，并会根据文件编码自动转换（ASCII 编码 → 实体，UTF-8 编码 →Unicode）

### 初始化

**定位 docx 技能根目录**（包含 `scripts/` 和 `ooxml/` 的目录）：

```Bash

# 搜索document.py以定位技能根目录
# 注意：此处以/mnt/skills为例；请根据实际环境确认具体路径
find /mnt/skills -name "document.py" -path "*/docx/scripts/*" 2>/dev/null | head -1
# 示例输出：/mnt/skills/docx/scripts/document.py
# 技能根目录为：/mnt/skills/docx
```

**设置 PYTHONPATH 并运行脚本**（指向 docx 技能根目录）：

```Bash

PYTHONPATH=/mnt/skills/docx python your_script.py
```

**在脚本中**，从技能根目录导入模块：

```Python

from scripts.document import Document, DocxXMLEditor

# 基础初始化（自动创建临时副本并搭建基础设施）
doc = Document('unpacked')

# 自定义作者和缩写
doc = Document('unpacked', author="John Doe", initials="JD")

# 启用修订跟踪模式
doc = Document('unpacked', track_revisions=True)

# 指定自定义RSID（未提供时自动生成）
doc = Document('unpacked', rsid="07DC5ECB")
```

### 创建修订跟踪记录

**关键要求**：仅标记实际发生变更的文本。所有未变更文本必须置于 `<w:del>`/`<w:ins>` 标签外部。标记未变更文本会导致编辑内容不规范，且增加审核难度。

**属性处理**：Document 类会自动为新元素注入属性（w:id、w:date、w:rsidR、w:rsidDel、w16du:dateUtc、xml:space）。保留原始文档中未变更的文本时，需复制原始 `<w:r>` 元素及其现有属性，以保证文档完整性。

**方法选择指南**：

- **为普通文本添加自定义修订**：使用 `replace_node()` 并配合 `<w:del>`/`<w:ins>` 标签；或使用 `suggest_deletion()` 删除整个 `<w:r>` 或 `<w:p>` 元素

- **部分修改其他作者的修订内容**：使用 `replace_node()` 将你的修改嵌套在其 `<w:ins>`/`<w:del>` 标签内

- **完全拒绝其他作者的插入内容**：对 `<w:ins>` 元素使用 `revert_insertion()`（禁止使用 `suggest_deletion()`）

- **完全拒绝其他作者的删除操作**：对 `<w:del>` 元素使用 `revert_deletion()`，通过修订跟踪恢复被删除的内容

```Python

# 最小化编辑 - 修改单个词汇："The report is monthly" → "The report is quarterly"
# 原始内容：<w:r w:rsidR="00AB12CD"><w:rPr><w:rFonts w:ascii="Calibri"/></w:rPr><w:t>The report is monthly</w:t></w:r>
node = doc["word/document.xml"].get_node(tag="w:r", contains="The report is monthly")
rpr = tags[0].toxml() if (tags := node.getElementsByTagName("w:rPr")) else ""
replacement = f'<w:r w:rsidR="00AB12CD">{rpr}<w:t>The report is </w:t></w:r><w:del><w:r>{rpr}<w:delText>monthly</w:delText></w:r></w:del><w:ins><w:r>{rpr}<w:t>quarterly</w:t></w:r></w:ins>'
doc["word/document.xml"].replace_node(node, replacement)

# 最小化编辑 - 修改数字："within 30 days" → "within 45 days"
# 原始内容：<w:r w:rsidR="00XYZ789"><w:rPr><w:rFonts w:ascii="Calibri"/></w:rPr><w:t>within 30 days</w:t></w:r>
node = doc["word/document.xml"].get_node(tag="w:r", contains="within 30 days")
rpr = tags[0].toxml() if (tags := node.getElementsByTagName("w:rPr")) else ""
replacement = f'<w:r w:rsidR="00XYZ789">{rpr}<w:t>within </w:t></w:r><w:del><w:r>{rpr}<w:delText>30</w:delText></w:r></w:del><w:ins><w:r>{rpr}<w:t>45</w:t></w:r></w:ins><w:r w:rsidR="00XYZ789">{rpr}<w:t> days</w:t></w:r>'
doc["word/document.xml"].replace_node(node, replacement)

# 完全替换 - 替换全部文本时保留格式
node = doc["word/document.xml"].get_node(tag="w:r", contains="apple")
rpr = tags[0].toxml() if (tags := node.getElementsByTagName("w:rPr")) else ""
replacement = f'<w:del><w:r>{rpr}<w:delText>apple</w:delText></w:r></w:del><w:ins><w:r>{rpr}<w:t>banana orange</w:t></w:r></w:ins>'
doc["word/document.xml"].replace_node(node, replacement)

# 插入新内容（无需手动添加属性 - 自动注入）
node = doc["word/document.xml"].get_node(tag="w:r", contains="existing text")
doc["word/document.xml"].insert_after(node, '<w:ins><w:r><w:t>new text</w:t></w:r></w:ins>')

# 部分删除其他作者的插入内容
# 原始内容：<w:ins w:author="Jane Smith" w:date="..."><w:r><w:t>quarterly financial report</w:t></w:r></w:ins>
# 目标：仅删除"financial"，使其变为"quarterly report"
node = doc["word/document.xml"].get_node(tag="w:ins", attrs={"w:id": "5"})
# 重要：保留外层<w:ins>的w:author="Jane Smith"属性，以维护作者信息
replacement = '''<w:ins w:author="Jane Smith" w:date="2025-01-15T10:00:00Z">
  <w:r><w:t>quarterly </w:t></w:r>
  <w:del><w:r><w:delText>financial </w:delText></w:r></w:del>
  <w:r><w:t>report</w:t></w:r>
</w:ins>'''
doc["word/document.xml"].replace_node(node, replacement)

# 修改其他作者插入内容的部分文本
# 原始内容：<w:ins w:author="Jane Smith"><w:r><w:t>in silence, safe and sound</w:t></w:r></w:ins>
# 目标：将"safe and sound"改为"soft and unbound"
node = doc["word/document.xml"].get_node(tag="w:ins", attrs={"w:id": "8"})
replacement = f'''<w:ins w:author="Jane Smith" w:date="2025-01-15T10:00:00Z">
  <w:r><w:t>in silence, </w:t></w:r>
</w:ins>
<w:ins>
  <w:r><w:t>soft and unbound</w:t></w:r>
</w:ins>
<w:ins w:author="Jane Smith" w:date="2025-01-15T10:00:00Z">
  <w:del><w:r><w:delText>safe and sound</w:delText></w:r></w:del>
</w:ins>'''
doc["word/document.xml"].replace_node(node, replacement)

# 删除整个文本段（仅在删除全部内容时使用；部分删除请使用replace_node）
node = doc["word/document.xml"].get_node(tag="w:r", contains="text to delete")
doc["word/document.xml"].suggest_deletion(node)

# 删除整个段落（原地操作，支持普通段落和编号列表段落）
para = doc["word/document.xml"].get_node(tag="w:p", contains="paragraph to delete")
doc["word/document.xml"].suggest_deletion(para)

# 添加新的编号列表项
target_para = doc["word/document.xml"].get_node(tag="w:p", contains="existing list item")
pPr = tags[0].toxml() if (tags := target_para.getElementsByTagName("w:pPr")) else ""
new_item = f'<w:p>{pPr}<w:r><w:t>New item</w:t></w:r></w:p>'
tracked_para = DocxXMLEditor.suggest_paragraph(new_item)
doc["word/document.xml"].insert_after(target_para, tracked_para)
# 可选：在内容前添加空白段落以优化视觉分隔效果
# spacing = DocxXMLEditor.suggest_paragraph('<w:p><w:pPr><w:pStyle w:val="ListParagraph"/></w:pPr></w:p>')
# doc["word/document.xml"].insert_after(target_para, spacing + tracked_para)
```

### 添加批注

```Python

# 为两个现有修订记录添加批注
# 注意：w:id自动生成。仅当通过XML检查确认w:id时，才可通过w:id搜索
start_node = doc["word/document.xml"].get_node(tag="w:del", attrs={"w:id": "1"})
end_node = doc["word/document.xml"].get_node(tag="w:ins", attrs={"w:id": "2"})
doc.add_comment(start=start_node, end=end_node, text="Explanation of this change")

# 为段落添加批注
para = doc["word/document.xml"].get_node(tag="w:p", contains="paragraph text")
doc.add_comment(start=para, end=para, text="Comment on this paragraph")

# 为新建的修订记录添加批注
# 先创建修订记录
node = doc["word/document.xml"].get_node(tag="w:r", contains="old")
new_nodes = doc["word/document.xml"].replace_node(
    node,
    '<w:del><w:r><w:delText>old</w:delText></w:r></w:del><w:ins><w:r><w:t>new</w:t></w:r></w:ins>'
)
# 为新建的元素添加批注
# new_nodes[0] 是<w:del>，new_nodes[1] 是<w:ins>
doc.add_comment(start=new_nodes[0], end=new_nodes[1], text="Changed old to new per requirements")

# 回复现有批注
doc.reply_to_comment(parent_comment_id=0, text="I agree with this change")
```

### 拒绝修订跟踪记录

**重要提示**：使用 `revert_insertion()` 拒绝插入操作，使用 `revert_deletion()` 通过修订跟踪恢复被删除的内容。仅对未标记的普通内容使用 `suggest_deletion()`。

```Python

# 拒绝插入操作（将其包裹在删除标签中）
# 适用于删除其他作者插入的文本场景
ins = doc["word/document.xml"].get_node(tag="w:ins", attrs={"w:id": "5"})
nodes = doc["word/document.xml"].revert_insertion(ins)  # 返回 [ins]

# 拒绝删除操作（创建插入标签恢复被删除的内容）
# 适用于恢复其他作者删除的文本场景
del_elem = doc["word/document.xml"].get_node(tag="w:del", attrs={"w:id": "3"})
nodes = doc["word/document.xml"].revert_deletion(del_elem)  # 返回 [del_elem, new_ins]

# 拒绝段落中的所有插入操作
para = doc["word/document.xml"].get_node(tag="w:p", contains="paragraph text")
nodes = doc["word/document.xml"].revert_insertion(para)  # 返回 [para]

# 拒绝段落中的所有删除操作
para = doc["word/document.xml"].get_node(tag="w:p", contains="paragraph text")
nodes = doc["word/document.xml"].revert_deletion(para)  # 返回 [para]
```

### 插入图片

**关键要求**：Document 类操作的是 `doc.unpacked_path` 路径下的临时副本。务必将图片复制到该临时目录，而非原始解压目录。

```Python

from PIL import Image
import shutil, os

# 先初始化文档
doc = Document('unpacked')

# 复制图片并计算全屏尺寸（保持宽高比）
media_dir = os.path.join(doc.unpacked_path, 'word/media')
os.makedirs(media_dir, exist_ok=True)
shutil.copy('image.png', os.path.join(media_dir, 'image1.png'))
img = Image.open(os.path.join(media_dir, 'image1.png'))
width_emus = int(6.5 * 914400)  # 6.5英寸可用宽度，914400 EMU/英寸
height_emus = int(width_emus * img.size[1] / img.size[0])

# 添加关系和内容类型
rels_editor = doc['word/_rels/document.xml.rels']
next_rid = rels_editor.get_next_rid()
rels_editor.append_to(rels_editor.dom.documentElement,
    f'<Relationship Id="{next_rid}" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/image" Target="media/image1.png"/>')
doc['[Content_Types].xml'].append_to(doc['[Content_Types].xml'].dom.documentElement,
    '<Default Extension="png" ContentType="image/png"/>')

# 插入图片
node = doc["word/document.xml"].get_node(tag="w:p", line_number=100)
doc["word/document.xml"].insert_after(node, f'''<w:p>
  <w:r>
    <w:drawing>
      <wp:inline distT="0" distB="0" distL="0" distR="0">
        <wp:extent cx="{width_emus}" cy="{height_emus}"/>
        <wp:docPr id="1" name="Picture 1"/>
        <a:graphic xmlns:a="http://schemas.openxmlformats.org/drawingml/2006/main">
          <a:graphicData uri="http://schemas.openxmlformats.org/drawingml/2006/picture">
            <pic:pic xmlns:pic="http://schemas.openxmlformats.org/drawingml/2006/picture">
              <pic:nvPicPr><pic:cNvPr id="1" name="image1.png"/><pic:cNvPicPr/></pic:nvPicPr>
              <pic:blipFill><a:blip r:embed="{next_rid}"/><a:stretch><a:fillRect/></a:stretch></pic:blipFill>
              <pic:spPr><a:xfrm><a:ext cx="{width_emus}" cy="{height_emus}"/></a:xfrm><a:prstGeom prst="rect"><a:avLst/></a:prstGeom></pic:spPr>
            </pic:pic>
          </a:graphicData>
        </a:graphic>
      </wp:inline>
    </w:drawing>
  </w:r>
</w:p>''')
```

### 获取节点

```Python

# 通过文本内容获取
node = doc["word/document.xml"].get_node(tag="w:p", contains="specific text")

# 通过行号范围获取
para = doc["word/document.xml"].get_node(tag="w:p", line_number=range(100, 150))

# 通过属性获取
node = doc["word/document.xml"].get_node(tag="w:del", attrs={"w:id": "1"})

# 通过精确行号获取（行号必须为标签起始行）
para = doc["word/document.xml"].get_node(tag="w:p", line_number=42)

# 组合筛选条件
node = doc["word/document.xml"].get_node(tag="w:r", line_number=range(40, 60), contains="text")

# 文本重复出现时的歧义消除 - 添加行号范围
node = doc["word/document.xml"].get_node(tag="w:r", contains="Section", line_number=range(2400, 2500))
```

### 保存

```Python

# 带自动验证的保存（复制回原始目录）
doc.save()  # 默认启用验证，验证失败时抛出错误

# 保存到其他位置
doc.save('modified-unpacked')

# 跳过验证（仅用于调试 - 生产环境需跳过验证说明XML存在问题）
doc.save(validate=False)
```

### 直接 DOM 操作

适用于库未覆盖的复杂场景：

```Python

# 访问任意XML文件
editor = doc["word/document.xml"]
editor = doc["word/comments.xml"]

# 直接访问DOM（defusedxml.minidom.Document）
node = doc["word/document.xml"].get_node(tag="w:p", line_number=5)
parent = node.parentNode
parent.removeChild(node)
parent.appendChild(node)  # 移至末尾

# 通用文档操作（无修订跟踪）
old_node = doc["word/document.xml"].get_node(tag="w:p", contains="original text")
doc["word/document.xml"].replace_node(old_node, "<w:p><w:r><w:t>replacement text</w:t></w:r></w:p>")

# 多次插入 - 使用返回值维持顺序
node = doc["word/document.xml"].get_node(tag="w:r", line_number=100)
nodes = doc["word/document.xml"].insert_after(node, "<w:r><w:t>A</w:t></w:r>")
nodes = doc["word/document.xml"].insert_after(nodes[-1], "<w:r><w:t>B</w:t></w:r>")
nodes = doc["word/document.xml"].insert_after(nodes[-1], "<w:r><w:t>C</w:t></w:r>")
# 结果顺序：original_node, A, B, C
```

## 修订跟踪（红线标注）

**所有修订跟踪操作均使用上述 Document 类。** 以下格式仅用于构建替换 XML 字符串时参考。

### 验证规则

验证器会检查：还原 Claude 的修订后，文档文本是否与原始文本一致。这意味着：

- **严禁修改其他作者** **`<w:ins>`** **或** **`<w:del>`** **标签内的文本**

- **必须使用嵌套删除**来移除其他作者的插入内容

- **所有编辑操作必须通过** **`<w:ins>`** **或** **`<w:del>`** **标签正确标记**

### 修订跟踪格式

**核心规则**：

1. 严禁修改其他作者修订内容内的文本。必须使用嵌套删除方式处理。

2. **XML 结构**：`<w:del>` 和 `<w:ins>` 必须置于包含完整`<w:r>`元素的段落层级。严禁嵌套在`<w:r>`元素内——此操作会生成无效 XML，导致文档处理异常。

**文本插入：**

```XML

<w:ins w:id="1" w:author="Claude" w:date="2025-07-30T23:05:00Z" w16du:dateUtc="2025-07-31T06:05:00Z">
  <w:r w:rsidR="00792858">
    <w:t>inserted text</w:t>
  </w:r>
</w:ins>
```

**文本删除：**

```XML

<w:del w:id="2" w:author="Claude" w:date="2025-07-30T23:05:00Z" w16du:dateUtc="2025-07-31T06:05:00Z">
  <w:r w:rsidDel="00792858">
    <w:delText>deleted text</w:delText>
  </w:r>
</w:del>
```

**删除其他作者的插入内容（必须使用嵌套结构）：**

```XML

<!-- 将删除操作嵌套在原始插入标签内 -->
<w:ins w:author="Jane Smith" w:id="16">
  <w:del w:author="Claude" w:id="40">
    <w:r><w:delText>monthly</w:delText></w:r>
  </w:del>
</w:ins>
<w:ins w:author="Claude" w:id="41">
  <w:r><w:t>weekly</w:t></w:r>
</w:ins>
```

**恢复其他作者的删除内容：**

```XML

<!-- 保留其删除标签不变，在其后添加新的插入标签 -->
<w:del w:author="Jane Smith" w:id="50">
  <w:r><w:delText>within 30 days</w:delText></w:r>
</w:del>
<w:ins w:author="Claude" w:id="51">
  <w:r><w:t>within 30 days</w:t></w:r>
</w:ins>
```
