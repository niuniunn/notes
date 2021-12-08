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

#### Vue

Vue.$nextTick()在DOM更新后才会执行，DOM更新是异步的

异步执行的运行机制：

（1）所有同步任务都在主线程上执行，形成一个执行栈（execution context stack）。
（2）主线程之外，还存在一个"任务队列"（task queue）。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件。
（3）一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。
（4）主线程不断重复上面的第三步

