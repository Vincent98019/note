

## 表格

- 为了让表格能够显示边框，`<table>`标签通常有`border`属性
- `<caption>`是表格的标题， 它常常作为 `<table>` 的第一个子元素出现
- `<tr>`代表一行
- `<td>`代表一行中的一个单元格

```html
<table border="1">
    <caption>这里是标题</caption>
    <tr>
        <td>第一列</td>
        <td>第二列</td>
        <td>第三列</td>
    </tr>
    <tr>
        <td>第二行第一列</td>
        <td>第二行第二列</td>
        <td>第二行第三列</td>
    </tr>
</table>
```

## `<th>` 标签

`<th>` 是“标题小格”，可以替代`<td>`的作用，表示标题小格

```html
<table border="1">
    <tr>
        <th>第一列</th>
        <th>第二列</th>
        <th>第三列</th>
    </tr>
    <tr>
        <td>第二行第一列</td>
        <td>第二行第二列</td>
        <td>第二行第三列</td>
    </tr>
</table>
```

## colspan属性(横合并)

`colspan`属性用来设置`td`或者`th`的列跨度

```html
<table border="1">
    <tr>
        <td colspan="2">A</td>
        <td>B</td>
        <td>C</td>
    </tr>
    <tr>
        <td>D</td>
        <td colspan="3">E</td>
    </tr>
    <tr>
        <td>F</td>
        <td>G</td>
        <td>H</td>
        <td>I</td>
    </tr>
</table>
```

## rowspan属性(纵合并)

`rowspan`属性用来设置`td`或者`th`的行跨度

```html
<table border="1">
    <tr>
        <td>A</td>
        <td>B</td>
        <td>C</td>
        <td>D</td>
    </tr>
    <tr>
        <td>E</td>
        <td rowspan="2">F</td>
        <td>G</td>
        <td rowspan="3">H</td>
    </tr>
    <tr>
        <td>I</td>
        <td>J</td>
    </tr>
    <tr>
        <td>K</td>
        <td>L</td>
        <td>M</td>
    </tr>
</table>
```

## `<thead>`、`<tbody>`、`<tfoot>`标签

- `<thead>`标签定义表头
- `<tbody>`标签定义表核心内容
- `<tfoot>`标签定义表脚，通常是汇总行

```html
<table border="1" width="400">
    <thead>
        <tr>
            <th>A</th>
            <th>B</th>
            <th>C</th>
            <th>D</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>E</td>
            <td>F</td>
            <td>G</td>
            <td>H</td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <td>I</td>
            <td>J</td>
            <td>K</td>
            <td>L</td>
        </tr>
    </tfoot>
</table>
```


## `cellspacing`、`cellpadding`属性

`cellpadding` 属性定义了表格单元的内容和边框之间的空间，已经废弃，使用CSS替代它
`cellspacing` 属性（使用百分比或像素）定义了两个单元格之间空间的大小，已经废弃，使用CSS替代它

