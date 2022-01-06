---
date: 2021-03-17 
title: 浅析深拷贝
tags:
  - 2021
  - js进阶
describe: 深拷贝看似简单，实际上要实现一个完善的深拷贝是很复杂的，本文实现了一个简单的深拷贝函数。
---

### 你真的理解深拷贝吗？

先看看深拷贝和浅拷贝的定义：

浅拷贝：

> 创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。如果属性是基本类型，拷贝的就是基本类型的值，如果属性是引用类型，拷贝的就是内存地址 ，所以如果其中一个对象改变了这个地址，就会影响到另一个对象。

深拷贝：

> 将一个对象从内存中完整的拷贝一份出来,从堆内存中开辟一个新的区域存放新对象,且修改新对象不会影响原对象。

下面来一步步实现一个有些意思的深拷贝。

### 乞丐版

```js
JSON.parse(JSON.stringify(obj));
```

能够应付大部分场景，但缺点是：

1. 不能copy函数

2. 不能拷贝Date等对象

3. 拷贝循环引用时会报错

### 基础版本

```js
function deepCopy(source) {
    if (typeof source !== 'object') {
        let target = Array.isArray(source) ? [] : {}; // 兼容数组
        for(const key in source) {
            target[key] = deepCopy(source[key]);
        }
        return target;
    }else {
        return source;
    }
}
```

### 解决循环引用的版本

复制循环引用的问题是因为递归进入死循环导致栈溢出，解决思路是将每一次复制的内容保存起来，复制时去查找之前是不是已经复制过，如果复制过就用已经复制的值。

```js
function deepCopy(source, map = new Map()) {
    if (typeof source === 'object' && source !== null) {
        let target = Array.isArray(source) ? [] : {};
        const oldValue = map.get(source);
        if (oldValue) {
            return oldValue;
        }
        for(const key in source) {
            target[key] = deepCopy(source[key], map);
        }
        map.set(source, target);
        return target;
    }else {
        return source;
    }
}
```

### 还有哪些问题？

- 对于函数，直接返回的原始引用，这是因为克隆函数是没什么意义的，包括lodash中也是直接返回的，如果真要写的话可以参考以下：
```js
function cloneFunction(func) {
    const funcString = func.toString();
    if (func.prototype) {
        // 普通函数
        const paramReg = /(?<=\().+(?=\)\s+{)/;
        const bodyReg = /(?<={)(.|\n)+(?=})/m;
        const param = paramReg.exec(funcString);
        const body = bodyReg.exec(funcString);
        if (body) {
            // console.log('匹配到函数体：', body[0]);
            if (param) {
                const paramArr = param[0].split(',');
                // console.log('匹配到参数：', paramArr);
                return new Function(...paramArr, body[0]);
            } else {
                return new Function(body[0]);
            }
        } else {
            return null;
        }
    } else {
        // 箭头函数
        return eval(funcString);
    }
}
```