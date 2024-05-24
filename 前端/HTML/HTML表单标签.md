
所有HTML表单都以一个`<form>`元素开始

- `action`属性表示表单要提交到的后台程序的网址
- `method`属性表示表单提交的方式，有`get`或`post`

```html
<form action="save.php" method="post">
</form>
```

## 单行文本框

使用`type`属性值被设置为`text`的`<input>`元素可以创建单行文本框，它是一个单标签

 - `value`属性表示已经填写好的值
 - `placeholder`属性表示提示文本，将以浅色文字写在文本框中，并不是文本框中的值
 - `disabled`属性表示用户不能与元素交互，即“锁死”

```html
<input type="text" value="123" placeholder="请输入姓名">
<input type="text" disabled>
```

## 单选按钮

使用`type`属性值被设置为`radio`的`<input>`元素可以创建单选按。

- 互斥的单选按钮应该设置它们的`name`为相同值
- 单选按钮要有`value`属性值，向服务器提交的就是value值
- 单选按钮如果加上了`checked`属性，表示默认被选中

```html
<input type="radio">
```

## label标签

`label`标签用来将文字和单选按钮进行绑定，用户单击文字的时候也视为点击了单选按钮

```html
<label>
    <input type="radio"> 男
</label>
<label>
    <input type="radio"> 女
</label>
```

在HTML4时代，`label`标签是通过`for`属性和单选按钮的id属性进行绑定的

```html
<input type="radio" id="nan"><label for="nan">男</label>
```

## 复选框

使用`type`属性值被设置为`checkbox`的`<input>`元素可以创建复选框

- 同组复选框应该设置它们的`name`为相同值
- 复选框要有`value`属性值，向服务器提交的就是value值

```html
<input type="checkbox">
```

## 密码框

使用`type`属性值被设置为`password`的`<input>`元素可以创建密码框

```html
<input type="password">
```

## 下拉菜单 `<select>`

`<select>`标签表示下拉菜单，`<option>`是它内部的选项

```html
<select>
    <option value="alipay">支付宝</option>
    <option value="wx">微信</option>
    <option value="bank">网银</option>
</select>
```

## 多行文本框 `<textarea>`

`<textarea></textarea>`表示多行文本框

- `rows`和`cols`属性，用于定义多行文本框的行数和列数

```html
<textarea cols="50" rows="30"></textarea>
```

## 按钮
表单中常见三种按钮，它们也都是`input`标签，`type`属性值不同

| type属性值 | 描述 |
|:--------|:-------------|
| button | 普通按钮，可以简写为`<button></button>` |
| submit | 提交按钮 |
| reset | 重置按钮 |

```html
<input type="butten" value="按钮">
<button>按钮</button>
<input type="submit">
<input type="reset">
```


## HTML5新增表单控件

| type属性值 | 描述 |
|:--------|:-------------|
| color | 颜色选择控件 |
| date、time | 日期、时间选择控件 |
| email | 电子邮件输入控件 |
| file | 文件选择控件 |
| number | 数字输入控件 |
| range | 拖拽条 |
| search | 搜索框 |
| url | 网址输入控件 |


## 备选项 `<datalist>`

`<datalist>`控件可以为输入框提供一些备选项，当用户输入的内容与备选项文字相同时，将会显示智能感应

```html
<input type="text" list="province-list">
<datalist id="province-list">
    <option value="山东">
    <option value="山西">
    <option value="广东">
    <option value="广西">
    <option value="河南">
    <option value="河北">
    <option value="湖南">
    <option value="湖北">
</datalist>
```


## 必填

使用`required`则表示必填

```html
<input type="text" required> 
```

