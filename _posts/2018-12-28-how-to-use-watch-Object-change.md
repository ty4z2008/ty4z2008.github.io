---
layout: post
title:  "在JS中如何检测对象的改变？"
date:   2018-12-28 22:53:49
categories: JavaScript
tags: JavaScript
---

有时候在业务当中会遇到当某个对象改变的时候做一些逻辑处理。在Vue里面数据的双向绑定则是也提到监控对象的变化。主要是利用Object.defineProperty特性。下面是一段对Object.defineProperty扩展实现对象的监控。

```javascript
//https://gist.github.com/eligrey/384583
// object.watch
if (!Object.prototype.watch) {
    //在所有对象定义一个watch熟悉，设置为不可重写
    Object.defineProperty(Object.prototype, "watch", {
        //当且仅当该属性的enumerable为true时，该属性才能够出现在对象的枚举属性中。
          enumerable: false
        //当且仅当该属性的 configurable 为 true 时，该属性描述符才能够被改变，同时该属性也能从对应的对象上被删除
        , configurable: true 
        //当且仅当该属性的writable为true时，value才能被赋值运算符改变。
        , writable: false
        //该属性对应的值。可以是任何有效的 JavaScript 值（数值，对象，函数等）。
        , value: function (prop, handler) {
        //watch属性的值是一个函数。参数为对象的key和回调
            var
              oldval = this[prop]
            , newval = oldval
            //读取时直接返回
            , getter = function () {
                return newval;
            }
            //修改时执行回调，回调的参数是key,旧值，新值
            , setter = function (val) {
                oldval = newval;
                return newval = handler.call(this, prop, oldval, val);
            }
            ;
            // can't watch constants
            if (delete this[prop]) { 
                Object.defineProperty(this, prop, {
                    //一个给属性提供 getter 的方法，如果没有 getter 则为 undefined。
                      get: getter
                    //修改时执行
                    , set: setter
                    , enumerable: true
                    , configurable: true
                });
            }
        }
    });
}

// object.unwatch,解除绑定
if (!Object.prototype.unwatch) {
    Object.defineProperty(Object.prototype, "unwatch", {
          enumerable: false
        , configurable: true
        , writable: false
        , value: function (prop) {
            var val = this[prop];
            delete this[prop]; // remove accessors
            this[prop] = val;
        }
    });
}
//example
let a = {test:1};    
a.watch('test',function(){
    console.log('change')
});
a.test = 3;
a.unwatch('test');
a.test = 2;
```

有一个开发者基于上面的思想实现了一个库，可以在多种环境下面使用。[Watch.JS](https://github.com/melanke/Watch.JS/blob/master/src/watch.js)

参考资料

- [Object.defineProperty()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
