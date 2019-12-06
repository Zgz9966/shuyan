## why git cz

给 `git commit` 添加一段简短有意义且规范的描述

## 组成

一个标准的 commit 应该包括下面几个部分

```xml
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

其中

### type

- feat 新功能
- fix 修复 Bug
- docs 只有文档改变
- style 并没有影响代码的意义(去掉空格，换行)
- refactor 没有修改 Bug 也没有提交新功能
- perf 代码修改提高性能
- test 添加测试
- chore 构建过程或者构建工具的改变

### scope

说明本次代码影响的范围（文件、文件夹）

### subject

简短描述

### body

当代码需要一些说明时

### foot

可以用来跟踪`issue`的`ID`，如`Close #123`

## 方便的库

- `npm i -g commitizen` 全局安装 commitizen

- `commitizen init cz-conventional-changelog --save --save-exact` 项目目录中运行

- 在`package.json`中添加

    ```json
    "config": {
        "commitizen": {
            "path": "cz-conventional-changelog"
        }
    }
    "scripts": {
        "commit": "git-cz"
    }
    ```

- 然后就可以通过 `npm run commit` 来运行了。

  

