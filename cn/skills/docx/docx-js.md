# DOCX 库教程

使用 JavaScript/TypeScript 生成.docx 文件。

**重要提示：开始前请阅读整个文档。** 整个文档中涵盖了关键的格式规则和常见问题 - 跳过某些部分可能导致文件损坏或渲染问题。

## 设置

假设 docx 已全局安装
如果未安装：`npm install -g docx`

```javascript
const {
  Document,
  Packer,
  Paragraph,
  TextRun,
  Table,
  TableRow,
  TableCell,
  ImageRun,
  Media,
  Header,
  Footer,
  AlignmentType,
  PageOrientation,
  LevelFormat,
  ExternalHyperlink,
  InternalHyperlink,
  TableOfContents,
  HeadingLevel,
  BorderStyle,
  WidthType,
  TabStopType,
  TabStopPosition,
  UnderlineType,
  ShadingType,
  VerticalAlign,
  SymbolRun,
  PageNumber,
  FootnoteReferenceRun,
  Footnote,
  PageBreak,
} = require("docx");

// 创建和保存
const doc = new Document({
  sections: [
    {
      children: [
        /* 内容 */
      ],
    },
  ],
});
Packer.toBuffer(doc).then((buffer) => fs.writeFileSync("doc.docx", buffer)); // Node.js
Packer.toBlob(doc).then((blob) => {
  /* 下载逻辑 */
}); // 浏览器
```

## 文本与格式

```javascript
// 重要：切勿使用\n换行 - 始终使用单独的Paragraph元素
// ❌ 错误: new TextRun("第1行\n第2行")
// ✅ 正确: new Paragraph({ children: [new TextRun("第1行")] }), new Paragraph({ children: [new TextRun("第2行")] })

// 包含所有格式选项的基本文本
new Paragraph({
  alignment: AlignmentType.CENTER,
  spacing: { before: 200, after: 200 },
  indent: { left: 720, right: 720 },
  children: [
    new TextRun({ text: "粗体", bold: true }),
    new TextRun({ text: "斜体", italics: true }),
    new TextRun({
      text: "下划线",
      underline: { type: UnderlineType.DOUBLE, color: "FF0000" },
    }),
    new TextRun({ text: "有色", color: "FF0000", size: 28, font: "Arial" }), // Arial默认
    new TextRun({ text: "高亮", highlight: "yellow" }),
    new TextRun({ text: "删除线", strike: true }),
    new TextRun({ text: "x2", superScript: true }),
    new TextRun({ text: "H2O", subScript: true }),
    new TextRun({ text: "小型大写字母", smallCaps: true }),
    new SymbolRun({ char: "2022", font: "Symbol" }), // 项目符号 •
    new SymbolRun({ char: "00A9", font: "Arial" }), // 版权符号 © - Arial用于符号
  ],
});
```

## 样式与专业格式

```javascript
const doc = new Document({
  styles: {
    default: { document: { run: { font: "Arial", size: 24 } } }, // 12pt默认
    paragraphStyles: [
      // 文档标题样式 - 覆盖内置Title样式
      {
        id: "Title",
        name: "Title",
        basedOn: "Normal",
        run: { size: 56, bold: true, color: "000000", font: "Arial" },
        paragraph: {
          spacing: { before: 240, after: 120 },
          alignment: AlignmentType.CENTER,
        },
      },
      // 重要：通过使用其确切ID覆盖内置标题样式
      {
        id: "Heading1",
        name: "Heading 1",
        basedOn: "Normal",
        next: "Normal",
        quickFormat: true,
        run: { size: 32, bold: true, color: "000000", font: "Arial" }, // 16pt
        paragraph: { spacing: { before: 240, after: 240 }, outlineLevel: 0 },
      }, // 目录所需
      {
        id: "Heading2",
        name: "Heading 2",
        basedOn: "Normal",
        next: "Normal",
        quickFormat: true,
        run: { size: 28, bold: true, color: "000000", font: "Arial" }, // 14pt
        paragraph: { spacing: { before: 180, after: 180 }, outlineLevel: 1 },
      },
      // 自定义样式使用您自己的ID
      {
        id: "myStyle",
        name: "My Style",
        basedOn: "Normal",
        run: { size: 28, bold: true, color: "000000" },
        paragraph: { spacing: { after: 120 }, alignment: AlignmentType.CENTER },
      },
    ],
    characterStyles: [
      {
        id: "myCharStyle",
        name: "My Char Style",
        run: {
          color: "FF0000",
          bold: true,
          underline: { type: UnderlineType.SINGLE },
        },
      },
    ],
  },
  sections: [
    {
      properties: {
        page: { margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 } },
      },
      children: [
        new Paragraph({
          heading: HeadingLevel.TITLE,
          children: [new TextRun("文档标题")],
        }), // 使用覆盖的Title样式
        new Paragraph({
          heading: HeadingLevel.HEADING_1,
          children: [new TextRun("标题1")],
        }), // 使用覆盖的Heading1样式
        new Paragraph({
          style: "myStyle",
          children: [new TextRun("自定义段落样式")],
        }),
        new Paragraph({
          children: [
            new TextRun("普通带有"),
            new TextRun({ text: "自定义字符样式", style: "myCharStyle" }),
          ],
        }),
      ],
    },
  ],
});
```

**专业字体组合：**

- **Arial（标题）+ Arial（正文）** - 最具通用支持，干净专业
- **Times New Roman（标题）+ Arial（正文）** - 经典衬线标题配现代无衬线正文
- **Georgia（标题）+ Verdana（正文）** - 针对屏幕阅读优化，优雅对比

**关键样式原则：**

- **覆盖内置样式**：使用确切 ID 如"Heading1"、"Heading2"、"Heading3"来覆盖 Word 的内置标题样式
- **HeadingLevel 常量**：`HeadingLevel.HEADING_1`使用"Heading1"样式，`HeadingLevel.HEADING_2`使用"Heading2"样式，等等
- **包含 outlineLevel**：为 H1 设置`outlineLevel: 0`，为 H2 设置`outlineLevel: 1`，等等，以确保目录正常工作
- **使用自定义样式**而非内联格式以保持一致性
- **设置默认字体**使用`styles.default.document.run.font` - Arial 得到普遍支持
- **建立视觉层次结构**，使用不同字体大小（标题>标题>正文）
- **添加适当间距**，使用段落的`before`和`after`间距
- **谨慎使用颜色**：默认为黑色(000000)和灰色阴影作为标题和标题（标题 1，标题 2 等）
- **设置一致的页边距**（1440 = 1 英寸是标准）

## 列表（始终使用正确的列表 - 切勿使用 Unicode 项目符号）

```javascript
// 项目符号 - 始终使用numbering配置，不要使用unicode符号
// 关键：使用LevelFormat.BULLET常量，而不是字符串"bullet"
const doc = new Document({
  numbering: {
    config: [
      {
        reference: "bullet-list",
        levels: [
          {
            level: 0,
            format: LevelFormat.BULLET,
            text: "•",
            alignment: AlignmentType.LEFT,
            style: { paragraph: { indent: { left: 720, hanging: 360 } } },
          },
        ],
      },
      {
        reference: "first-numbered-list",
        levels: [
          {
            level: 0,
            format: LevelFormat.DECIMAL,
            text: "%1.",
            alignment: AlignmentType.LEFT,
            style: { paragraph: { indent: { left: 720, hanging: 360 } } },
          },
        ],
      },
      {
        reference: "second-numbered-list", // 不同引用 = 重新从1开始
        levels: [
          {
            level: 0,
            format: LevelFormat.DECIMAL,
            text: "%1.",
            alignment: AlignmentType.LEFT,
            style: { paragraph: { indent: { left: 720, hanging: 360 } } },
          },
        ],
      },
    ],
  },
  sections: [
    {
      children: [
        // 项目符号列表项
        new Paragraph({
          numbering: { reference: "bullet-list", level: 0 },
          children: [new TextRun("第一个项目点")],
        }),
        new Paragraph({
          numbering: { reference: "bullet-list", level: 0 },
          children: [new TextRun("第二个项目点")],
        }),
        // 编号列表项
        new Paragraph({
          numbering: { reference: "first-numbered-list", level: 0 },
          children: [new TextRun("第一个编号项")],
        }),
        new Paragraph({
          numbering: { reference: "first-numbered-list", level: 0 },
          children: [new TextRun("第二个编号项")],
        }),
        // ⚠️ 关键：不同引用 = 独立列表，重新从1开始
        // 相同引用 = 继续前一个编号
        new Paragraph({
          numbering: { reference: "second-numbered-list", level: 0 },
          children: [new TextRun("再次从1开始（因为使用不同引用）")],
        }),
      ],
    },
  ],
});

// ⚠️ 关键编号规则：每个引用创建一个独立的编号列表
// - 相同引用 = 继续编号（1, 2, 3... 然后 4, 5, 6...）
// - 不同引用 = 重新从1开始（1, 2, 3... 然后 1, 2, 3...）
// 为每个单独的编号部分使用唯一的引用名称！

// ⚠️ 关键：切勿使用unicode项目符号 - 它们创建不能正常工作的假列表
// new TextRun("• 项目")           // 错误
// new SymbolRun({ char: "2022" }) // 错误
// ✅ 始终使用numbering配置与LevelFormat.BULLET为真正的Word列表
```

## 表格

```javascript
// 完整表格，带边距、边框、表头和项目符号
const tableBorder = { style: BorderStyle.SINGLE, size: 1, color: "CCCCCC" };
const cellBorders = {
  top: tableBorder,
  bottom: tableBorder,
  left: tableBorder,
  right: tableBorder,
};

new Table({
  columnWidths: [4680, 4680], // ⚠️ 关键：在表格级别设置列宽 - 值以DXA为单位（点的二十分之一）
  margins: { top: 100, bottom: 100, left: 180, right: 180 }, // 为所有单元格一次性设置
  rows: [
    new TableRow({
      tableHeader: true,
      children: [
        new TableCell({
          borders: cellBorders,
          width: { size: 4680, type: WidthType.DXA }, // 同时也为每个单元格设置宽度
          // ⚠️ 关键：始终使用ShadingType.CLEAR防止在Word中出现黑色背景
          shading: { fill: "D5E8F0", type: ShadingType.CLEAR },
          verticalAlign: VerticalAlign.CENTER,
          children: [
            new Paragraph({
              alignment: AlignmentType.CENTER,
              children: [new TextRun({ text: "表头", bold: true, size: 22 })],
            }),
          ],
        }),
        new TableCell({
          borders: cellBorders,
          width: { size: 4680, type: WidthType.DXA }, // 同时也为每个单元格设置宽度
          shading: { fill: "D5E8F0", type: ShadingType.CLEAR },
          children: [
            new Paragraph({
              alignment: AlignmentType.CENTER,
              children: [
                new TextRun({ text: "项目符号", bold: true, size: 22 }),
              ],
            }),
          ],
        }),
      ],
    }),
    new TableRow({
      children: [
        new TableCell({
          borders: cellBorders,
          width: { size: 4680, type: WidthType.DXA }, // 同时也为每个单元格设置宽度
          children: [new Paragraph({ children: [new TextRun("普通数据")] })],
        }),
        new TableCell({
          borders: cellBorders,
          width: { size: 4680, type: WidthType.DXA }, // 同时也为每个单元格设置宽度
          children: [
            new Paragraph({
              numbering: { reference: "bullet-list", level: 0 },
              children: [new TextRun("第一个项目点")],
            }),
            new Paragraph({
              numbering: { reference: "bullet-list", level: 0 },
              children: [new TextRun("第二个项目点")],
            }),
          ],
        }),
      ],
    }),
  ],
});
```

**重要：表格宽度与边框**

- 同时使用`columnWidths: [width1, width2, ...]`数组和每个单元格的`width: { size: X, type: WidthType.DXA }`以确保兼容性
- 值以 DXA 为单位（点的二十分之一）：1440 = 1 英寸，信纸可用宽度 = 9360 DXA（1 英寸页边距）
- 将边框应用到个别`TableCell`元素，而非`Table`本身

**预计算列宽（信纸尺寸，1 英寸页边距 = 总计 9360 DXA）：**

- **2 列：** `columnWidths: [4680, 4680]`（等宽）
- **3 列：** `columnWidths: [3120, 3120, 3120]`（等宽）

## 链接与导航

```javascript
// 目录（需要标题）- 关键：仅使用HeadingLevel，不要使用自定义样式
// ❌ 错误: new Paragraph({ heading: HeadingLevel.HEADING_1, style: "customHeader", children: [new TextRun("标题")] })
// ✅ 正确: new Paragraph({ heading: HeadingLevel.HEADING_1, children: [new TextRun("标题")] })
new TableOfContents("目录", { hyperlink: true, headingStyleRange: "1-3" }),

// 外部链接
new Paragraph({
  children: [new ExternalHyperlink({
    children: [new TextRun({ text: "谷歌", style: "Hyperlink" })],
    link: "https://www.google.com"
  })]
}),

// 内部链接和书签
new Paragraph({
  children: [new InternalHyperlink({
    children: [new TextRun({ text: "跳转到章节", style: "Hyperlink" })],
    anchor: "section1"
  })]
}),
new Paragraph({
  children: [new TextRun("章节内容")],
  bookmark: { id: "section1", name: "section1" }
}),
```

## 图像和媒体

```javascript
// 基本图像，带大小和位置设置
// 关键：始终指定'type'参数 - 这是ImageRun所必需的
new Paragraph({
  alignment: AlignmentType.CENTER,
  children: [
    new ImageRun({
      type: "png", // 新要求：必须指定图像类型(png, jpg, jpeg, gif, bmp, svg)
      data: fs.readFileSync("image.png"),
      transformation: { width: 200, height: 150, rotation: 0 }, // 旋转角度
      altText: { title: "Logo", description: "公司标志", name: "Name" }, // 重要：三个字段都是必需的
    }),
  ],
});
```

## 分页符

```javascript
// 手动分页符
new Paragraph({ children: [new PageBreak()] }),
  // 段落前分页符
  new Paragraph({
    pageBreakBefore: true,
    children: [new TextRun("这将从新页面开始")],
  });

// ⚠️ 关键：切勿单独使用PageBreak - 它将创建Word无法打开的无效XML
// ❌ 错误: new PageBreak()
// ✅ 正确: new Paragraph({ children: [new PageBreak()] })
```

## 页眉/页脚和页面设置

```javascript
const doc = new Document({
  sections: [
    {
      properties: {
        page: {
          margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 }, // 1440 = 1英寸
          size: { orientation: PageOrientation.LANDSCAPE },
          pageNumbers: { start: 1, formatType: "decimal" }, // "upperRoman", "lowerRoman", "upperLetter", "lowerLetter"
        },
      },
      headers: {
        default: new Header({
          children: [
            new Paragraph({
              alignment: AlignmentType.RIGHT,
              children: [new TextRun("页眉文本")],
            }),
          ],
        }),
      },
      footers: {
        default: new Footer({
          children: [
            new Paragraph({
              alignment: AlignmentType.CENTER,
              children: [
                new TextRun("第 "),
                new TextRun({ children: [PageNumber.CURRENT] }),
                new TextRun(" 页，共 "),
                new TextRun({ children: [PageNumber.TOTAL_PAGES] }),
                new TextRun(" 页"),
              ],
            }),
          ],
        }),
      },
      children: [
        /* 内容 */
      ],
    },
  ],
});
```

## 制表位

```javascript
new Paragraph({
  tabStops: [
    { type: TabStopType.LEFT, position: TabStopPosition.MAX / 4 },
    { type: TabStopType.CENTER, position: TabStopPosition.MAX / 2 },
    { type: TabStopType.RIGHT, position: (TabStopPosition.MAX * 3) / 4 },
  ],
  children: [new TextRun("左\t中\t右")],
});
```

## 常量和快速参考

- **下划线：** `SINGLE`, `DOUBLE`, `WAVY`, `DASH`
- **边框：** `SINGLE`, `DOUBLE`, `DASHED`, `DOTTED`
- **编号：** `DECIMAL` (1,2,3), `UPPER_ROMAN` (I,II,III), `LOWER_LETTER` (a,b,c)
- **制表位：** `LEFT`, `CENTER`, `RIGHT`, `DECIMAL`
- **符号：** `"2022"` (•), `"00A9"` (©), `"00AE"` (®), `"2122"` (™), `"00B0"` (°), `"F070"` (✓), `"F0FC"` (✗)

## 关键问题和常见错误

- **关键：PageBreak 必须始终在 Paragraph 内部** - 独立的 PageBreak 会创建 Word 无法打开的无效 XML
- **始终为表格单元格阴影使用 ShadingType.CLEAR** - 切勿使用 ShadingType.SOLID（会导致黑色背景）
- 测量单位为 DXA（1440 = 1 英寸）| 每个表格单元格需要 ≥1 个 Paragraph | 目录需要仅使用 HeadingLevel 样式
- **始终使用带 Arial 字体的自定义样式**，以获得专业外观和适当的视觉层次结构
- **始终设置默认字体**，使用`styles.default.document.run.font` - 建议使用 Arial
- **始终为表格使用 columnWidths 数组** + 单个单元格宽度，以确保兼容性
- **切勿使用 unicode 符号作为项目符号** - 始终使用带有`LevelFormat.BULLET`常量的适当编号配置（不是字符串"bullet"）
- **切勿在任何地方使用\n 进行换行** - 始终为每行使用单独的 Paragraph 元素
- **始终在 Paragraph 的 children 内使用 TextRun 对象** - 切勿直接在 Paragraph 上使用 text 属性
- **图像关键点**：ImageRun 需要`type`参数 - 始终指定"png"、"jpg"、"jpeg"、"gif"、"bmp"或"svg"
- **项目符号关键点**：必须使用`LevelFormat.BULLET`常量，而非字符串"bullet"，并且包括`text: "•"`作为项目符号字符
- **编号关键点**：每个编号引用创建一个独立列表。相同引用 = 继续编号（1,2,3 然后 4,5,6）。不同引用 = 重新从 1 开始（1,2,3 然后 1,2,3）。为每个单独的编号部分使用唯一的引用名称！
- **目录关键点**：使用 TableOfContents 时，标题必须仅使用 HeadingLevel - 不要向标题段落添加自定义样式，否则目录将失效
- **表格**：设置`columnWidths`数组 + 单个单元格宽度，将边框应用到单元格而非表格
- **在表格级别设置表格边距**，以获得一致的单元格内边距（避免每个单元格重复）
