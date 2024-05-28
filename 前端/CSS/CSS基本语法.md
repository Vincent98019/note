
**CSS（cascading style sheet，层叠式样式表）** 是用来给HTML标签添加样式的语言，CSS3是CSS的最新版本，增加了大量的样式、动画、3D特效和移动端特性等。
CSS使样式和结构分离，样式和结构不用“杂糅着写”，而是彼此分开：HTML就负责结构，CSS负责样式。
CSS就是样式的“清单”，要书写合适的选择器，然后把指定元素的样式“一条一条罗列”出来。
CSS没有加减乘除、与或非、循环、选择、判断，CSS不是 “编程” ，就是简单直接的罗列样式。

## CSS的位置

### 内嵌式

在`<head></head>`标签对中，书写`<style></style>`标签对，里面书写CSS语句

### 外链式

可以将CSS单独存为.css文件，然后使用<link>标签引入它，多个html网页，可以共用一个css样式表文件

`<link rel="stylesheet" href="css/css.css">`

### 导入式

导入式是最不常见的样式表导入方法，使用导入式引入的样式表，不会等待css文件加载完毕，而是会立即渲染HTML结构，所以页面会有几秒中的“素面朝天”的时间

### 行内式

样式可以直接通过style属性写在标签身上，行内式牺牲了样式表的批量设置样式的能力，只能给一个标签设置样式，所以不常用。

`<h2 style="color: red;">我是一个二级标题</h2>`

## 基本语法

最后一条样式可以不书写分号

`选择器 {key: value; key...}`

```css
h1 {
    color: green;
    font-weight: bold;
    font-style: italic;
}
p {
    font-size: 20px;
    background-color: orange
}
```

**注释：**

```css
/* */
```
