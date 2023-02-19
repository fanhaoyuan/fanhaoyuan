---
title: 【JS】InstanceOf
---

# InstanceOf

`instanceof` 运算符用于检测构造函数的 `prototype` 属性是否出现在某个实例对象的原型链上。

也就是说，看看这个对象是不是由这个模版对象来生成的。

> [模版对象](./PROTOTYPE.md)

## 检测类型

instanceof 只能检测数据类型为对象(数组)的数据。

-   如果是[基本类型](./DATA_TYPE.md),则一律返回 `false`
-   如果是 `null` 或者是 `undefined` ，也一律为 `false`
-   检查每一层模版，是否由该模版对象生成，如果是则返回 `true，` 如果没有任意一层符合，则返回 `false`

## 手动实现

利用循环来判断每一层的模版。

```ts
function instanceOf(obj, fn) {
    /**
     * 基本类型 返回 false
     */
    if (typeof obj !== 'object') {
        return false;
    }

    /**
     * 如果是 null 或者 undefined 也返回 false
     */
    if (!obj) {
        return false;
    }

    // let _proto = obj.__proto__
    let _proto = Object.getPrototypeOf(obj);

    /**
     * 递归原型对象
     */
    while (_proto) {
        /**
         * 如果其中一层模版对象 fn.prototype
         */
        if (_proto === fn.prototype) {
            return true;
        }

        // _proto = obj.__proto__
        _proto = Object.getPrototypeOf(_proto);
    }

    /**
     * 如果一直找不到
     *
     * 直到 null
     *
     * 则返回 false
     */
    return false;
}

class A {}

class B extends A {}

const a = new A();

const b = new B();

/**
 * 结果为true
 */
console.log(instanceOf(a, A));
console.log(instanceOf(a, Object));
console.log(instanceOf(b, B));
console.log(instanceOf(b, A));
console.log(instanceOf(b, Object));
console.log(instanceOf({}, Object));

/**
 * 结果为 false
 */
console.log(instanceOf(a, B));
console.log(instanceOf('1', Object));
console.log(instanceOf(1, Object));
console.log(instanceOf(undefined, Object));
console.log(instanceOf(null, Object));
console.log(instanceOf(111n, Object));
console.log(instanceOf(Symbol(), Symbol));
```
