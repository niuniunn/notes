# CSS小笔记

#### 深入理解content

content开启闭合符号生成

```css
.content h1 {
   quotes: '-' '-';
}

.content h1:before {
   content: open-quote;
}

.content h1:after {
   content: close-quote;
}
```

content计数器

```CSS
.counter {
   counter-reset: mycounter 0; /* 计数器名称 从多少开始计数 */
}

.counter div:before {
   content: counter(mycounter, lower-roman); /* 计数器名称 递增递减样式 可以是罗马字符、拉丁文等等 */
   counter-increment: mycounter 2; /* 计数器名称 每次递增多少 */
}
```

counters用法见

https://demo.cssworld.cn/4/1-18.php

#### margin

在实现内容的左对齐或右对齐时，可以使用margin来实现，少使用float

```css
.to-right {
	margin-left: auto;
}
```

实现水平垂直居中

```css
.father {
	position: relative;
	width: 300px;
	height: 300px;
}
.son {
   position: absolute;
   width: 100px;
   height: 100px;
   left: 0;
   right: 0;
   top: 0;
   bottom: 0;
   margin: auto;
}
```

单行文字省略显示不生效：

```css
overflow: hidden;
text-overflow: ellipsis;
white-space: nowrap;
```

**块级元素**才有效果

