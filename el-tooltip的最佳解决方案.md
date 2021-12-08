# el-tooltip气泡内容过长最佳解决方案

在做项目的时候，有时候表格内容非常长，如果不做处理就不太美观，这好办，直接给表格colunm加一个`show-overflow-tooltip`完事！表格就只会显示一行文字，显示不下会显示省略号然后鼠标放上去会有气泡显示全部的内容。



然后，就在某一天准备回归测试完打tag下班的时候，测试妹妹突然喊过去看看气泡，鼠标放上去一直在闪。啊，为什么这个表格的内容会有这么长啊！因为气泡样式是固定了宽度的，内容已经溢出了屏幕，没错，不只是溢出了气泡，页面出现了滚动条，没事，这事儿我熟，加个最大高度和`overflow`就行。

```css
.el-tooltip__popper {
   max-width: 180px;
   max-height: 94px;
   overflow: hidden;
}
```

![](pictures\tt_overflow.png)

好嘞，结果产品说后面要省略号，提示用户没有显示完全。

#### 单行文字缺省显示

平时用的比较多的是单行文字溢出省略显示：

```css
white-space: nowrap; /* 不换行*/
overflow: hidden; /* 溢出隐藏 */
text-overflow: ellipsis; /* 溢出部分省略显示 */
```

#### 多行文字缺省显示

多行也简单，使用[-webkit-line-clamp](https://developer.mozilla.org/zh-CN/docs/Web/CSS/-webkit-line-clamp)：

```css
overflow: hidden;
text-overflow: ellipsis;
display: -webkit-box;
-webkit-line-clamp: 5; /* 显示几行后添加省略号 */
-webkit-box-orient: vertical;
```

![](pictures\tt_sl.png)

查了一下浏览器兼容性新版本只有ie不支持，还是能接受的

<img src="pictures\tt_line_clamp.png"  />

行了，不用加班了。正当我开心劲儿还没过去的时候，没错，是不是你也发现了，我的三角形呢！



接着我就去搜了伪类因为父元素设置了`overflow: hidden`被隐藏了怎么解决的，说是再包一层`div`，给内层需要设置`overflow: hidden`的元素设置，不要覆盖到`:after`，可是这是人家的组件，我没法改呀，此路不通。



因为是用`:after`伪类生成并作为气泡的子元素，被当做溢出部分被隐藏了，接着我就去查怎么把被父元素`hidden`的子元素显示出来，诶，还真有，只要让被隐藏的元素不要相对父元素进行定位就可以了，大概有两种方式。



第一种方法是给子元素设置`position: absolute`，第二种是给子元素设置`position: fixed`，显然第二种不行，可以试试第一种，但是我发现他本身就是绝对定位啊，咋没有效果呢，然后写了demo试试，没问题啊，原来是因为气泡本身就有了定位，所以还是逃脱不了相对父元素定位，此路不通。



CSS不行能不能用js实现呢，刚好就截取五行的文字，但是我发现英文和逗号的宽度不一样，这就不好算呀，可以渲染到dom中然后去获取，但是也没法知道我想要的长度是截到哪个字，而且性能不行，兜兜转转又回到了css。



然后瞎调试半天，发现可以把三角形往上移动，但是跟背景融为一体了，能不能把三角形所在的这几行像素调成透明的呢，我想起了渐变可以画条纹！用渐变画一个背景图，上部分是正常气泡的颜色，下部分也就是三角部分设置成透明，虽然看起来有点骚操作，但是也没有别的办法，试试吧。

```css
.el-tooltip__popper {
   background: linear-gradient(#303133 97%, transparent 3%) !important;
}

.el-tooltip__popper[x-placement^=top] .popper__arrow,
.el-tooltip__popper[x-placement^=top] .popper__arrow:after {
		bottom: 0 !important; /* 把三角形往上移动 */
}
```

红色框起来那一块是气泡的整个背景上面是正常的颜色，下面一点是透明的，尖尖出来了，但是这也太丑了吧，不光会影响到别的地方没有溢出的小气泡，而且左下角和右下角的圆角都没有了，此路不通。

![](pictures\tt_liner.png)



那么除了使用`overflow`属性来隐藏还有没有别的方法呢？

```css
// 直接隐藏
visibility:hidden
display:none
// 对溢出内容隐藏
overflow:hidden
text-overflow:ellipsis
// 对元素透明度进行调整
opacity:0
background:transparent
// 将元素移除当前屏幕
position:absolute/relative
margin:-1000px
transform:translate(-9999px)
text-indent:-9999px
// 对元素的层级关系进行调整
z-index:-1000
// 对元素进行裁剪
clip(clip-path):rect()/inset()/polygon()
```

综合考虑，决定去看看`clip-path`是个怎么回事。

#### [clip-path](https://developer.mozilla.org/zh-CN/docs/Web/CSS/clip-path)

`clip-path` CSS 属性使用裁剪方式创建元素的可显示区域。区域内的部分显示，区域外的隐藏。

于是我想着剪裁一个气泡的形状，好像可行，但是以前没有使用过这个属性，要让各种大小的气泡都兼容才行，所以只能只用百分比，那中间那个三角形要怎么剪裁呢，如果能使用`calc`就好了，好像还真行！如果内容已经长得溢出了屏幕，长得页面出现了滚动条，可以给body加`overflow: hidden`

```css
.el-tooltip__popper {
   max-width: 180px;
   max-height: 94px;
   text-overflow: ellipsis;
   display: -webkit-box;
   -webkit-line-clamp: 5;
   -webkit-box-orient: vertical;
    /* 剪裁气泡形状 */
   clip-path: polygon(0% 0%, 100% 0%, 100% 100%, calc(50% + 6px) 100%, 50% calc(100% + 6px), calc(50% - 6px) 100%, 0% 100%);
 }
```

最后出来的效果：

![](pictures\tt_final.png)

浏览器兼容性：

![](pictures\tt_clip_path.png)

Edge和IE需要使用url，其它的都支持。

#### 总结

最终是使用`clip-path`来剪裁一个和气泡一模一样的形状来隐藏溢出部分的，另外配合使用了多行缺省显示来实现，一个小气泡真是用尽了我毕生CSS所学。