

本博客在[Vno Jekyll](https://github.com/onevcat/vno-jekyll)基础上修改的。  

### Jekyll vs Hugo

| **Feature**               | **Jekyll**                          | **Hugo**                          |
|---------------------------|-------------------------------------|-----------------------------------|
| **First Release**         | 2008                                | 2013                              |
| **Built With**            | Ruby                                | Go                                |
| **Templating Engine**     | Liquid                              | Go Templates                      |
| **Plugins**               | Yes (Ruby Gems)                     | No (Built-in features instead)    |
| **GitHub Stars**          | 42K+                                | 49K+                              |
| **Themes**                | Yes                                 | Yes                               |
| **License**               | MIT                                 | Apache 2.0                        |
| **Installation**          | Requires Ruby                       | Single binary (No dependencies)   |
| **Asset Pipeline**        | SASS/CoffeeScript + plugins         | SASS, JS, bundling (built-in)     |
| **Build Speed**           | Fast-ish (improved in v4.0)         | Fastest in class                  |
| **Multilingual/i18n**     | Via plugins                         | Built-in                          |
| **Shortcodes**            | Yes                                 | Yes                               |
| **WordPress Migration**   | Yes (`jekyll-import`)               | Yes (Hugo converters)             |
| **Learning Curve**        | Gentle (Ruby familiarity helps)     | Steeper (Go templating quirks)    |
| **Content Types**         | Markdown, HTML (+ plugins)          | Markdown, AsciiDoc, RST, HTML     |
| **Community**             | [Jekyll Talk](https://talk.jekyllrb.com/), Gitter | [Hugo Discourse](https://discourse.gohugo.io/) |
| **CLI Tool**              | Yes                                 | Yes                               |
| **Hot Reloading**         | Yes                                 | Yes                               |
| **Twitter**               | [@jekyllrb](https://twitter.com/jekyllrb) | [@GoHugoIO](https://twitter.com/GoHugoIO) |
| **GitHub Repo**           | [jekyll/jekyll](https://github.com/jekyll/jekyll) | [gohugoio/hugo](https://github.com/gohugoio/hugo) |


## Markdown 基本语法 / Basic Markdown Syntax

## 标题 / Headers
# H1 / `# Header 1`  
## H2 / `## Header 2`  
### H3 / `### Header 3`  
#### H4 / `#### Header 4`  
##### H5 / `##### Header 5`  
###### H6 / `###### Header 6`  

## 文本样式 / Text Styles
**加粗 / Bold**: `**Bold**`  
*斜体 / Italics*: `*Italics*`  
~~删除线 / Strikethrough~~: `~~text~~`  

## 链接与图片 / Links & Images
[链接 / Link](URL): `[Title](URL)`  
![图片 / Image](image-url): `![Alt text](image-url)`

## 段落与换行 / Paragraphs & Line Breaks
段落 / Paragraph: 段落之间空一行  
换行 / Line break: 行尾加两个空格  

## 列表 / Lists
- 无序列表 / Unordered list: `* Item` 或 `- Item`  
1. 有序列表 / Ordered list: `1. Item`  

## 引用 / Blockquotes
> 引用内容 / Blockquote: `> Quote text`

## 代码 / Code
内嵌代码 / Inline code: `` `code` ``  
代码块 / Code block:  
  ```language
code
```
 
## 水平线 / Horizontal Rule
---: `---` 或 `***`

## 表格 / Tables
| Syntax      | Description |  
|-------------|-------------|  
| Header      | Title       |  
| Paragraph   | Text        |  
        
