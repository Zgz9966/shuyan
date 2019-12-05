# Markdown 编写规范

采用 [CTF Wiki Team](https://github.com/ctf-wiki/) 的 Markdown 编写规范。

## 文档格式

- 使用 `.md` 后缀
- 文件夹、文件名使用小写，单词之间使用连字符 `-` 分隔

  建议使用连字符而非下划线的原因是，搜索引擎会将连字符处理为两个单词，而将下划线看做一个单词。参考 [Google SEO 指南](https://support.google.com/webmasters/answer/76329?hl=zh-Hans)。

- 文档编码使用 UTF-8

## 标题

- 根据 MkDocs 的要求，文档中至多有一个一级标题 `#`，当没有一级标题时采用目录配置中的标题作为当前页面的一级标题

- 章节标题从 `##` 开始

- 章节标题在 `##` 后添加空格，且之后没有 `##`

  ```markdown
  // bad
  ##章节1

  // bad
  ## 章节1 ##

  // good
  ## 章节1
  ```

- 章节标题与正文间有且仅有一个空行

## 段落

- 使用空行换行，尽量不适用两空格换行

  部分 IDE 会在提交时自动清理行末的空格

- 一个段落只表达一个主题

- 使用主动语态

- 陈述句中使用肯定说法

- 删除不必要的词

- 避免啰嗦、口语化的语句

- 需要强调的内容酌情使用 [Admonition 插件](https://github.com/ctf-wiki/ctf-wiki/wiki/Material-%E4%B8%BB%E9%A2%98%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E#admonition)，有 `note`、`abstract`、`info`、`tip`、`success`、`question`、`warning`、`failure`、`danger`、`bug`、`example`、`quote` 多种样式

## 列表

### 有序列表无需编码

```markdown
1.  Foo.
1.  Bar.
    1.  Foofoo.
    1.  Barbar.
1.  Baz.
```

### 嵌套列表

在有序和无需嵌套列表时使用 4 空格缩进。

```markdown
1.  2 spaces after a numbered list.
    4 space indent for wrapped text.
2.  2 spaces again.

*   3 spaces after a bullet.
    4 space indent for wrapped text.
    1.  2 spaces after a numbered list.
        8 space indent for the wrapped text of a nested list.
    2.  Looks nice, don't it?
*   3 spaces after a bullet.
```

当没有嵌套时，也尽量使用 4 空格缩进。

```markdown
*   Foo,
    wrapped.

1.  2 spaces
    and 4 space indenting.
2.  2 spaces again.
```

当列表结构很简单时，可以在标识符后使用 1 个空格作为标记。

```markdown
* Foo
* Bar
* Baz.

1. Foo.
2. Bar.
```

## 代码

### 代码块

- 代码块使用 Fenced Block

  ````markdown
  ​```js
  console.log("");
  ​```
  ````

- 代码块注明语言，以便代码高亮，参见 Pygments 文档


### 行内代码

- 行内代码使用反引号，且当引用文件时使用行内代码

  ```markdown
  Be sure to update your `README.md`!
  ```



## 表格

以 [GitHub Flavored Markdown](https://help.github.com/articles/organizing-information-with-tables/) 格式为准。

```markdown
| First Header  | Second Header |
| ------------- | ------------- |
| Content Cell  | Content Cell  |
| Content Cell  | Content Cell  |

| Left-aligned | Center-aligned | Right-aligned |
| :---         |     :---:      |          ---: |
| git status   | git status     | git status    |
| git diff     | git diff       | git diff      |
```

## 排版

### 空格

- 中英文之间需要增加空格（包括行内代码）
- 中文与数字之间需要增加空格
- 数字与单位之间需要增加空格
- 全角标点与其他字符之间不加空格

### 标点符号

- 不重复使用标点符号
- 用直角引号 `「」` 代替双引号 `“”`
- 使用规范的省略号 `……`

### 全角与半角

- 使用全角中文标点
- 数字使用半角字符
- 遇到完整的英文整句、特殊名词，其內容使用半角标点

### 名词

- 专有名词使用正确的大小写

### 公式

合理使用行内公式和行间公式，前后无文字的尽量使用行间公式，可以居中显示，提升阅读体验。

```markdown
行内公式 $ a + b = c $

行间公式
$$
a + b = c
$$
```

### 其他

更多排版相关内容和例子请参考 [中文排版指南](https://github.com/ctf-wiki/ctf-wiki/wiki/%E4%B8%AD%E6%96%87%E6%8E%92%E7%89%88%E6%8C%87%E5%8D%97)。