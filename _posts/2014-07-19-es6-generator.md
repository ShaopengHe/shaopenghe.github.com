---
title: ES6 的 Generator
layout: post
tags:
    - javascript
    - nodejs
---

Generator 是ECMAScript Harmony (ES6) 引入的新特性，通过该特性可以改变异步代码结构，提高代码的可读性。

创建Generator函数

```
function* Demo () {     // function后面带上*号
    yield "hello";      // yield中断函数执行并返回值
    yield "world";
    return "!";
}
var demo = Demo();              // 需要实例化
console.log (demo.next());      // { value: 'hello', done: false }
console.log (demo.next());      // { value: 'world', done: false }
console.log (demo.next());      // { value: '!', done: true }
```
可以看出， 通过yield关键字，可以中断函数的执行，并返回一个对象。```{ value: 'hello', done: false}``` 其中done字段表示函数是否执行完毕。每次调用next方法就可以继续执行yield下面的代码。

再看一个复杂一点的例子：

```
function* Demo (num) {
    var result = num;
    var base = 0;
    base = yield result;        // result = 5

    console.log (base);         //2. 输出2， base的值等于第二次next调用的参数
    result *= base;             // result = 5 * 2
    base = yield result;

    console.log (base);         //4. 输出3，base的值等于第三次next调用的参数
    result *= base;             // result = 10 * 3
    return result;
}

var demo = Demo(5);
console.log (demo.next(1));     // 1. 输出 { value: 5, done: false }
console.log (demo.next(2));     // 3. 输出 { value: 10, done: false }
console.log (demo.next(3));     // 5. 输出 { value: 30, done: true }

/* 输出依次是：
{ value: 5, done: false }
2
{ value: 10, done: false }
3
{ value: 30, done: true }
*/

```

从上述例子可以得出以下结论：

  1. **Generator函数同样可以传递参数。**

  2. **yield表达式的值是等于后一次调用next方法传入的参数值。（可以理解为：执行表达式```yield result```后，函数立刻中断并返回第一次next()的结果， 当第二次调用next的时候，函数继续执行并将传入的参数将作为表达式```yield result```的值。）**

  3. **函数最后```return```跟```yield```实现一样的功能，并且返回结果中的```done```等于```true```。 （建议尝试把return语句去掉看看会发生什么。）**

----------

好，至此已经说完了Generator的使用，那Generator跟异步调用究竟有什么关系？


先说需求：执行异步任务A，B，C，要求执行完A之后得到的结果作为参数传递给任务B，任务B完成之后再将任务B的结果传递给任务C去执行。 这里为了方便，假设A，B，C都是同一类的任务，定义函数如下：

```
/*
 *  异步任务，传入时间time，callback返回结果time
 */
function somethingAsync (time, callback) {
    setTimeout (function (){
        var err = null;
        console.log ("delay: " + time + "ms");
        if (time >= 10000) {
            err = new Error('timeout');
        }
        callback(err, time);
    }, time);
}
```

一般异步的代码是这样(callback hell)：

```
// task A: 1000ms
somethingAsync(1000, function (err, time) {
    if (err) {
        throw new Error(err);
    }   
    // task B: (1000 + 100)ms
    somethingAsync(time + 100, function (err, time) {
        if (err) {
            throw new Error(err);
        }   
        // task C: (1000 + 100 + 500)ms
        somethingAsync(time + 500, function (err, time) {
            if (err) {
                throw new Error(err);
            }   
            console.log ("done!");
        }); 
    }); 
});
```

可以看出，异步代码会出现层层嵌套的结构，假如再多几个任务D，E，F，G，代码会变得难以阅读和维护（现在很多开源模块都能改善这个问题，例如：[async](https://github.com/caolan/async), [Q](https://github.com/kriskowal/q), [wind.js](https://github.com/JeffreyZhao/wind), [then.js](https://github.com/teambition/then.js) 等）。引入Generator之后，可以利用yield中断特性改变代码结构：

```
wrap(function *() {
    var time = 1000;
    time = yield somethingAsyncWrap (time);         // task A: 1000
    time = yield somethingAsyncWrap (time + 100);   // task B: 1000 + 100
    time = yield somethingAsyncWrap (time + 500);   // task C: 1000 + 100 + 500
    return time;;
})(function (err, time){
    if (err) {
        console.error (err);
    } else {
        console.log ("done! time: " + time);    // done!
    }   
});
```

以上是改造完后的逻辑代码，通过wrap函数，消除了层层嵌套的callback结构，方便阅读以及维护。

以下是wrap函数的粗略实现：

```
/*
 * 定义wrap函数，利用Generator的特性消除callback嵌套
 */
function wrap (Gen) {
    var gen = new Gen();
    
    // 返回一个函数，该函数传入callback，当最后一个异步操作完成后就会触发该callback。
    return function (callback) {
        var next = function (result) {
            var fn = result.value;
            if (result.done) {  // 已经完成所有yield中断，最后一个异步任务完成
                callback (null, result.value);
            }   
            else {
                // 执行异步任务
                fn (function (err, val) {
                    if (err) {
                        callback (err);
                    }   
                    else {
                        // 完成后以返回结果作为参数调用下一次yield（下一个任务）
                        result = gen.next (val);
                        // 递归实现
                        next (result);
                    }   
                }); 
            }   
        };  
        next (gen.next());
    };  
} 
```

除此之外，需要对原本异步函数进行封装，以便适应wrap的调用。（这是这种方法的一个缺点： 需要对每个异步函数封装）

```
/* 封装原本的异步函数使其适应wrap函数的调用。*/
function somethingAsyncWrap (time) {
    return function (callback) {
        somethingAsync (time, callback);
    }   
}
```

到此为止， 我们实现了利用Generator的特性对异步代码结构进行优化。

其中，上述的```wrap```函数其实是[TJ大神](https://github.com/visionmedia)开源项目[co](https://github.com/visionmedia/co)的粗略版，建议大家去阅读一下大神的源码。 另外，TJ大神还有[thunkify](https://github.com/visionmedia/node-thunkify)用于封装异步函数。
co是新一代框架[koa](https://github.com/koajs/koa)的核心，建议node.js的朋友去了解一下。
[co example](https://github.com/visionmedia/co#example)

----------

P.S: 以上代码基于Node v0.11.13 harmony模式运行。

```
$ node --harmony
```

只有v0.11.*的版本才支持Generator的特性。

