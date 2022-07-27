## JavaScript中的this和call/apply/bind

`this`关键字总是让初学者感到迷惑，在JavaScript中`this`总是指向一个对象，这个对象是基于函数的执行环境动态绑定的，而非函数被声明时的环境。

### this

`this`的指向在不同的函数调用形式中可能不同，但是**this 永远指向最后调用它的那个对象。**下面就从不同的函数调用方式来分析。

##### 1. 作为普通函数调用

```js
// "use strict";
var name = "globalName";
function getName() {
	return this.name;
}
getName();
```

当函数作为普通函数调用时，此时的`this`总是指向全局对象，在浏览器中非严格模式下这个对象是`window`，严格模式下`window`不能作为全局绑定，因此会是`undefined`。这里`getName`没有调用对象，默认就是`window`，相当于`window.getName()`。因此**this 永远指向最后调用它的那个对象。**



##### 2. 作为对象的方法调用

```js
var obj = {
	a: 1,
	getA: function () {
		console.log(this === obj) // true
		return this.a;
	}
}

obj.getA(); // 1
```

当函数作为对象的方法调用时，`this`指向该对象，因此`getA`方法中的`this`自然也是等于`obj`自身的，因为这里是`obj`调用的，还是那句话**this 永远指向最后调用它的那个对象。**

可是下面这种方式为什么又是`undefined`呢，这再一次证明了**this 永远指向最后调用它的那个对象。**这里的`this`指向`window`，而`window`中没有属性a，自然也就是`undefined`了，等同于上面的作为普通函数调用。

```js
var obj = {
	a: 1,
	getA: function () {
		return this.a;
	}
}

console.log(obj.getA()); // 1
var getA = obj.getA;
console.log(getA()); // undefined
```



##### 3. 构造器调用

> 如果函数调用前使用了 `new` 关键字, 则是调用了构造函数。

当使用`new`运算符调用函数时，该函数总会返回一个对象，通常情况下，构造器里的`this`就指向返回的这个对象。

```js
var myClass = function () {
	this.name = "Tom";
};
var obj = new myClass();
console.log(obj.name); // Tom
```

但是使用`new`调用构造器时还需要注意一个问题，如果构造器显式地返回了一个`object`类型的对象，那么此次运算结果最终会返回这个对象，而不是我们之前的`this`。



##### 4. 函数方法调用

> 在 JavaScript 中, 函数是对象。
>
> JavaScript 函数有它的属性和方法。
> call() 和 apply() 是预定义的函数方法。 两个方法可用于调用函数，两个方法的第一个参数必须是对象本身
>
> 在 JavaScript 严格模式(strict mode)下, 在调用函数时第一个参数会成为 this 的值， 即使该参数不是一个对象。
> 在 JavaScript 非严格模式(non-strict mode)下, 如果第一个参数的值是 null 或 undefined, 它将使用全局对象替代。

跟普通函数调用相比，用函数方法可以动态地传入函数的this：

```js
var obj1 = {
	name: "Tom",
	getName: function () {
		return this.name;
	}
}

var obj2 = {
	name: "Jerry"
}
console.log(obj1.getName()); // Tom
console.log(obj1.getName.call(obj2)); // Jerry
```

再来看看下面这个例子的this又是指向哪个对象的呢？

```js
var name = "windowName";
var a = {
	name : "aName",

	func: function () {
		setTimeout(  function () {
			console.log(this.name); // windowName
		},100 );
	}

};
a.func()
```

可能这里有些令人疑惑，这里调用func的明明是a对象，为什么this又是指向window的呢？[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/setTimeout)官方是这样说的：

> 由`setTimeout()`调用的代码运行在与所在函数完全分离的执行环境上。这会导致，这些代码中包含的 `this` 关键字在非严格模式会指向 `window` (或全局) 对象，严格模式下为 undefined，这和所期望的`this`的值是不一样的。
>
> 备注：即使是在严格模式下，`setTimeout()`的回调函数里面的`this`仍然默认指向`window`对象， 并不是`undefined`。

严格与非严格模式下`setTimeout()`的回调函数里的this都是默认指向`window`的。

### call、apply和bind

call、apply和bind都能够改变this指向，在传参和使用上略有不同，但第一个参数都是函数的this指向：

1. 非严格模式下：this指定为null，undefined，fun中的this指向window对象.
2. 严格模式下：函数的`this`为`undefined`
3. 值为原始值(数字，字符串，布尔值)的this会指向该原始值的自动包装对象，如 String、Number、Boolean



```js
var func = function (a, b, c) {
	console.log([a, b, c]);
}

func.apply(null, [1, 2, 3]);
func.call(null, 1, 2, 3);
func.bind(null, 1, 2, 3)();
```

`apply`接受两个参数，第一个参数指定了函数体内this的指向，第二个参数为一个带下标的集合，这个集合可以为数组也可以为类数组，apply把这个集合中的元素作为参数传递给被调用的函数。

`call`传入的参数数量不固定，与apply相同的是，第一个参数也是代表函数体内的this指向，从第二个参数开始往后，每个参数被依次传入函数。

`bind`看起来外观与其余两个不太一样，后面多了一对括号，`bind()` 方法创建一个新的函数，在 `bind()` 被调用时，这个新函数的 `this` 被指定为 `bind()` 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。因此这里`func.bind(null, 1, 2, 3)`只是创建了一个新的函数，所以需要我们手动去调用它。



##### 核心理念：借用方法

A对象有个方法，B对象这时候也需要用到同样的方法，那么这时候我们就可以借用一下A对象的方法，这样既达到了目的又减少了重复的代码。

这就是**借助已实现的方法，改变方法中数据的this指向，减少重复代码，节省内存。**



##### 应用场景

###### 1. 获取最值

用`apply`直接传递数组，省去展开数组的步骤。

```js
var arr = [1, 2, 3, 66, 45];
Math.max.apply(null, arr);
```

###### 2. 类数组借用数组方法

类数组不是真正的数组因此没有数组的方法，但是我们可以借用数组的方法，也就是上面提到的核心理念：**借用方法**。

例如类数组借用数组的`push`方法：

```js
var arrayLike = {
	0: 'OB',
	1: 'Koro1',
	length: 2
}
Array.prototype.push.call(arrayLike, '添加元素1', '添加元素2');
console.log(arrayLike) // {"0":"OB","1":"Koro1","2":"添加元素1","3":"添加元素2","length":4}
```

[附V8 push实现源码](https://github.com/v8/v8/blob/98d735069d0937f367852ed968a33210ceb527c2/src/js/array.js#L414)

伪数组再转换为数组

```js
Array.prototype.slice.call(arrayLike) // ['OB', 'Koro1', '添加元素1', '添加元素2']
```

###### 3. 判断数据类型

```js
function isType(data, type) {
    const typeObj = {
        '[object String]': 'string',
        '[object Number]': 'number',
        '[object Boolean]': 'boolean',
        '[object Null]': 'null',
        '[object Undefined]': 'undefined',
        '[object Object]': 'object',
        '[object Array]': 'array',
        '[object Function]': 'function',
        '[object Date]': 'date', // Object.prototype.toString.call(new Date())
        '[object RegExp]': 'regExp',
        '[object Map]': 'map',
        '[object Set]': 'set',
        '[object HTMLDivElement]': 'dom', // document.querySelector('#app')
        '[object WeakMap]': 'weakMap',
        '[object Window]': 'window',  // Object.prototype.toString.call(window)
        '[object Error]': 'error', // new Error('1')
        '[object Arguments]': 'arguments',
    }
    let name = Object.prototype.toString.call(data) // 借用Object.prototype.toString()获取数据类型
    let typeName = typeObj[name] || '未知类型' // 匹配数据类型
    return typeName === type // 判断该数据类型是否为传入的类型
}
console.log(
    isType({}, 'object'), // true
    isType([], 'array'), // true
    isType(new Date(), 'object'), // false
    isType(new Date(), 'date'), // true
)
```

参考资料

- [js基础-面试官想知道你有多理解call,apply,bind？](https://juejin.cn/post/6844903906279964686#heading-9)
- [this、apply、call、bind](https://juejin.cn/post/6844903496253177863)
