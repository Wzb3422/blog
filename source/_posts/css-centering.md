---
title: 🀄️CSS 居中大集合
keywords: CSS Centering
tags: CSS Centering
categories:
  - CSS
date: 2019-06-06 09:37:56
thumbnail: https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/1607355807299.png
---

「居中」是进行布局时最常见的需求之一了。CSS 有多种居中的方式，总结一下。

<!-- MORE -->

# 法律声明

**警告**：本作品遵循 **署名-非商业性使用-禁止演绎 4.0 未本地化版本（CC BY-NC-ND 4.0）** 协议发布。你应该明白与本文有关的一切行为都应该遵循此协议。

> [这是什么？](https://creativecommons.org/licenses/by-nc-nd/4.0/)


## 写在前面

本文将会按照具体场景来选择相应的居中方式，帮助你系统地理清 CSS 居中。

注意：为了简洁，文中给出的 CSS 代码只会给出关键的定义布局的代码。

## 在水平方向上的居中（Horizontally Centering）

### 对于行内（inline / inline-* ）元素

要将行内元素居中，只需要给其父块级元素（block-level parent element）定义以下 CSS 规则：

```css
.block-level-parent-of-inline-element {
  text-align: center;
}
```



> 这对 `inline` `inline-block` `inline-table` `inline-flex` etc. 都生效


`text-align` 不仅仅是针对于 `text` 的对齐描述，实际上，它影响的是块级元素下的行内元素与文本（inline contents）。

> 参考：[text-align - CSS | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/text-align)


### 对于块级元素（block element）

要将块级元素居中，给其**定宽（width）之后**定义以下 CSS 规则

```css
.block-element {
  margin: 0 auto;
}
```

![](https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20201207235124.png)

如果没有给定块级元素宽度？那它会充满整行，以至于不需要居中了... 🔨

就像这样

![](https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20201207235200.png)

### 如果有多个块级元素？

如果你想将多个块级元素在水平方向上居中，有以下两种方法。

- 将多个块级元素设置 `display: inline-box;` 然后给父级元素定义 `text-align: center;`（与上文的行内元素居中同理）

```css
.parent {
  text-align: center;
}
.block-elements {
  display: inline-block;
}
```

- 使用 Flexbox

```css
.parent {
  display: flex;
  justify-content: center;
}
```

## 在垂直方向上的居中（Vertically Centering）

### 对于行内（inline / inline-* ）元素

比如文字（text）和链接（links）

#### 只有一行时

- 要让行内元素/文字看起来在竖直方向上的中间，可以给其定义上下相等的内边距（padding）

```css
a {
  padding-top: 20px;
  padding-bottom: 20px;
}
```

- 还有一种方案是，将文字的行高值设置与其元素高度相同。

```css
.box {
  height: 60px;
  line-height: 60px;
}
```
仔细思考，如果文字的内容超过一行，或者是父级元素的大小在响应式变化的情况下变小导致文字需要换行，会出现这样的情况。

解决这个问题，只需要设置文字不换行即可

```css
.box {
  height: 60px;
  line-height: 60px;
  white-space: nowarp;
}
```

#### 有多行文字时

- 同样的，可以给元素设置上下相等的 `padding`，让文字看起来在中间
- 将父元素定义为 `display: table;` 同时将子元素设置为 `display: table-cell;` 并且给予 `vertical-align: middle;` 属性。

```css
.box {
  display: table;
  height: 60px;
}

.box p {
  display: table-cell;
  vertical-align: middle;
}
```

- 使用 Flexbox 居中

将文字放在块级元素中，给其父元素定义 `display: flex; align-items: center;`

```css
.box {
  display: flex;
  flex-direction: column;
  justify-content: center;
}
```

**以上效果相同，如下图。**

![](https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/20201208155716.png)

### 对于块级（block）元素

#### 已经确定块级元素的高度

在实际情况下，很少会明确地确定某一个元素的高度。如果真的能够确定某个元素的高度，可以使用下面的方法。

```css
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 50%;
  height: 100px;
  margin-top: -50px;
}
```

这里需要注意的是 `top` 属性的计算方法，可以参考 [CSS - top | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/top)。

#### 不能确定块级元素的高度

我们仍可以使用与上例类似的方法，不过需要使用 `transform` 属性。

```css
.parent {
  position: relative;
}
.box {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
}
```

> 对于 translate 的计算方式，可以参考 [Is the CSS3 transform translate percentage values relative to its width and/or height? - Stack Overflow](https://stackoverflow.com/questions/34551381/is-the-css3-transform-translate-percentage-values-relative-to-its-width-and-or-h)


#### 如果你不在乎元素撑大的话

你不在乎元素是否撑开，只想要它的内容竖直居中，可以使用 table 或者是 CSS display 将其变成 table 行为。

```css
main {
  padding: 5px;
  display: table;
}
main div {
  display: table-cell;
  vertical-align: middle;
}
```

可见，元素被撑大了。🔨

##### Flexbox

使用 Flexbox 在竖直方向上居中是同样的方便。

```css
.parent {
  display: flex;
  flex-direction: column;
  justify-content: center;
}
```



## 水平和竖直方向上同时居中

你可以将上面的方法结合使用达到此目的。不过这里总结了几种比较方便的方式。

### 元素确定了高度和宽度时

使用绝对定位之后，设置为负的margin属性，其值等于 高度/宽度 的50%。

```css
.parent {
  position: relative;
}

.child {
    height: 60px;
    width: 180px;

    position: absolute;
    top: 50%;
    left: 50%;

    margin: -30px 0 0 -90px;
}
```

这样进行居中的浏览器兼容性非常好。
### 不确定元素高宽

与上例相同，使用绝对定位，不过换成了 translate 属性。

```css
.parent {
  position: relative;
}

.child {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
```



### Flexbox

相信你已经想到了 Flexbox 如何实现，只需要把主轴和侧轴方向上都设置为居中就好了。

```css
.parent {
  display: flex;
  justify-content: center;
  align-items: center;
}
```



### Gird

对于居中一个元素来说，可以使用 Gird 的 trick。

```css
.parent {
  display: grid;
}
.parent div {
  margin: auto;
}
```



## 写在最后

现在你对居中的方法与选择有了清楚的认识，你再也不怕居中了。XD

![](https://wzb-img-base.oss-cn-shanghai.aliyuncs.com/img/1607416848836.png)

## References

- [Centering in CSS - CSS Tricks](https://css-tricks.com/centering-css-complete-guide/)
- [MDN](https://developer.mozilla.org/en-US/)
- [Flex 布局教程 - 阮一峰的博客](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)
- [CSS Transforms Module Level 1 - W3C Candidate Recommendation](https://www.w3.org/TR/css-transforms-1/)
