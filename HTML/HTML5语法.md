---
tags:
- HTML
---

## HTML语法规则
HTML 标签是由尖括号包围的关键词，例如`<html>`。

- **围堵标签（双标签）：** 有开始标签和结束标签。如 `<html> </html>`
- **自闭合标签（单标签）：** 开始标签和结束标签在一起。如 `<br/>`

## 标签的关系

双标签关系可以分为两类：包含关系和并列关系

**包含关系：**

```html
<head>  
	<title> </title> 
</head>
```

**并列关系：**

```html
<head> </head>
<body> </body>
```

需要正确嵌套，不能你中有我，我中有你

- 错误：`<a><b></a></b>`
- 正确：`<a><b></b></a>`

在开始标签中可以定义属性。
属性是由键值对构成，值需要用引号(单双都可)引起来。

html的标签不区分大小写，但是建议使用小写。

文字和文字之间的多个空格、换行会被折叠成一个空格，标签“内壁”和文字之间的空格会被忽略。

**常用转义字符：**

| 转义字符 | 描述 |
|:--------| :-------------|
| `&lt;` | 小于号 |
| `&gt;` | 大于号 |
| `&nbsp;` | 空格（不会被折叠） |
| `&copy;` | 版权符号© |

## HTML注释
为代码书写清晰的注释是非常重要的，可以使日后再阅读代码或者他人阅读代码提供提示
HTML的注释语法如下，可以在VScode编辑器中使用`ctrl+/键`输入

```html
<!-- -->
```

## 文档类型声明DTD
HTML文件第一行必须是DTD（Document Type Definition，文档类型声明），不写DTD会引发浏览器的一些兼容问题，不同版本的HTML有不同的DTD写法。

```html
HTML5：
<!DOCTYPE html>

HTML4.01严格版：
<!DOCTYPE HTML PUBLIC "-
//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">

HTML4.01过渡版：
<!DOCTYPE HTML PUBLIC "-
//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">

HTML4.01框架版：
<!DOCTYPE HTML PUBLIC "-
//W3C//DTD HTML 4.01 Frameset//EN" "http://www.w3.org/TR/html4/frameset.dtd">
```

