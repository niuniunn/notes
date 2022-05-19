## Tailwind CSS v3.+ 

#### 简介

`Tailwind CSS` 是一个功能类优先的 CSS 框架，它集成了诸如 `flex`, `pt-4`, `text-center` 和 `rotate-90` 这样的的类，它们能直接在脚本标记语言中组合起来，构建出任何设计。

```vue
<template>
    <div class="container">
        <img src="/images/logo-1.png" alt="">
        <div class="center">
            内容
        </div>
        <div>
            内容
        </div>
    </div>
</template>
<style scoped>
    .container {
        display: flex;
        justify-content: space-between;
        align-items: center;
    }
    img {
        display: inline-block;
        width: 118px;
    }
    .center {
        display: flex;
        justify-content: space-between;
        width: 50%;
    }
</style>
```

使用Tailwind css框架编写样式：

```vue
<template>
    <div class="flex justify-between items-center">
        <img class="inline-block w-[118px]" src="/images/logo-1.png" alt="">
        <div class="flex justify-between w-1/2">
            内容
        </div>
        <div>
            内容
        </div>
    </div>
</template>
```

- 首先看起来代码减少了很多，`Tailwind`提供了大量开箱即用的语义化原子类，我们无需再写css样式，而是直接引用预置的类名即可完成样式设计。
- 试过之后你会发现不用在`html`结构和`css`样式之间来回切换真的很爽，让你只专注于一个地方。
- 大家可能都经历过，有时候取名真的是一个老大难的问题，而现在，你不用再去考虑取名！
- 生产优化，`Tailwind`在构建生产环境时会自动移除未使用的CSS，以获得最佳性能。
- 还有更多的优点等大家发掘...



#### 安装

```
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

创建了`tailwind.config.js`和`postcss.config.js`

#### 在vue中配置

`tailwind.config.js`配置

```js
module.exports = {
  // after3.0 purge renamed to content
  content: [
    "./index.html",
    "./src/**/*.{vue,js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}

```

一种引入方式是在src目录下新建`index.css`，加入以下内容，导入`Tailwind`的基础、组件和功能样式，使用比较老版本的`webstorm` `@tailwind`可能会报红，在2020.3版本后加入了对`tailwind`的支持，`vscode`需要安装插件`Tailwind CSS IntelliSense`，在写class时会有相应的提示。

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

在src目录下的main.js文件中引入

```js
import { createApp } from 'vue';
import App from './App.vue';
import router from './router';
import ElementPlus from 'element-plus';
import './index.css';
import 'element-plus/dist/index.css'; // tailwind样式可能会覆盖element样式，要先引入tailwind后引入element的css

const app = createApp(App).use(router);
app.use(ElementPlus);

app.mount("#app");
```

如果是使用的vue或者react这样的框架，也可以直接导入css文件。

```js
import "tailwindcss/tailwind.css"
```

现在就启动项目来试试吧

```vue
<template>
  <h1 class="text-3xl font-bold underline">
    Hello world!
  </h1>
</template>
```

[官方其他项目配置方式](https://tailwindcss.com/docs/installation/framework-guides)

刚开始使用的时候需要借助官方文档和编辑器插件的提示，查看各个样式对应的类名，熟练之后就会感到如鱼得水。



#### 那这岂不是和内联样式差不多吗？相似但又不同。

使用功能类比内联样式具有一些重要的优点：

> - **基于约束的设计**. 使用内联样式, 每个值都是一个魔术数字。 使用功能类, 您是从预定义的[设计系统](https://www.tailwindcss.cn/docs/theme)中选择样式，这使得构建统一的UI变得更加容易。
> - **响应式的设计**. 在内联样式中您不能使用媒体查询, 但您可以使用 Tailwind 的[响应式功能类](https://www.tailwindcss.cn/docs/responsive-design)非常容易的构建完全响应式的界面。
> - **Hover, focus, 以及其它状态**. 内联样式无法设置 hover 或者 focus 这样的状态, 但 Tailwind 的[状态变体](https://www.tailwindcss.cn/docs/hover-focus-and-other-states)使用功能类可以非常容易的为这些状态设置样式。



#### 指令

##### `@tailwind`

> 使用 `@tailwind` 指令向您的 CSS 添加 Tailwind 的 `base`, `components`, `utilities` 和 `screens` 样式。

主要用法就是像我们前面在配置的时候在`index.css`中用于将`Tailwind`的一些基础功能添加到我们的项目中。

##### `@apply`

 **`@apply` 将任何现存的功能类内联到自定义 CSS 中。**

例如当我有好几个标签需要同样的样式我是否需要重复写class？

有相似的标签我们有时候可能是需要封装成一个组件，这样可以避免代码重复，但对于按钮和表单元素之类的小型组件，与简单的 CSS 类相比，创建模板片断或 JavaScript 组件通常会感觉过重。

可以使用Tailwind 提供的`@apply`功能创造这样的抽象类。

```vue
<div class="text-right text-2xl text-white">
	<el-icon class="arrow-btn mr-3">
		<arrow-left />
	</el-icon>
	<el-icon class="arrow-btn">
		<arrow-right />
	</el-icon>
</div>
```

```css
.arrow-btn {
	@apply p-1.5 bg-gray-800 box-content cursor-pointer;
}
```

##### ` @layer`

> 使用 `@layer` 指令告诉 Tailwind 一组自定义样式应该属于哪个 “bucket”。可用的层有 `base`, `components` 和 `utilities`。

##### `@variants`

> 通过在 `@variants` 指令中声明自己的功能类来生成他们的 `responsive`, `hover`, `focus`, `active` 及其它 [变体](https://www.tailwindcss.cn/docs/hover-focus-and-other-states)。

##### `@responsive`

> 通过在 `@responsive` 指令中声明他们的定义来生成您自己的类的响应式变体。

##### `@screen`

> `@screen` 指令允许创建通过名称引用断点的媒体查询，而不是在CSS 中复制他们的值。

相当于是一个媒体查询的简写方式

```css
@media (min-width: 640px) {
  /* ... */
}
```

```css
@screen sm {
  /* ... */
}
```



#### 如何使用自定义值？

在写一些例如字体大小、边距、背景图时，使用预设的尺寸不足以供我们使用。

Tailwind 支持以 `utility-[value]` 的形式来临时使用任意样式值，其中 `utility` 表示基础类，后面用 `-` 连字符相连的，在方括号 `[]` 中的 `value` 就是该样式的值。而且这种形式也支持使用状态变量，例如 `hover:utility-[value]` 设置在鼠标悬停时元素的样式。

```html
<div class="before:content-['丨'] before:text-[20px]">
	我是内容我是内容
</div>
```

#### 在tailwind中使用伪类

在`Tailwind`中使用`:before ` `:after`这样的伪类需要安装伪类插件`tailwindcss-pseudo-elements`。

```
// npm 安装
npm install tailwindcss-pseudo-elements --save-dev
// yarn 安装
yarn add tailwindcss-pseudo-elements -D
```

然后在`tailwind.config.js`中加入

```js
plugins: [
    require("tailwindcss-pseudo-elements"),
],
```



```html
<div class="before:content-['丨'] before:text-xl before:text-white">
	我是内容我是内容
</div>
```

#### 响应式

`	Tailwind`内置了5个断点，我们可以直接使用它来代替媒体查询

```html
<img class="w-16 md:w-32 lg:w-48" src="...">
```

 内置断点前缀代表的尺寸：

| 断点前缀 | 最小宽度 | CSS                                  |
| -------- | -------- | ------------------------------------ |
| `sm`     | 640px    | `@media (min-width: 640px) { ... }`  |
| `md`     | 768px    | `@media (min-width: 768px) { ... }`  |
| `lg`     | 1024px   | `@media (min-width: 1024px) { ... }` |
| `xl`     | 1280px   | `@media (min-width: 1280px) { ... }` |
| `2xl`    | 1536px   | `@media (min-width: 1536px) { ... }` |

当然我们也可以在`tailwind.config.js` 中自定义自己的尺寸：

```js
screens: {
    'tablet': '640px',
    // => @media (min-width: 640px) { ... }

    'laptop': '1024px',
    // => @media (min-width: 1024px) { ... }

     'desktop': '1280px',
    // => @media (min-width: 1280px) { ... }

    '2k': '1920px',
    // => @media (min-width: 1920px) { ... }
},
```

在class中这样去使用它

```html
<div class="min-h-screen 2k:w-4/5 desktop:w-3/4 laptop:w-4/5 tablet:w-full min-w-[760px] px-4 mx-auto text-white">content</div>
```



#### `@tailwindcss/line-clamp` 插件

多行文字缺省显示通常需要好几行CSS

```css
overflow: hidden;
text-overflow: ellipsis;
display: -webkit-box;
-webkit-line-clamp: 5; /* 显示几行后添加省略号 */
-webkit-box-orient: vertical;
```

使用`scss`的话可以封装这样一个方法

```scss
@mixin ellipsis($line: 1, $substract: 0) {
    @if $line==1 {
        white-space: nowrap;
        text-overflow: ellipsis;
    } @else {
        display: -webkit-box;
        -webkit-line-clamp: $line;
        -webkit-box-orient: vertical;
    }
    width: 100% - $substract;
    overflow: hidden;
}
```

但是——`Tailwind`只需要加一个类名！

```html
<div class="line-clamp-2">相信一个崭新的源头永远始于改变，我们</div>
```

安装插件命令

```
npm install -D @tailwindcss/line-clamp
```

然后在`tailwind.config.js`中加入

```js
plugins: [
    require("@tailwindcss/line-clamp"),
],
```



#### 一些弊端

- 刚开始使用的时候需要去查文档，查看它预置的类名，而随着版本的更新，这些单词可能会发生变化，你得更新你的词库。
- 可读性较差，一个标签上可能加了一长串的类名，也没有换行，看起来很费力，当然可以使用@apply放到一起，会多次使用的那种公共的还好，仅仅是因为类名太长而提出来岂不是又有点回到写普通CSS的时候了吗。
- ...

#### 参考资料

[官方文档](https://www.tailwindcss.cn/docs)

[Tailwind CSS v3——核心思想（三）自定义样式](https://juejin.cn/post/7072173663851642893)



