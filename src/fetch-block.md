# 一个可中断请求fetch的原理分析和应用

## 1.背景介绍

很多时候，当我们发起一个请求，我们需要对其进行请求超时处理，不能一直傻傻地在等待。通常情况下，我们需要使用Promise.race实现，如下：

```javascript
// 请求超时函数
const handleTimeOut = (delay=100) => {
    return new Promise(resolve => {
        setTimeout(() => {
            // 返回数据
            resolve({
                code: 1,
                msg: '请求超时'
            })
        }, delay)
    })
}

// 处理请求超时
Promise.race([handleTimeOut(), fetch(url)]).then((res) => {
    if (res.code === 1) {
        // 请求超时
    } else {
       // 处理正常返回 
    }
    // ....
})
```

可是这样会有一个问题，请求超时后，我们的请求依旧还是会正常发送出去，其实从性能上考虑，请求超时后我们应该中断fetch请求，以节省我们的IO资源，降低客户端和服务端的负载

```javascript
// 添加中止功能
const controller = new AbortController()
const { signal } = controller

// 请求超时函数
const handleTimeOut = (delay=100) => {
    return new Promise(resolve => {
        setTimeout(() => {
            // 触发中止
            controller.abort()
            // 返回数据
            resolve({
                code: 1,
                msg: '请求超时'
            })
        }, delay)
    })
}

// 处理请求超时，添加中止信号
Promise.race([handleTimeOut(), fetch(url, { signal })]).then((res) => {
    if (res.code === 1) {
        // 请求超时
    } else {
       // 处理正常返回 
    }
    // ....
})
```

AbortController究竟是什么？

它又是如何中止一个fetch请求的呢？

内部实现的原理是什么？

下面就让我们结合源码一探究竟

## 2.AbortController介绍

我们打开控制台，如下：
![](https://cdn.nlark.com/yuque/0/2020/png/1387052/1598523817528-ce8aa725-033f-40ad-903d-6701aa7d42ee.png#align=left&display=inline&height=145&margin=%5Bobject%20Object%5D&originHeight=145&originWidth=434&size=0&status=done&style=none&width=434)

AbortController由两部分构成：abort-signal和abort-controller，架构图如下：

![](https://cdn.nlark.com/yuque/0/2020/png/1387052/1598523817499-cfdcb889-09a9-49e8-ab89-d9fd060c2bd7.png#align=left&display=inline&height=461&margin=%5Bobject%20Object%5D&originHeight=461&originWidth=896&size=0&status=done&style=none&width=896)

红色部分是我们重点需要关注的部分，因为它们将直接体现在实际应用中

### 2-1）AbortSignal源码解析

```javascript
...
export default class AbortSignal extends EventTarget<Events, EventAttributes> {
    /**
     * 从abortedFlags中获取当前AbortSignal实例aborted状态
     */
    public get aborted(): boolean {
        const aborted = abortedFlags.get(this)
        if (typeof aborted !== "boolean") {
            throw new TypeError(
                `Expected 'this' to be an 'AbortSignal' object, but got ${
                    this === null ? "null" : typeof this
                }`,
            )
        }
        return aborted
    }
}
// 设置abort自定义事件
defineEventAttribute(AbortSignal.prototype, "abort")

...

/**
 * 创建一个AbortSinal实例，并设置aborted状态为false，存入abortedFlags中，同时绑定abort事件属性
 */
export function createAbortSignal(): AbortSignal {
    const signal = Object.create(AbortSignal.prototype)
    EventTarget.call(signal)
    abortedFlags.set(signal, false)
    return signal
}

/**
 * 设置AbortSinal实例aborted状态为true，同时触发abort监听事件回调
 */
export function abortSignal(signal: AbortSignal): void {
    if (abortedFlags.get(signal) !== false) {
        return
    }

    abortedFlags.set(signal, true)
    signal.dispatchEvent<"abort">({ type: "abort" })
}
...
```

1）AbortSignal继承EventTarget（第三方依赖包，作用是赋予实例监听自定义事件，它会帮你解决兼容性问题addEventListener/onXX/dispatchEvent）,AbortSignal自定义了abort监听事件

2）AbortSignal.aborted()：获取当前实例是否已经启动了abort监听事件。

3）abortedFlags：map类型，用于存储每个实例的是否已经启动了abort监听事件，默认为false（createAbortSignal创建实例的时候设置），调用abortSignal函数的时候会设置为true

4）createAbortSignal()：构建函数，初始化实例对象为false，绑定abort监听事件（需要用户自己设置abort监听回调事件）

5）abortSignal(instance)：设置当前实例状态为ture，同时触发abort监听回调事件

### 2-2）AbortController源码解析
```javascript
...
export default class AbortController {
    /**
     * 构造函数，创建一个AbortSignal实例并存入signals中
     */
    public constructor() {
        signals.set(this, createAbortSignal())
    }

    /**
     * 从signals中获取当前AbortSignal实例
     */
    public get signal(): AbortSignal {
        return getSignal(this)
    }

    /**
     * 先从signals中获取当前AbortSignal实例，然后设置实例aborted状态为true，触发abort监听回调事件
     */
    public abort(): void {
        abortSignal(getSignal(this))
    }
}
...
```
1）AbortController构造函数中会调用createAbortSignal创建AbortSignal实例并存入一个Map类型的signals中。

2）abort()方法会调用abortSignal函数，传入的参数就是从signals中取出来的AbortSignal实例

3）signal()方法作用是从signals中取出AbortSignal实例

### 2-3）AbortController兼容性
![](https://cdn.nlark.com/yuque/0/2020/png/1387052/1598523817565-59353f78-604f-4df6-9ee4-9e205ddb3d6a.png#align=left&display=inline&height=656&margin=%5Bobject%20Object%5D&originHeight=656&originWidth=1341&size=0&status=done&style=none&width=1341)

AbortController兼容性不是特别好，可以通过引用polyfill来处理

### 2-4）高级进阶

> 问：每次请求，都会重新创建一个AbortSignal实例吗？
> 答：是的

> 思考：signals和abortedFlags都是Map类型，每一个请求都会创建一个实例，随着时间的推移和请求的增多，如何防止缓存雪崩问题？
> 答：signals和abortedFlags准确的说是WeakMap类型，而WeakMap跟Map会有所区别，WeakMap的键只能是对象的引用，当垃圾回收机制执行时，会检测WeakMap的键是否被引用，若没有被引用，该键对会被删除，并自动回收，从而防止缓存雪崩的问题。

> 思考：AbortSignal是如何具备监听事件能力的？
> 答：它本身并不具备事件处理能力，它继承了一个EventTarget类使其具备监听处理事件能力


## 3.可中止的fetch
说了那么多原理上的东西，真正如何应用？其实，一个可中止的fetch依赖于一个signal的属性。源码如下：
```
// fetch源码
export function fetch(input, init) {
  return new Promise(function(resolve, reject) {
    var request = new Request(input, init)
    
    // 解决发起前的中断
    if (request.signal && request.signal.aborted) {
      return reject(new DOMException('Aborted', 'AbortError'))
    }
    ...
    
    var xhr = new XMLHttpRequest()

    // 触发abort
    function abortXhr() {
      xhr.abort()
    }
    ...
    
    // 处理abort事件回调，用于解决发起过程中中止事件的回调
    xhr.onabort = function() {
      setTimeout(function() {
        reject(new DOMException('Aborted', 'AbortError'))
      }, 0)
    }
    ....
    
    // 监听abort事件，监听发起过程中的abort事件
    if (request.signal) {
      request.signal.addEventListener('abort', abortXhr)

      xhr.onreadystatechange = function() {
        // DONE (success or failure)
        if (xhr.readyState === 4) {
          request.signal.removeEventListener('abort', abortXhr)
        }
      }
    }
    ....
  }
  ...
}
```
实现过程如下：
![](https://cdn.nlark.com/yuque/0/2020/png/1387052/1598523817635-f96eb4d2-cbfb-4f31-ba01-23e7fa813d7d.png#align=left&display=inline&height=741&margin=%5Bobject%20Object%5D&originHeight=741&originWidth=781&size=0&status=done&style=none&width=781)

黄色方框：属于外部触发的动作，比如0-100ms内是进行中的请求，50ms时，外部调用了controller.abort()方法，触发中断正在进行中的请求。

## 4.应用场景
**1）通过设置超时请求，判断文件/图片是否属于大文件/大图片，服务端渲染ssr可在不加载文件的情况下判断哪些小图片可优先渲染，哪些图片加载超时应使用骨架**

**2）在网络资源充足的情况下，向多个域名发起同一请求，以最快速度获取这份数据**
**...**

参考源码：

1. [EventTarget源码传送门](https://github.com/mysticatea/event-target-shim)
1. [AbortController源码传送门](https://github.com/mysticatea/abort-controller)
1. [fetch源码传送门](https://github.com/github/fetch)
