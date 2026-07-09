# Word 转 PDF 工具页上线规格

## 页面定位

- 中文 URL：`/zh/tools/word-to-pdf`
- 英文 URL：`/tools/word-to-pdf`
- 日文 URL：`/ja/tools/word-to-pdf`
- 韩文 URL：`/ko/tools/word-to-pdf`
- 页面用途：帮助用户上传 Word 文档并保存为 PDF，同时提供在 Word、Word Online、iPad 等环境中手动导出 PDF 的方法。

## 文案与本地化约束

- 面向用户的页面文案必须像目标语言的原生文案，避免直译和机翻腔。
- 中文页面必须使用完整、自然的中文。除 Word、PDF、DOC、DOCX、RTF、PDF/A、Kollab、Markdown、Windows、Mac、iPad 等专有名词或格式名外，不夹杂未翻译的英文短语、占位符、Fallback Key 或代码变量名。
- 专有名词保留官方大小写，例如 Kollab、Instagram、TikTok。
- 如果英文搜索句本身需要保留，例如 `how to save a word document as a pdf`，必须放在自然解释里，不能堆砌。
- 不写内部 SEO 策略，不暴露“关键词承接”“搜索意图”等面向团队的说法。页面只直接说明工具能帮用户完成什么、输入什么、得到什么。

## SEO 元信息

```ts
export const metadata = {
  title: "Word 转 PDF 在线工具：如何将 Word 保存为 PDF | Kollab",
  description:
    "上传 DOC 或 DOCX 文件，在线将 Word 保存为 PDF。保留排版、图片、表格和链接，并了解 Windows、Mac、Word Online、iPad 上导出 PDF 的方法。",
  alternates: {
    canonical: "https://kollab.im/zh/tools/word-to-pdf",
    languages: {
      "zh-CN": "https://kollab.im/zh/tools/word-to-pdf",
      en: "https://kollab.im/tools/word-to-pdf",
      ja: "https://kollab.im/ja/tools/word-to-pdf",
      ko: "https://kollab.im/ko/tools/word-to-pdf",
      "x-default": "https://kollab.im/tools/word-to-pdf",
    },
  },
  openGraph: {
    type: "website",
    siteName: "Kollab",
    locale: "zh_CN",
    title: "Word 转 PDF 在线工具：如何将 Word 保存为 PDF | Kollab",
    description:
      "上传 DOC 或 DOCX 文件，在线将 Word 保存为 PDF。保留排版、图片、表格和链接，也可查看不同设备上的保存方法。",
    url: "https://kollab.im/zh/tools/word-to-pdf",
    images: [
      {
        url: "https://kollab.im/images/tools/word-to-pdf-clay-hero-transparent.png",
        alt: "Word 文档转换成 PDF 文件的 3D 粘土风插画",
      },
    ],
  },
  twitter: {
    card: "summary_large_image",
    title: "Word 转 PDF 在线工具：如何将 Word 保存为 PDF | Kollab",
    description: "上传 Word 文档，在线保存为 PDF，并保留排版、图片、表格和链接。",
    images: [
      {
        url: "https://kollab.im/images/tools/word-to-pdf-clay-hero-transparent.png",
        alt: "Word 文档转换成 PDF 文件的 3D 粘土风插画",
      },
    ],
  },
};
```

## 推荐复用组件

请优先复用现有工具页的共享组件，而不是重新实现样式：

- `ToolPageLayout`：工具页外层布局、返回工具入口、SEO 容器。
- `ToolHero`：H1、副标题、主 CTA、辅助 CTA。
- `HeroVisual` 或现有图片组件：使用 `word-to-pdf-clay-hero-transparent.png` 作为透明背景 3D clay 主视觉。
- `FileUploadDropzone`：文件上传区域，允许 `.doc`、`.docx`、`.rtf`。
- `TaskPromptBox`：转换要求输入框。
- `PromptChipList`：格式保留、简历转 PDF、合同转 PDF、PDF/A、表格文档、批注与修订。
- `FeatureCardGrid`：价值说明三卡。
- `HowItWorksSteps`：四步转换流程。
- `UseCaseGrid`：使用场景。
- `RelatedTools`：相关工具。
- `FAQAccordion`：FAQ。
- `JsonLd`：`HowTo`、`FAQPage`、`BreadcrumbList`、`WebApplication`。

## 中文页面文案配置

```ts
export const wordToPdfZh = {
  slug: "word-to-pdf",
  locale: "zh-CN",
  backLabel: "返回工具",
  eyebrow: "DOC / DOCX 转 PDF",
  h1: "Word 转 PDF，保留格式在线保存",
  description:
    "上传 Word 文档，把 DOC 或 DOCX 保存为 PDF。适合简历、合同、报告、讲义和需要固定版式的文件。Kollab 会把文件、转换要求和输出结果放在同一个任务中，方便检查、修改和分享。",
  primaryCta: "上传 Word 文件",
  secondaryCta: "查看保存方法",
  heroImage: {
    src: "/images/tools/word-to-pdf-clay-hero-transparent.png",
    alt: "Word 文档转换成 PDF 文件的 3D 粘土风插画",
  },
  upload: {
    title: "上传你的 Word 文件",
    description: "支持 DOC、DOCX、RTF。拖放文件或点击选择。",
    accept: [".doc", ".docx", ".rtf"],
  },
  prompt: {
    label: "转换要求",
    defaultValue:
      "将 Word 文档转成 PDF。保留标题、页眉页脚、图片、表格、链接和分页。如果包含批注或修订，请生成干净版本，并说明是否需要 PDF/A、压缩或密码保护。",
  },
  chips: ["保留格式", "简历转 PDF", "合同转 PDF", "PDF/A", "表格文档", "批注与修订"],
  featureIntro: "普通转换器只给你一个下载按钮。Kollab 更适合需要检查、修改和交付的文档工作流。",
  features: [
    ["保留版式", "转换时关注字体、图片、表格、链接、页眉页脚和分页，减少发给别人后排版变化的问题。"],
    ["带着要求转换", "你可以说明是否隐藏批注、接受修订、生成 PDF/A、压缩文件或准备打印版本。"],
    ["结果可继续处理", "转换后的 PDF 可以继续进入 Kollab 的其他任务，例如压缩、摘要、提取 Markdown 文档或生成交付说明。"],
  ],
  stepsIntro:
    "如果你想知道 how to save a word document as a pdf，可以直接上传文件在线转换，也可以参考下面的手动保存方法。",
  steps: [
    ["上传文档", "选择 DOC、DOCX 或 RTF 文件。简历、合同、报告和表格文档都可以开始处理。"],
    ["说明要求", "告诉 Kollab 是否需要保留批注、接受修订、生成 PDF/A、压缩体积或保留链接。"],
    ["运行转换", "Kollab 会把文件、指令和输出放在同一任务中，便于回看上下文和继续修改。"],
    ["检查下载", "确认分页、图片、表格、页码和链接无误后，下载可分享、打印或归档的 PDF。"],
  ],
};
```

## 页面结构

1. `ToolHero` + `HeroVisual` + `FileUploadDropzone` + `TaskPromptBox`
2. 价值说明三卡
3. 在线转换四步流程
4. 手动保存方法模块，锚点 `#methods`
5. 转换前检查项
6. 使用场景
7. 相关工具
8. FAQ
9. 底部 CTA

## 结构化数据

- `WebApplication`：工具本身。
- `BreadcrumbList`：`首页 > 工具 > Word 转 PDF`。
- `HowTo`：如何将 Word 文档保存为 PDF。
- `FAQPage`：根据页面 FAQ 生成。

## 图片与 Alt 文本

- 文件：`/images/tools/word-to-pdf-clay-hero-transparent.png`
- 中文 alt：`Word 文档转换成 PDF 文件的 3D 粘土风插画`
- 英文 alt：`3D clay illustration of a Word document turning into a PDF file`
- 日文 alt：`Word 文書を PDF ファイルに変換する 3D クレイ風イラスト`
- 韩文 alt：`Word 문서를 PDF 파일로 변환하는 3D 클레이 스타일 일러스트`

## 相关工具

- PDF 转 Markdown：`/zh/tools/pdf-to-markdown`
- Markdown 转 PDF：`/zh/tools/markdown-to-pdf`
- PDF 压缩：`/zh/tools/compress-pdf`
