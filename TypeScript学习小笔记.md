# TypeScript学习小笔记

#### 编辑器设置

ts文件编译成js文件执行命令`tsc fileName`，可在webstorm中设置改变后自动编译。

settings -> language & frameworks -> TypeScript -> 勾选recompile on changes

![image-20210817180503070](C:\Users\牛昕怡\AppData\Roaming\Typora\typora-user-images\image-20210817180503070.png)

#### 学习文档

http://ts.xcatliu.com/basics/primitive-data-types.html

#### 基本数据类型

`null`和`undefine`也是原始数据类型

```ts
let u: undefined = undefined;
let n: null = null;
```

**如果定义的时候没有赋值，不管之后有没有赋值，都会被推断成 `any` 类型而完全不被类型检查**。

如果定义的时候被赋值了，就会进行类型检查，自动推断出变量类型。

**联合类型（Union Types）**表示取值可以为多种类型中的一种。联合类型使用 `|` 分隔每个类型。

```ts
let myFavoriteNumber: string | number;
myFavoriteNumber = 'seven';
myFavoriteNumber = 7;
```

在 TypeScript 中，我们使用**接口（Interfaces）**来定义对象的类型。

应用了接口类型的变量必须与接口保持一致，不能多也不能少，当不需要与一个接口中的所有的变量类型保持一致时，可以使用可选属性。可选属性的含义是该变量允许不存在。

```ts
interface Person {
    name: string;
    age?: number;
}

let tom: Person = {
    name: 'Tom'
};
```

有时候我们需要在接口中添加**任意属性**

```ts
interface Person {
    name: string;
    age?: number;
    [propName: string]: any;
}

let tom: Person = {
    name: 'Tom',
    gender: 'male'
};
```

**一旦定义了任意属性，那么确定属性和可选属性的类型都必须是它的类型的子集**

```ts
interface Person {
    name: string;
    age?: number;
    [propName: string]: string;
}

let tom: Person = {
    name: 'Tom',
    age: 25,
    gender: 'male'
};
```

报错，`number`类型的`age`不是`string`类型的子集。

需要使用联合类型：

```ts
interface Person {
    name: string;
    age?: number;
    [propName: string]: string | number;
}

let tom: Person = {
    name: 'Tom',
    age: 25,
    gender: 'male'
};
```

有时候我们希望对象中的一些字段只能在创建的时候被赋值，那么可以用 `readonly` 定义**只读属性**

```ts
interface Person {
    readonly id: number;
    name: string;
    age?: number;
    [propName: string]: any;
}
```

**只读的约束存在于第一次给对象赋值的时候，而不是第一次给只读属性赋值的时候**

如果对象中包含只读属性，则第一次给对象赋值时必须给只读属性赋值。**包含只读属性的对象在第一次赋值时是否需要一次性赋值？**

**数组类型**

```ts
let fibonacci: number[] = [1, 1, 2, 3, 5];
```

**函数声明**

```ts
function sum(x: number, y: number): number {
    return x + y;
}
```

注意：输入多的或少的变量时不允许的。

```ts
let mySum: (x: number, y: number) => number = function (x: number, y: number): number {
    return x + y;
};
```

在 TypeScript 的类型定义中，`=>` 用来表示函数的定义，左边是输入类型，需要用括号括起来，右边是输出类型。

如果想要不传某个参数，可以使用**可选参数**，但是可选参数必须在必传参数之后。

在ES6中，允许给函数参数添加默认值，添加了默认值的参数会被认为是可选的：

```ts
function buildName(firstName: string, lastName: string = 'Cat') {
    return firstName + ' ' + lastName;
}
let tomcat = buildName('Tom', 'Cat');
let tom = buildName('Tom');
```

此时就不受**可选参数必须在必传参数后面**限制了。

**剩余参数和可选参数同时存在？**

**函数重载**

```ts
function reverse(x: number): number;
function reverse(x: string): string;
function reverse(x: number | string): number | string | void {
    if (typeof x === 'number') {
        return Number(x.toString().split('').reverse().join(''));
    } else if (typeof x === 'string') {
        return x.split('').reverse().join('');
    }
}
```

TypeScript 会优先从最前面的函数定义开始匹配，所以多个函数定义如果有包含关系，需要优先把精确的定义写在前面。

**断言**

- 联合类型可以被断言为其中一个类型
- 父类可以被断言为子类
- 任何类型都可以被断言为 any
- any 可以被断言为任何类型
- 要使得 `A` 能够被断言为 `B`，只需要 `A` 兼容 `B` 或 `B` 兼容 `A` 即可

```ts
interface Cat {
    name: string;
    run(): void;
}
interface Fish {
    name: string;
    swim(): void;
}

function isFish(animal: Cat | Fish) {
    if (typeof (animal as Fish).swim === 'function') { // typeof animal.swim === 'function' 会报错
        return true;
    }
    return false;
}
```

类型断言只能够「欺骗」TypeScript 编译器，无法避免运行时的错误，反而滥用类型断言可能会导致运行时错误。

要判断的值所属类型是一个接口，使用断言，如果是一个类，则可以直接使用`instanceof`。

在给一个对象直接添加一个属性时，可以给他断言为`any`类型再赋值，直接添加是会报错的。

```ts
(window as any).foo = 1; // 编译通过
window.foo = 1; // 编译失败
```

类型断言不是类型转换，它不会真的影响到变量的类型。
