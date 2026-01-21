# Office Open XML PowerPoint 技术参考文档

**重要提示：开始操作前请通读本文档。** 文档通篇涵盖了关键的 XML 模式规则和格式要求。若实现方式不正确，可能会生成 PowerPoint 无法打开的无效 PPTX 文件。

## 技术规范

### 模式合规性

- `<p:txBody>` 中的元素顺序：`<a:bodyPr>`、`<a:lstStyle>`、`<a:p>`

- 空白字符：为包含前导/尾随空格的 `<a:t>` 元素添加 `xml:space='preserve'` 属性

- 统一码（Unicode）：对 ASCII 内容中的特殊字符进行转义，例如双引号 `"` 需转为 `&#8220;`

- 图片：将图片放入 `ppt/media/` 目录，在幻灯片 XML 中引用，并设置尺寸以适配幻灯片边界

- 关联关系：为每张幻灯片的资源更新 `ppt/slides/_rels/slideN.xml.rels` 文件

- 脏数据属性：为 `<a:rPr>` 和 `<a:endParaRPr>` 元素添加 `dirty="0"` 属性，标识为干净状态

## 演示文稿结构

### 基本幻灯片结构

```XML

<!-- ppt/slides/slide1.xml -->
<p:sld>
  <p:cSld>
    <p:spTree>
      <p:nvGrpSpPr>...</p:nvGrpSpPr>
      <p:grpSpPr>...</p:grpSpPr>
      <!-- 形状定义区域 -->
    </p:spTree>
  </p:cSld>
</p:sld>
```

### 带文本的文本框/形状

```XML

<p:sp>
  <p:nvSpPr>
    <p:cNvPr id="2" name="Title"/>
    <p:cNvSpPr>
      <a:spLocks noGrp="1"/>
    </p:cNvSpPr>
    <p:nvPr>
      <p:ph type="ctrTitle"/>
    </p:nvPr>
  </p:nvSpPr>
  <p:spPr>
    <a:xfrm>
      <a:off x="838200" y="365125"/>
      <a:ext cx="7772400" cy="1470025"/>
    </a:xfrm>
  </p:spPr>
  <p:txBody>
    <a:bodyPr/>
    <a:lstStyle/>
    <a:p>
      <a:r>
        <a:t>Slide Title</a:t>
      </a:r>
    </a:p>
  </p:txBody>
</p:sp>
```

### 文本格式设置

```XML

<!-- 加粗 -->
<a:r>
  <a:rPr b="1"/>
  <a:t>Bold Text</a:t>
</a:r>

<!-- 斜体 -->
<a:r>
  <a:rPr i="1"/>
  <a:t>Italic Text</a:t>
</a:r>

<!-- 下划线 -->
<a:r>
  <a:rPr u="sng"/>
  <a:t>Underlined</a:t>
</a:r>

<!-- 高亮 -->
<a:r>
  <a:rPr>
    <a:highlight>
      <a:srgbClr val="FFFF00"/>
    </a:highlight>
  </a:rPr>
  <a:t>Highlighted Text</a:t>
</a:r>

<!-- 字体和字号 -->
<a:r>
  <a:rPr sz="2400" typeface="Arial">
    <a:solidFill>
      <a:srgbClr val="FF0000"/>
    </a:solidFill>
  </a:rPr>
  <a:t>Colored Arial 24pt</a:t>
</a:r>

<!-- 完整格式设置示例 -->
<a:r>
  <a:rPr lang="en-US" sz="1400" b="1" dirty="0">
    <a:solidFill>
      <a:srgbClr val="FAFAFA"/>
    </a:solidFill>
  </a:rPr>
  <a:t>Formatted text</a:t>
</a:r>
```

### 列表

```XML

<!-- 项目符号列表 -->
<a:p>
  <a:pPr lvl="0">
    <a:buChar char="•"/>
  </a:pPr>
  <a:r>
    <a:t>First bullet point</a:t>
  </a:r>
</a:p>

<!-- 编号列表 -->
<a:p>
  <a:pPr lvl="0">
    <a:buAutoNum type="arabicPeriod"/>
  </a:pPr>
  <a:r>
    <a:t>First numbered item</a:t>
  </a:r>
</a:p>

<!-- 二级缩进 -->
<a:p>
  <a:pPr lvl="1">
    <a:buChar char="•"/>
  </a:pPr>
  <a:r>
    <a:t>Indented bullet</a:t>
  </a:r>
</a:p>
```

### 形状

```XML

<!-- 矩形 -->
<p:sp>
  <p:nvSpPr>
    <p:cNvPr id="3" name="Rectangle"/>
    <p:cNvSpPr/>
    <p:nvPr/>
  </p:nvSpPr>
  <p:spPr>
    <a:xfrm>
      <a:off x="1000000" y="1000000"/>
      <a:ext cx="3000000" cy="2000000"/>
    </a:xfrm>
    <a:prstGeom prst="rect">
      <a:avLst/>
    </a:prstGeom>
    <a:solidFill>
      <a:srgbClr val="FF0000"/>
    </a:solidFill>
    <a:ln w="25400">
      <a:solidFill>
        <a:srgbClr val="000000"/>
      </a:solidFill>
    </a:ln>
  </p:spPr>
</p:sp>

<!-- 圆角矩形 -->
<p:sp>
  <p:spPr>
    <a:prstGeom prst="roundRect">
      <a:avLst/>
    </a:prstGeom>
  </p:spPr>
</p:sp>

<!-- 圆形/椭圆形 -->
<p:sp>
  <p:spPr>
    <a:prstGeom prst="ellipse">
      <a:avLst/>
    </a:prstGeom>
  </p:spPr>
</p:sp>
```

### 图片

```XML

<p:pic>
  <p:nvPicPr>
    <p:cNvPr id="4" name="Picture">
      <a:hlinkClick r:id="" action="ppaction://media"/>
    </p:cNvPr>
    <p:cNvPicPr>
      <a:picLocks noChangeAspect="1"/>
    </p:cNvPicPr>
    <p:nvPr/>
  </p:nvPicPr>
  <p:blipFill>
    <a:blip r:embed="rId2"/>
    <a:stretch>
      <a:fillRect/>
    </a:stretch>
  </p:blipFill>
  <p:spPr>
    <a:xfrm>
      <a:off x="1000000" y="1000000"/>
      <a:ext cx="3000000" cy="2000000"/>
    </a:xfrm>
    <a:prstGeom prst="rect">
      <a:avLst/>
    </a:prstGeom>
  </p:spPr>
</p:pic>
```

### 表格

```XML

<p:graphicFrame>
  <p:nvGraphicFramePr>
    <p:cNvPr id="5" name="Table"/>
    <p:cNvGraphicFramePr>
      <a:graphicFrameLocks noGrp="1"/>
    </p:cNvGraphicFramePr>
    <p:nvPr/>
  </p:nvGraphicFramePr>
  <p:xfrm>
    <a:off x="1000000" y="1000000"/>
    <a:ext cx="6000000" cy="2000000"/>
  </p:xfrm>
  <a:graphic>
    <a:graphicData uri="http://schemas.openxmlformats.org/drawingml/2006/table">
      <a:tbl>
        <a:tblGrid>
          <a:gridCol w="3000000"/>
          <a:gridCol w="3000000"/>
        </a:tblGrid>
        <a:tr h="500000">
          <a:tc>
            <a:txBody>
              <a:bodyPr/>
              <a:lstStyle/>
              <a:p>
                <a:r>
                  <a:t>Cell 1</a:t>
                </a:r>
              </a:p>
            </a:txBody>
          </a:tc>
          <a:tc>
            <a:txBody>
              <a:bodyPr/>
              <a:lstStyle/>
              <a:p>
                <a:r>
                  <a:t>Cell 2</a:t>
                </a:r>
              </a:p>
            </a:txBody>
          </a:tc>
        </a:tr>
      </a:tbl>
    </a:graphicData>
  </a:graphic>
</p:graphicFrame>
```

### 幻灯片版式

```XML

<!-- 标题幻灯片版式 -->
<p:sp>
  <p:nvSpPr>
    <p:nvPr>
      <p:ph type="ctrTitle"/>
    </p:nvPr>
  </p:nvSpPr>
  <!-- 标题内容 -->
</p:sp>

<p:sp>
  <p:nvSpPr>
    <p:nvPr>
      <p:ph type="subTitle" idx="1"/>
    </p:nvPr>
  </p:nvSpPr>
  <!-- 副标题内容 -->
</p:sp>

<!-- 内容幻灯片版式 -->
<p:sp>
  <p:nvSpPr>
    <p:nvPr>
      <p:ph type="title"/>
    </p:nvPr>
  </p:nvSpPr>
  <!-- 幻灯片标题 -->
</p:sp>

<p:sp>
  <p:nvSpPr>
    <p:nvPr>
      <p:ph type="body" idx="1"/>
    </p:nvPr>
  </p:nvSpPr>
  <!-- 内容主体 -->
</p:sp>
```

## 文件更新

添加内容时，需更新以下文件：

**`ppt/_rels/presentation.xml.rels`** **:**

```XML

<Relationship Id="rId1" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/slide" Target="slides/slide1.xml"/>
<Relationship Id="rId2" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/slideMaster" Target="slideMasters/slideMaster1.xml"/>
```

**`ppt/slides/_rels/slide1.xml.rels`** **:**

```XML

<Relationship Id="rId1" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/slideLayout" Target="../slideLayouts/slideLayout1.xml"/>
<Relationship Id="rId2" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/image" Target="../media/image1.png"/>
```

**`[Content_Types].xml`** **:**

```XML

<Default Extension="png" ContentType="image/png"/>
<Default Extension="jpg" ContentType="image/jpeg"/>
<Override PartName="/ppt/slides/slide1.xml" ContentType="application/vnd.openxmlformats-officedocument.presentationml.slide+xml"/>
```

**`ppt/presentation.xml`** **:**

```XML

<p:sldIdLst>
  <p:sldId id="256" r:id="rId1"/>
  <p:sldId id="257" r:id="rId2"/>
</p:sldIdLst>
```

**`docProps/app.xml`** **:** 更新幻灯片数量和统计信息

```XML

<Slides>2</Slides>
<Paragraphs>10</Paragraphs>
<Words>50</Words>
```

## 幻灯片操作

### 新增幻灯片

在演示文稿末尾添加幻灯片的步骤：

1. **创建幻灯片文件**（`ppt/slides/slideN.xml`）

2. **更新 ** **`[Content_Types].xml`**：为新幻灯片添加 Override 节点

3. **更新 ** **`ppt/_rels/presentation.xml.rels`**：为新幻灯片添加关联关系

4. **更新 ** **`ppt/presentation.xml`**：在 `<p:sldIdLst>` 中添加幻灯片 ID

5. **创建幻灯片关联文件**（若需要，创建 `ppt/slides/_rels/slideN.xml.rels`）

6. **更新 ** **`docProps/app.xml`**：增加幻灯片计数，并更新相关统计信息（如有）

### 复制幻灯片

1. 复制源幻灯片 XML 文件并命名为新文件名

2. 更新新幻灯片中的所有 ID，确保唯一性

3. 按照上述「新增幻灯片」步骤操作

4. **关键操作**：移除或更新 `_rels` 文件中所有备注幻灯片的引用

5. 移除对未使用媒体文件的引用

### 调整幻灯片顺序

1. **更新 ** **`ppt/presentation.xml`**：调整 `<p:sldIdLst>` 中 `<p:sldId>` 元素的顺序

2. `<p:sldId>` 元素的顺序决定了幻灯片的展示顺序

3. 保留幻灯片 ID 和关联关系 ID 不变

示例：

```XML

<!-- 原始顺序 -->
<p:sldIdLst>
  <p:sldId id="256" r:id="rId2"/>
  <p:sldId id="257" r:id="rId3"/>
  <p:sldId id="258" r:id="rId4"/>
</p:sldIdLst>

<!-- 将第3张幻灯片移至第2位后的顺序 -->
<p:sldIdLst>
  <p:sldId id="256" r:id="rId2"/>
  <p:sldId id="258" r:id="rId4"/>
  <p:sldId id="257" r:id="rId3"/>
</p:sldIdLst>
```

### 删除幻灯片

1. **从 ** **`ppt/presentation.xml`** ** 中移除**：删除对应的 `<p:sldId>` 节点

2. **从 ** **`ppt/_rels/presentation.xml.rels`** ** 中移除**：删除对应的关联关系

3. **从 ** **`[Content_Types].xml`** ** 中移除**：删除对应的 Override 节点

4. **删除文件**：移除 `ppt/slides/slideN.xml` 和 `ppt/slides/_rels/slideN.xml.rels`

5. **更新 ** **`docProps/app.xml`**：减少幻灯片计数，并更新相关统计信息

6. **清理未使用媒体文件**：从 `ppt/media/` 中移除无引用的图片

注意：无需对剩余幻灯片重新编号，保留其原始 ID 和文件名即可。

## 需避免的常见错误

- **编码问题**：对 ASCII 内容中的 Unicode 字符进行转义，例如双引号 `"` 需转为 `&#8220;`

- **图片问题**：将图片放入 `ppt/media/` 目录并更新关联关系文件

- **列表问题**：列表标题中不要包含项目符号

- **ID 问题**：UUID 需使用有效的十六进制值

- **主题问题**：检查 `theme` 目录下所有主题的颜色配置

## 基于模板的演示文稿验证清单

### 打包前务必执行：

- **清理未使用资源**：移除未引用的媒体、字体和备注目录

- **修正 Content_Types.xml**：声明包中存在的所有幻灯片、版式和主题

- **修正关联关系 ID**：

  - 若未使用嵌入式字体，移除字体嵌入引用

- **移除无效引用**：检查所有 `_rels` 文件，删除指向已删除资源的引用

### 模板复制常见陷阱：

- 复制后多张幻灯片引用同一张备注幻灯片

- 模板幻灯片引用了已不存在的图片/媒体文件

- 未包含字体却保留了字体嵌入引用

- 遗漏 12-25 号版式的幻灯片版式声明

- docProps 目录可能无法解包（该目录为可选目录）
  > （注：文档部分内容可能由 AI 生成）
