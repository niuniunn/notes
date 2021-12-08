# sass基础（二）

### 插值

通过 `#{}` 插值语句可以在选择器或属性名中使用变量。

在选择器中使用：

```scss
@mixin generate-sizes($class, $small, $medium, $big) {
  #{$class}-small {
    font-size: $small;
  }
  #{$class}-medium {
    font-size: $medium;
  }
  #{$class}-big {
    font-size: $big;
  }
}

@include generate-sizes("header-text", 12px, 14px, 16px);
```

```css
header-text-small {
  font-size: 12px;
}

header-text-medium {
  font-size: 14px;
}

header-text-big {
  font-size: 16px;
}
```

在属性名中使用：

```scss
@mixin padding-direction ($direction, $size) {
  padding-#{$direction}: $size;
}

.box5 {
  @include padding-direction("left", 12px);
}
```

```scss
.box5 {
  padding-left: 12px;
}
```

还可以在何处使用：

继承使用插值：

```scss
%btn-selected {
  color: #fff;
  background-color: #07f;
}

$status: "selected";

.box6 {
  @extend %btn-#{$status};
}
```

```css
.box6 {
  color: #fff;
  background-color: #07f;
}
```

反例：

- 不能在属性值中使用；
- 不能在混合器@include 混合器名中使用；

### 数据类型

- 数字，`1, 2, 13, 10px`
- 字符串，有引号字符串与无引号字符串，`"foo", 'bar', baz`
- 颜色，`blue, #04a3f9, rgba(255,0,0,0.5)`
- 布尔型，`true, false`
- 空值，`null`
- 数组 (list)，用空格或逗号作分隔符，`1.5em 1em 0 2em, Helvetica, Arial, sans-serif`
- maps, 相当于 JavaScript 的 object，`(key1: value1, key2: value2)`

```scss
$map: (
    $key1: value1,
    $key2: value2,
    $key3: value3
)
```

SassScript 也支持其他 CSS 属性值，比如 Unicode 字符集，或 `!important` 声明。然而Sass 不会特殊对待这些属性值，一律视为无引号字符串。

### 运算

所有数据类型均支持相等运算 `==` 或 `!=`，此外，每种数据类型也有其各自支持的运算方式。

#### 数字运算

SassScript 支持数字的加减乘除、取余等运算 (`+, -, *, /, %`)，如果必要会在不同单位间转换值。

关系运算 `<, >, <=, >=` 也可用于数字运算。

##### 除法运算

因除号在css中已经被作为一种符号使用，因此在作除法运算时需要特别注意一下。

”/ ”符号被当作除法运算符时有以下几种情况：

•   如果数值或它的任意部分是存储在一个变量中或是函数的返回值。
•   如果数值被圆括号包围。
•   如果数值是另一个数学表达式的一部分。

```scss
p {
  font: 10px/8px;             // 纯 CSS，不是除法运算
  $width: 1000px;
  width: $width/2;            // 使用了变量，是除法运算
  width: round(1.5)/2;        // 使用了函数，是除法运算
  height: (500px/2);          // 使用了圆括号，是除法运算
  margin-left: 5px + 8px/2px; // 使用了加（+）号，是除法运算
}
```

#### 颜色运算

不推荐使用，在将来可能会被遗弃。

#### 字符运算

如果有引号的字符串被添加了一个没有引号的字符串 （也就是，带引号的字符串在 + 符号左侧）， 结果会是一个有引号的字符串。 同样的，如果一个没有引号的字符串被添加了一个有引号的字符串 （没有引号的字符串在 + 符号左侧）， 结果将是一个没有引号的字符串。

```scss
p:before {
  content: "Foo " + Bar;
  font-family: sans- + "serif";
}
```

```css
p:before {
  content: "Foo Bar";
  font-family: sans-serif;
}
```

### 控制指令

##### @if

@if 指令是一个 SassScript，它可以根据条件来处理样式块，如果条件为 true 返回一个样式块，反之 false 返回另一个样式块。在 Sass 中除了 @if 之，还可以配合 @else if 和 @else 一起使用。

```scss
@mixin blockOrHidden($boolean:true) {
  @if $boolean {
    display: block;
  }
  @else {
    display: none;
  }
}

.block {
  @include blockOrHidden;
}

.hidden{
  @include blockOrHidden(false);
}
```

```css
.block {
  display: block;
}

.hidden {
  display: none;
}
```

##### @for循环

在制作网格系统的时候，大家应该对 .col1~.col12 这样的印象较深。在 CSS 中你需要一个一个去书写，但在 Sass 中，可以使用 @for 循环来完成。在 Sass 的 @for 循环中有两种方式：

```
@for $i from <start> through <end>
@for $i from <start> to <end>
```

- $i 表示变量
- start 表示起始值
- end 表示结束值

这两个的区别是关键字 through 表示包括 end 这个数，而 to 则不包括 end 这个数。

```scss
@for $i from 1 through 3 {
  .item-#{$i} { width: 2em * $i; }
}
```

```css
.item-1 {
  width: 2em;
}

.item-2 {
  width: 4em;
}

.item-3 {
  width: 6em;
}
```

##### @while循环

@while 指令也需要 SassScript 表达式（像其他指令一样），并且会生成不同的样式块，直到表达式值为 false 时停止循环。这个和 @for 指令很相似，只要 @while 后面的条件为 true 就会执行。

```scss
$count: 4;
@while $count > 0 {
  .while-#{$count} {
    width: $count + 20px;
  }
  $count: $count - 1;
}
```

```css
.while-4 {
  width: 24px;
}

.while-3 {
  width: 23px;
}

.while-2 {
  width: 22px;
}

.while-1 {
  width: 21px;
}
```

##### @each循环

@each 循环就是去遍历一个列表，然后从列表中取出对应的值。

@each 循环指令的形式：

```
@each $var in <list>
```

```scss
$icon-list: success, fail, edit, delete;
@each $item in $icon-list {
  .el-icon-#{$item} {
    background-image: url("images/#{$item}.png");
  }
}
```

```css
.el-icon-success {
  background-image: url("images/success.png");
}

.el-icon-fail {
  background-image: url("images/fail.png");
}

.el-icon-edit {
  background-image: url("images/edit.png");
}

.el-icon-delete {
  background-image: url("images/delete.png");
}
```

### 函数

#### 字符串函数

##### unquote 去除引号

unquote() 函数主要是用来删除一个字符串中的引号，如果这个字符串没有带有引号，将返回原始的字符串。

```scss
.box9 {
  content: unquote("hello sass");
}
```

```css
.box9 {
  content: hello sass;
}
```

##### quote添加引号

quote() 函数刚好与 unquote() 函数功能相反，主要用来给字符串添加引号。如果字符串，自身带有引号会统一换成双引号 ""。

字符串中间有单引号或者空格必须使用双引号括起来，否则会报错。

```scss
.box10 {
  content: quote(hello);
}

.box11 {
  content: quote('hello sass');
}
```

```css
.box10 {
  content: "hello";
}

.box11 {
  content: "hello sass";
}
```

##### to-lower-case 转换为小写

##### to-upper-case 转换为大写

#### 数字函数

**percentage()**函数主要是将一个不带单位的数字转换成百分比形式。

**round()** 函数可以将一个数四舍五入为一个最接近的整数。

**ceil()**函数是向上取整函数。

**floor()**函数是向下取整函数。

**abs()** 函数会返回一个数的绝对值。

**min()** 函数功能主要是在多个数之中返回最小值。

**max()**函数是在多个数中返回最大值。

**random()** 函数是用来获取一个0-1的随机数。如果传入了参数，则随机数是在1-参数之间取值，包括1和参数。

**注：除了percentage()函数和random函数，其余函数的参数都可带单位，percentage()带单位会报错，random()带单位编译之后单位自动去除。**

#### 列表函数

**length()**函数返回列表的长度。

如果参数是一个list变量，则该list变量可以使用逗号或空格隔开。如果是直接在length()中拼接，例如：`length(1,1,1,1)`则会报错，只能使用空格，改为`length(1 1 1 1)`编译通过。

**nth()** 函数用来指定列表中某个位置的值。不过在 Sass 中，nth() 函数和其他语言不同，1 是指列表中的第一个标签值，2 是指列给中的第二个标签值，依此类推

注：在 nth($list,$n) 函数中的 $n 必须是大于 0 的整数

**join()** 函数是将两个列表连接合并成一个列表。

在 join() 函数中还有一个很特别的参数 $separator，这个参数主要是用来给列表函数连接列表值是，使用的分隔符号，默认值为 auto，还有 comma 和 space 两个值，comma是逗号分隔，space是空格分隔。

**append()** 函数是用来将某个值插入到列表中，并且处于最末位，在 append() 函数中，可以显示的设置 $separator 参数，参数值同join()。

**zip()**函数将多个列表值转成一个多维的列表。

```
zip(a b c, 1 2 3, A B C)
a 1 A, b 2 B, c 3 C
```

**index()** 函数类似于索引一样，主要让你找到某个值在列表中所处的位置。在 Sass 中，第一个值就是1，第二个值就是 2，依此类推。如果没有在列表中找到，就会返回null。

#### Introspection函数

**type-of()** 函数主要用来判断一个值是属于什么类型：

返回值：

- number 为数值型。
- string 为字符串型。
- bool 为布尔型。
- color 为颜色型。

**unit()** 函数主要是用来获取一个值所使用的单位，碰到复杂的计算时，其能根据运算得到一个“多单位组合”的值，不过只充许乘、除运算。

**unitless()** 函数用来判断一个值是否带有单位，如果不带单位返回的值为 true，带单位返回的值为 false。可以在混合器中使用，当调用时没有加单位时，用来添加一个默认单位。

**comparable()** 函数主要是用来判断两个数是否可以进行“加，减”以及“合并”。如果可以返回的值为 true，如果不可以返回的值是 false。

**Miscellaneous**函数称为三元条件函数，主要因为他和 JavaScript 中的三元判断非常的相似。他有两个值，当条件成立返回一种值，当条件不成立时返回另一种值。

```scss
if($condition,$if-true,$if-false)
```

#### Maps函数

**map-get($map,$key)** 函数的作用是根据 $key 参数，返回 $key 在 $map 中对应的 value 值。如果 $key 不存在 $map中，将返回 null 值。此函数包括两个参数：

- $map：定义好的 map。
- $key：需要遍历的 key。

**map-has-key($map,$key)** 函数将返回一个布尔值。当 $map 中有这个 $key，则函数返回 true，否则返回 false。

**map-keys($map)** 函数将会返回 $map 中的所有 key组成的一个列表。

**map-values($map)** 函数返回$map中所有的value组成的一个列表。

**map-merge($map1,$map2)** 函数是将 $map1 和 $map2 合并，然后得到一个新的 $map。

**map-remove($map,$key)** 函数是用来删除当前 $map 中的某一个 $key，从而得到一个新的 map。如果删除的key不存在，则返回和原来相同的map。

#### RGB颜色函数

- **rgb($red,$green,$blue)**：根据红、绿、蓝三个值创建一个颜色；
- **rgba($red,$green,$blue,$alpha)**：根据红、绿、蓝和透明度值创建一个颜色；
- **red($color)**：从一个颜色中获取其中红色值；
- **green($color)**：从一个颜色中获取其中绿色值；
- **blue($color)**：从一个颜色中获取其中蓝色值；
- **mix($color-1,$color-2,[$weight])**：把两种颜色混合在一起，默认值为50%。

#### hsl函数

- **hsl($hue,$saturation,$lightness)**：通过色相（hue）、饱和度(saturation)和亮度（lightness）的值创建一个颜色；
- **hsla($hue,$saturation,$lightness,$alpha)**：通过色相（hue）、饱和度(saturation)、亮度（lightness）和透明（alpha）的值创建一个颜色；
- **hue($color)**：从一个颜色中获取色相（hue）值；
- **saturation($color**)：从一个颜色中获取饱和度（saturation）值；
- **lightness($color)**：从一个颜色中获取亮度（lightness）值；
- **adjust-hue($color,$degrees)**：通过改变一个颜色的色相值，创建一个新的颜色；
- **lighten($color,$amount)**：通过改变颜色的亮度值，让颜色变亮，创建一个新的颜色；
- **darken($color,$amount)**：通过改变颜色的亮度值，让颜色变暗，创建一个新的颜色；
- **saturate($color,$amount)**：通过改变颜色的饱和度值，让颜色更饱和，从而创建一个新的颜色
- **desaturate($color,$amount)**：通过改变颜色的饱和度值，让颜色更少的饱和，从而创建出一个新的颜色；
- **grayscale($color)**：将一个颜色变成灰色，相当于desaturate($color,100%);
- **complement($color)**：返回一个补充色，相当于adjust-hue($color,180deg);
- **invert($color)**：反回一个反相色，红、绿、蓝色值倒过来，而透明度不变。

#### Opacity函数

-  **alpha($color) /opacity($color)**：获取颜色透明度值；
-   **rgba($color, $alpha)**：改变颜色的透明度值；
-   **opacify($color, $amount) / fade-in($color, $amount)**：使颜色更不透明；
-   **transparentize($color, $amount) / fade-out($color, $amount)**：使颜色更加透明。
