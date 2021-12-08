#### el-table高度自适应

`<el-table>`高度是由表格的header、body、footer的高度组成。

表格总高度是通过table的DOM实例的clientHeight获取。

header高度根据是否显示表头而定，不显示表头header高度为0，显示表头则是表头的offsetHeight。header部分是一个表格。

body部分最多是由三个表格组成，没有设置固定列时，只有一个表格组成，当设置了固定列，左侧固定列是一个表格，右侧固定列又是另一个表格。（最多可能由七个表格组成：表头、body、左右固定列分别又是两个table，footer也是一个表格）

footer是在`show-summary`为true时才会存在，对应表尾合计功能。footer也是通过获取offsetHeight得到的。

Body Height = Table Height - Table Header Height

固定列高度 Fixed Body Height = Table Height - Table Header Height - Scroll Bar Height



#### 核心代码

```javascript
// table-layout.js
// 设置表格的总体高度
setHeight(value, prop = 'height') {
  if (Vue.prototype.$isServer) return;
  const el = this.table.$el;
  value = parseHeight(value);
  this.height = value;

  if (!el && (value || value === 0)) return Vue.nextTick(() => this.setHeight(value, prop));

  if (typeof value === 'number') {
    el.style[prop] = value + 'px';
    this.updateElsHeight();
  } else if (typeof value === 'string') {
    el.style[prop] = value;
    this.updateElsHeight();
  }
}

// 更新表格高度
updateElsHeight() {
  if (!this.table.$ready) return Vue.nextTick(() => this.updateElsHeight());
  const { headerWrapper, appendWrapper, footerWrapper } = this.table.$refs;
  this.appendHeight = appendWrapper ? appendWrapper.offsetHeight : 0;

  if (this.showHeader && !headerWrapper) return;

  // fix issue (https://github.com/ElemeFE/element/pull/16956)
  const headerTrElm = headerWrapper ? headerWrapper.querySelector('.el-table__header tr') : null;
  const noneHeader = this.headerDisplayNone(headerTrElm);

  const headerHeight = this.headerHeight = !this.showHeader ? 0 : headerWrapper.offsetHeight;  // header高度
  if (this.showHeader && !noneHeader && headerWrapper.offsetWidth > 0 && (this.table.columns || []).length > 0 && headerHeight < 2) {
    return Vue.nextTick(() => this.updateElsHeight());
  }
  const tableHeight = this.tableHeight = this.table.$el.clientHeight;  // 表格总高度
  const footerHeight = this.footerHeight = footerWrapper ? footerWrapper.offsetHeight : 0;
  if (this.height !== null) {
    this.bodyHeight = tableHeight - headerHeight - footerHeight + (footerWrapper ? 1 : 0);
  }
  this.fixedBodyHeight = this.scrollX ? (this.bodyHeight - this.gutterWidth) : this.bodyHeight;

  const noData = !(this.store.states.data && this.store.states.data.length);
  // 视口高度 = 表格总高高度 - 滚动条高度，横向滚动条无法点击就是此处视口高度 = 表格总高度了
  this.viewportHeight = this.scrollX ? tableHeight - (noData ? 0 : this.gutterWidth) : tableHeight;

  this.updateScrollY();
  this.notifyObservers('scrollable');
}
```

##### el-table横向滚动条在固定列处无法点击

```
Viewport Height = Table Height - Scroll Bar Height
Viewport Height 和 Table Height相等了，没有减到滚动条宽度
重新触发window的resize，再次计算高度固定列处的滚动条就可以点击拖动了
```

:slightly_smiling_face:解决方案：

1. 固定列把滚动条覆盖住了，利用定位时将固定列稍微上移（bottom: 6px），另外还要设置height: auto才有效果。

	```js
	.el-table__fixed-right, // 右固定列
	.el-table__fixed { // 固定列
	    height: auto !important;
	    // 改为自动高度后，设置与父容器的底部距离，高度会动态改变，
	    // 值可以设置比滚动条的高度稍微大一些
	    bottom: 18px; 
	}
	```

	

2. 给`.el-table__body-wrapper`设置`z-index: 2`，调整层级关系即可。但是会出现新的问题，固定列`tbody`的阴影会消失被盖住了，但是`theader`的阴影还在，又要修改`.el-table th.is-leaf`的`z-index`级别。

	这样就修改了`el-table`本身固定列的样式，着实不妥

3. 当获取数据后，手动触发一次`doLayout`事件，进行一次布局。目前看来此种方法比较完美。

	```js
	// 例如设备管理软件，底层通信网，标签管理，在线zigbee列表
	watch: {
	    tableOriginData: {
	        handler() {
	            this.table_data = this.tableOriginData;
	            this.select_all = false;
	            this.select_all_page = false;
	            ALL_SELECT_TAG_ID = new Set();
	            this.select_tag_list = ALL_SELECT_TAG_ID;
	            // ==========
	            this.$nextTick(() => {
	                this.$refs["table"].doLayout();
	            });
	        },
	    	immediate: true
	    }
	},
	```

	



element-ui表格组件卡顿问题：🧐

1. 在 ElementUI 实现中，左侧 `fixed` 表格和右侧 `fixed` 表格从 DOM 上都渲染了完整的列，然后从样式上控制它们的显隐，渲染了很多不必要的DOM。![](pictures\隐藏.png)

2. 每次勾选复选框时，整个表格都会全部重新渲染。

3. [《一顿操作，我把 Table 组件性能提升了十倍》]: https://juejin.cn/post/7007252464726458399

   文章中说每次渲染行都会读取一次`this.store`，触发响应式数据内部的 `getter` 函数，进而执行它的依赖收集，一个1000 * 7的表格就会收集7*1000次。

4. 数据太多时，滚动也很卡，可能是hover每一行时给当前行添加样式导致的，目前使用的是mouseenter和mouseleave，为什么不用hover伪类实现呢 :eyes:

```css
.el-table__body tr:hover {
 	background-color: ...;
}
```



针对第一个问题，在渲染的时候检查是否是隐藏单元格，如果是则`return`不进行渲染，普通表格目前没有问题，渲染时间会快一些，只有左固定列也是正常显示，当既有左固定列又有右固定列时，左固定列会错位，右固定列无法正常显示（可能是左边部分的表格因为没有渲染，而定位使用的是absolute,left: 0）。查找了是否是宽高引起的，好像宽高是自动计算但是又没有找到在哪里计算的，总的来说不能减少单元格的渲染，因为表格渲染的规则，每一列的宽度是由colgroup控制的，减少了DOM渲染，最右侧的固定列就会跑到左边，无法实现对齐，因此还是把原本隐藏的单元格渲染成了一个空的td元素，这样减少了一些计算和渲染，右侧固定列可以正常显示。



针对第二个问题，借鉴了vue3.2+新增的v-mome功能，通过diff算法，只渲染发生了改变的DOM元素，新加了一个变量`vnodeCache`来存储记录上一次的勾选情况。通过对比当前勾选和上一次勾选来决定是否重新渲染该行。单选的确会快很多，但是全选会比以前更耗时，因为总的来说全选是需要全部重新渲染的，又增加了一些判断条件。

但是出现了新的问题，18个测试没有通过，检查之后，发现事情并不简单，引起表格重绘不只是勾选复选框，还有很多其它的表格功能，例如排序、展开树型表格、筛选、单选等都会引起表格重绘，而当前只处理了多选框功能，所以存在其它功能失效的问题。



针对第三个问题，在render函数的时候获取一次store，然后作为参数传递，避免重复获取，在数据量很大的时候scripting时间才会得到明显的优化。

条件：2000条数据，有左固定列

![优化前](pictures\store优化前.png)  ![优化后](pictures\store优化后.png)



针对第四个问题，查看了scss样式代码：

```scss
@include m(enable-row-hover) {
  .el-table__body tr:hover > td.el-table__cell {
    background-color: $--table-row-hover-background-color;
  }
}
```

的确有这样一行，但是不明白为什么还要添加`mouseenter`和`mouseleave`事件。

