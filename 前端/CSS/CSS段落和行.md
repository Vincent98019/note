
## text-indent属性
`text-indent`属性定义首行文本内容之前的缩进量，缩进两个字符应该写作 `text-indent: 2em;` 
em表示字符宽度


## line-height
`line-height`属性用于定义行高，单位可以是以`px`为单位的数值，也可以是没有单位的数值，表示字号的倍数，这是最推荐的写法，也可以是百分数，表示字号的倍数

```css
line-height: 30px;
line-height: 1.5;
line-height: 150%;
```


## 单行文本垂直居中
设置行高=盒子高度，即可实现单行文本垂直居中，设置`text-align: center`，即可实现文本水平居中


## font合写属性
`font`属性可以用来作为`font-style`，`font-weight`，`font-size`，`line-height`和`font-family`属性的合写

```css
font: 20px/1.5 Arial, "微软雅黑";
font: italic bold 20px/1.5 Arial, "微软雅黑";
```


## 继承性
文本相关的属性普遍具有继承性，只需要给祖先标签设置，即可在后代所有标签中生效

color
font- 开头的
list- 开头的
text- 开头的
line- 开头的

因为文字相关属性有继承性，所以通常会设置body标签的字号、颜色、行高等，这样就能当做整个网页的默认样式了

**就近原则：**
在继承的情况下，选择器权重计算失效，而是“就近原则”

```html
<div id="box1" class="box1">
    <div id="box2" class="box2">
        <div id="box3" class="box3">
            <p>我是文字</p>
        </div>
    </div>
</div>

#box1 #box2 #box3 { <!-- 继承 -->
    color: red;
}
p { <!-- 选中了 -->
    color: green;
}
```

```html
<div id="box1" class="box1">
    <div id="box2" class="box2">
        <div id="box3" class="box3">
            <p>我是文字</p>
        </div>
    </div>
</div>

#box1 #box2 {   <!-- 远 -->
    color: red;
}
.box1 .box3 {   <!-- 近 -->
    color: blue;
}
```
