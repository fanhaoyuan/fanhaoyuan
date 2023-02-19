---
title: 【JS】数据类型
---

# 数据类型

在`JavaScript`中，数据类型分为**基本数据类型**和**引用数据类型**。

基本类型（基本数值、基本数据类型）是一种既**非对象**也**无方法**的数据。

## 基本数据类型

### Boolean

在布尔值中，只有两个值：`true`和`false`。示例如下：

```js
const bool = true;

const bool1 = false;
```

<Alert type="info">
资料参考：<a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean">MDN</a>
</Alert>

### String

字符串类型在`JavaScript`中是使用`""`、`''`包裹着的数据类型。示例如下：

```js
const str = '1';

const str1 = 'string';

const str2 = 'string2';
```

#### 字符字面量

字符串类型中含有一些字符字面量：

| 字面量   | 含义                                                                      |
| -------- | ------------------------------------------------------------------------- |
| `\n`     | 换行                                                                      |
| `\t`     | 制表符                                                                    |
| `\b`     | 空格                                                                      |
| `\r`     | 回车                                                                      |
| `\f`     | 换页符                                                                    |
| `\\`     | 反斜杠                                                                    |
| `\'`     | 单引号                                                                    |
| `\“`     | 双引号                                                                    |
| `\0nnn`  | 八进制代码 nnn 表示的字符（n 是 0 到 7 中的一个八进制数字）               |
| `\xnn`   | 十六进制代码 nn 表示的字符（n 是 0 到 F 中的一个十六进制数字）            |
| `\unnnn` | 十六进制代码 nnnn 表示的 Unicode 字符（n 是 0 到 F 中的一个十六进制数字） |

<Alert type="info">
资料参考：<a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String">MDN</a>
</Alert>

### Number

在`JavaScript`中，数字类型使用小数点或者不使用小数点均可。

```js
const num = 1;

const num2 = 200104;

const num3 = 0.3;

const num4 = 0.1495;
```

#### 科学计数法

在超大或者是超小的数值上可以使用科学计数法。示例如下：

```js
const num = 123e5; // 12300000

const num1 = 123e-5; // 0.00123
```

#### 特殊值

在`number`类型中，有几个特殊的值。

-   `Infinity` 表示无穷大
-   `-Infinity` 表示无穷小
-   `NaN` (Not a Number) 表示非数值类型

```js
const a = Infinity;

const b = -Infinity;

const c = NaN;
```

<Alert type="info">
资料参考：<a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number">MDN</a>
</Alert>

### Null

`null`代表为空值，但`null`与`undefined`的意义并不相同，`null`代表的是该变量已经初始化，但值为空，`undefined`代表的是该变量尚未初始化。

```js
const a = null;

const b = null;
```

<Alert type="info">
资料参考：<a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/null">MDN</a>
</Alert>

### Undefined

`undefined`类型只有一个值，也就是`undefined`。

当一个变量在声明中没有初始化的时候，这个变量就默认为`undefined`。

```js
var a; // undefined

let b; // undefined
```

<Alert type="info">
资料参考：<a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined">MDN</a>
</Alert>

### Symbol

`symbol`是`ES6`新引入的数据类型。

`symbol`代表的是独一无二的值，最大的用法是用来定义对象的唯一属性名。

```js
const a = Symbol('string');

const b = Symbol('string');

console.log(a === b); // false
```

#### Symbol.for

如果要使用同一个`Symbol`值的话，可以使用`Symbol`中的`for`方法。

1. 首先会在全局搜索被登记的`Symbol`中是否有该字符串参数作为名称的`Symbol`值
2. 如果有即返回该`Symbol`值。
3. 若没有则新建并返回一个以该字符串参数为名称的`Symbol`值，并登记在全局环境中供搜索

```js
//case1
const sym = Symbol('string');

const a = Symbol.for('string');

console.log(sym === a); //true

//case2
const b = Symbol.for('string1');

const c = Symbol('string1');

console.log(b === c); //false
```

#### Symbol.keyFor

`Symbol.keyFor`返回一个已登记的`Symbol`类型值的`key`。

用来检测该字符串参数作为名称的`Symbol`值是否已被登记。

```js
const sym = Symbol('test');
Symbol.keyFor(sym); // test
```

<Alert type="info">
资料参考：<a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/symbol">MDN</a>
</Alert>

### BigInt

`BigInt`是一种内置对象，它提供了一种方法来表示大于`2e53-1`的整数。

这原本是 `JavaScript`中可以用`Number`表示的最大数字。

`BigInt`可以表示任意大的整数。

#### 用法

可以用在一个整数字面量后面加`n`的方式定义一个`BigInt`，如：`10n`，或者调用函数`BigInt()`。

```js
const bigNumber = BigInt(9007199254740991);

const bigNumber1 = 9007199254740991n;

console.log(bigNumber === bigNumber1); //true
```

<Alert type="info">
资料参考：<a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/BigInt">MDN</a>
</Alert>

## 引用数据类型

### Array

数组是一种使用整数作为键属性和长度属性之间关联的常规对象。

所以数组实际上也是对象。

```js
const array = [1, 2, 3];
//相当于
const object = {
    0: 1,
    1: 2,
    2: 3,
};
```

<Alert type="info">
资料参考：<a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/array">MDN</a>
</Alert>

### Object

在`JavaScript`里，对象可以被看作是一组属性的集合。

用对象字面量语法来定义一个对象时，会自动初始化一组属性。

在对象中，会以`key/value`的形式定义存储一个数据属性。`key`的值可以为`string`类型，也可以为`symbol`类型。`value`的值可以为任意值。

```js
const a = {
    key: 'value',
    [Symbol('key')]: 10000,
    key2: 'akaomf',
};
```

对象可以以构造器函数创建，也可以字面量定义来创建。

```js
const a = {};

const b = Object.create({});
```

每个对象对应一个内存地址。所以每定义一个对象，所分配的内存地址会不相同，即使他们看起来一样，但是实际上并不是同一个对象。

```js
const a = {};

const b = {};

console.log(a === c); //false
```

<Alert type="info">
资料参考：<a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/object">MDN</a>
</Alert>

### Function

函数实际上是对象。

每个函数都是`Function`类型的实例，而且与其他引用类型一样具有属性和方法。

`function`可以通过`function`语句来声明，或者使用构造函数来创建，也可以使用`ES6`的箭头函数定义。

```js
const fn1 = function () {
    //todo something...
};

function fn2() {
    //todo something...
}

const fn3 = () => {
    //todo something...
};

const fn4 = new Function('return 1 + 2'); //不推荐使用
//等同于
function fn4() {
    return 1 + 2;
}
```

<Alert type="info">
资料参考：<a href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/function">MDN</a>
</Alert>
