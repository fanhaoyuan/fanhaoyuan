---
title: 【JS】__proto__ 与 prototype
---

# \_\_proto\_\_ 与 prototype

## 介绍

### \_\_proto\_\_

`__proto__`是对象中都会含有的属性，是`[[Prototype]]`的一个`getter`方法。

### prototype

`prototype` 是**函数**特有的属性，因为函数也是对象的一种，所以在函数上也会有`__proto__`属性。

## 关系

`__proto__` 与 `prototype` 看上去很像，所以他们是什么关系呢？

`__proto__` 其实是当前对象的原型对象，也就是用于生成这个对象的对象。

看上去是不是很复杂，我们用代码来创建一个对象吧。

### 原型对象（模板）

![以原型对象创建一个新的对象](./images/prototype_1.gif)

```ts
const obj = {
    a: 'proto',
};

const obj2 = Object.create(obj);

console.log(obj2.a); // 'proto'
```

上述例子中, `obj`就是`obj2`的原型对象。所以我们可以在 `obj2` 访问到 `obj` 的属性。

```ts
console.log(obj.__proto__ === obj); // true
```

所以，原型对象就是用来创建这个对象的**模版**。

这个时候，我们应该明白什么叫做原型对象了。

### 创建模版

上文有提到，`prototype` 属性只存在于函数中，所以我们先声明一个函数。

```ts
function Fn() {}
```

这看上去是一个普通的函数，但是这也可以是一个构造函数。

`prototype` 在函数中是一个对象，他里面包含着一个属性`constructor`，这个属性也就是所说的构造函数，它是等价于 `Fn` 。

```ts
console.log(Fn.prototype.constructor === Fn); // true
```

我们可以对`prototype`进行一系列的操作

```ts
Fn.prototype.sayHello = () => {
    console.log('hello');
};
```

我们创建一个利用 `Fn.prototype` 作为原型对象来生成一个对象。

```ts
const obj = Object.create(Fn.prototype);

obj.sayHello(); // 'hello'
```

这样看起来，与上文的原型对象的例子是一样的。fn 这个对象是由 `Fn.prototype` 这个模版创建出来的。

```ts
console.log(obj.__proto__ === Fn.prototype); // true
```

上面的例子，可以用 `new` 操作符来简化。

```ts
const obj = new Fn();
```

所以 `prototype` 相当于一个原型对象

## 总结

![以原型对象创建一个新的对象](./images/prototype_2.gif)

-   `prototype` 属性是一个对象的原型对象**模版**。

-   `__proto__` 是用来创建当前对象的模版的集合。

为什么是一个集合？是因为模版也可以由模版创建而成，这个集合，也就是所说的原型链。所以在这个对象中总有一层 `__proto__` 等价于 `prototype`。

虽然子子孙孙无尽也，但是最上层的模版一定不是由模版生成的，所以 `__proto__` 的尽头是 `null`
