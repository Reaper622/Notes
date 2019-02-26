# 浅析Vue 中 $nextTick 机制

## nextTick 出现的前提

因为Vue是异步驱动视图更新数据的，即当我们在事件中修改数据时，视图并不会即时的更新，而是等在同一事件循环的所有数据变化完成后，再进行视图更新。类似于Event Loop事件循环机制。

### 官方介绍

首先我们看下官网给出的介绍:

#### Vue.nextTick([callback, context])

- 参数:
  - {Function} [callback]
  - {Object} [context]

- 用法：

  在下次**DOM更新循环**结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后DOM。

```javascript
// 修改数据
vm.msg = 'Hello'
// 当我们在这里调用DOM的数据时，它其实还没有更新
Vue.nextTick(function () {
    // DOM 更新了
})

// 2.1.0新增 Promise用法
Vue.nextTick()
    .then(function () {
    // 此时DOM已经更新
})
```

>2.1.0 起新增：如果没有提供回调且在支持 Promise 的环境中，则返回一个 Promise。请注意 Vue 不自带 Promise 的 polyfill，所以如果你的目标浏览器不原生支持 Promise (IE：你们都看我干嘛)，你得自己提供 polyfill。

## DOM更新循环

首先，Vue实现响应式并不是在数据改变后就立即更新DOM，而是在一次事件循环的所有数据变化后再异步执行DOM更新.

有关异步以及事件循环，可以看下我之前写过的一篇关于文章[说说异步](https://juejin.im/post/5c0faca7e51d45710f7561e9)

如果不想去详细了解，这边我就简单总结一下事件循环:

同步代码的执行 => 查找异步队列，进入执行栈，执行Callback1[事件循环1] => 查找异步队列，进入执行栈，执行Callback2[事件循环2] => .....

>  即每个异步的Callback都会再独立形成一次事件循环

所以我们可以退出nextTick的触发时机

**一次事件循环中的代码执行完毕 => DOM更新 => 触发nextTick的回调 => 进入下一循环**

## 示例展示

> Talk is cheap, show me the code.					—— Linus Torvalds

可能只凭一些概念性的讲解还是无法对nextTick机制有很清晰的了解，还是上个示例来了解一下吧。

```vue
<template>
	<div class="app">
        <div ref="contentDiv">{{content}}</div>
        <div>在nextTick执行前获取内容:{{content1}}</div>
        <div>在nextTick执行之后获取内容:{{content2}}</div>
        <div>在nextTick执行前获取内容:{{content3}}</div>
    </div>
</template>

<script>
    export default {
        name:'App',
        data: {
            content: 'Before NextTick',
            content1: '',
            content2: '',
            content3: ''
        },
        methods: {
            changeContent () {
                this.content = 'After NextTick' // 在此处更新content的数据
                this.content1 = this.$refs.contentDiv.innerHTML //获取DOM中的数据
                this.$nextTick(() => {
                    // 在nextTick的回调中获取DOM中的数据
                    this.content2 = this.$refs.contentDiv.innerHTML 
                })
                this.content3 = this.$refs.contentDiv.innerHTML
            }
        },
        mount () {
            this.changeContent()
        }
    }
</script>
```

当我们打开页面后我们可以发现结果为:

```
After NextTick

在nextTick执行前获取内容:Before NextTick

在nextTick执行之后获取内容:After NextTick

在nextTick执行前获取内容:Before NextTick
```

所以我们可以知道,虽然`content1`和`content3`获得内容的语句是写在`content`数据改变语句之后的，但他们属于同一个事件循环中，所以`content1`和`content3`获取的还是 'Before NextTick' ,而`content2 `获得内容的语句写在nextTick的回调中，在DOM更新之后再执行，所以获取的是更新后的 'After NextTick'。

## 应用场景

下面是一些nextTick的主要应用场景

### 在created 生命周期执行DOM操作

当在`created()`生命周期中直接执行DOM操作是不可取的，因为此时的DOM并未进行任何的渲染。所以解决办法是将DOM操作写进`Vue.nextTick()`的回调函数中。或者是将操作放入`mounted()`钩子函数中

### 在数据变化后需要进行基于DOM结构的操作

在我们更新数据后，如果还有操作要根据更新数据后的DOM结构进行，那么我们应当将这部分操作放入**Vue.nextTick()**回调函数中

这部分的详细原因在Vue的官方文档中解释的非常清晰:

>可能你还没有注意到，Vue **异步**执行 DOM 更新。只要观察到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据改变。如果同一个 watcher 被多次触发，只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作上非常重要。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。Vue 在内部尝试对异步队列使用原生的 `Promise.then` 和 `MessageChannel`，如果执行环境不支持，会采用 `setTimeout(fn, 0)` 代替。
>
>例如，当你设置 `vm.someData = 'new value'` ，该组件不会立即重新渲染。当刷新队列时，组件会在事件循环队列清空时的下一个“tick”更新。多数情况我们不需要关心这个过程，但是如果你想在 DOM 状态更新后做点什么，这就可能会有些棘手。虽然 Vue.js 通常鼓励开发人员沿着“数据驱动”的方式思考，避免直接接触 DOM，但是有时我们确实要这么做。为了在数据变化之后等待 Vue 完成更新 DOM ，可以在数据变化之后立即使用 `Vue.nextTick(callback)` 。这样回调函数在 DOM 更新完成后就会调用。



## 附：nextTick源码解析

> 个人翻译，若有不妥请随时提出

```javascript
export const nextTick = (function () {
  // 存放所有的回调函数
  const callbacks = []
  // 是否正在执行回调函数的标志
  let pending = false
  // 触发执行回调函数
  let timerFunc
// 处理回调函数
  function nextTickHandler () {
    pending = false
    const copies = callbacks.slice(0)
    callbacks.length = 0
    for (let i = 0; i < copies.length; i++) {
      // 执行回调函数
      copies[i]()
    }
  }
  
  // nextTick行为利用了微任务队列
  // 它可以通过原生Promise或者MutationObserver实现
  // MutationObserver已经有了广泛的浏览器支持，然而他仍然在UIWebView在ios系统9.3.3以上的
  // 系统有严重的Bug，问题发生在我们触摸事件的触发时。
  // 它会在我们触发一段时间后完全停止，所以原生Promise是有效可以利用的，我们会使用它:
  /* istanbul ignore if */
  if (typeof Promise !== 'undefined' && isNative(Promise)) {
    var p = Promise.resolve()
    var logError = err => { console.error(err) }
    timerFunc = () => {
      p.then(nextTickHandler).catch(logError)
      // 在有问题的 UIWebViews 中，Promise.then 方法不会完全的停止，但它可能会在一个
      // 奇怪的状态卡住当我们把回调函数推入一个微任务队列但是这个队列并不是在冲洗中，知道
      // 浏览器需要做一些其他的任务时，例如：执行一个定时函数。因此我们可以"强制"微任务队
      // 列被冲洗通过加入一个空的定时函数
      if (isIOS) setTimeout(noop)
    }
  } else if (!isIE && typeof MutationObserver !== 'undefined' && (
    isNative(MutationObserver) ||
    // PhantomJS and iOS 7.x
    MutationObserver.toString() === '[object MutationObserverConstructor]'
  )) {
    // 使用MutationObserver当Promise不可用时，
    // 例如 PhantomJS, iOS7, Android 4.4
    var counter = 1
    var observer = new MutationObserver(nextTickHandler)
    var textNode = document.createTextNode(String(counter))
    observer.observe(textNode, {
      characterData: true
    })
    timerFunc = () => {
      counter = (counter + 1) % 2
      textNode.data = String(counter)
    }
  } else {
    // 当MutationObserver 和 Promise都不可以使用时
    // 我们使用setTimeOut来实现
    /* istanbul ignore next */
    timerFunc = () => {
      setTimeout(nextTickHandler, 0)
    }
  }

  return function queueNextTick (cb?: Function, ctx?: Object) {
    let _resolve
    callbacks.push(() => {
      if (cb) {
        try {
          cb.call(ctx)
        } catch (e) {
          handleError(e, ctx, 'nextTick')
        }
      } else if (_resolve) {
        _resolve(ctx)
      }
    })
    if (!pending) {
      pending = true
      timerFunc()
    }
    if (!cb && typeof Promise !== 'undefined') {
      return new Promise((resolve, reject) => {
        _resolve = resolve
      })
    }
  }
})()
```

我们通过源码可以知道，`timeFunc`这个函数起延迟执行的作用，它有三种实现方式

- Promise
- MutationObserver
- setTimeout

其中`Promise`和 `setTimeout` 我们都不陌生，下面重点介绍一下`MutationObserver`

`MutationObserver`是HTML5中的新API，是个用来监视DOM变动的接口。他能监听一个DOM对象上发生的子节点删除、属性修改、文本内容修改等等。
调用过程很简单，但是有点不太寻常：你需要先给他绑回调：

```javascript
let mo = new MutationObserver(callback)
```

通过给`MutationObserver`的构造函数传入一个回调，能得到一个`MutationObserver`实例，这个回调就会在`MutationObserver`实例监听到变动时触发。

这个时候你只是给`MutationObserver`实例绑定好了回调，他具体监听哪个DOM、监听节点删除还是监听属性修改，还没有设置。而调用他的`observer`方法就可以完成这一步:

```javascript
var domTarget = 你想要监听的dom节点
mo.observe(domTarget, {
      characterData: true //说明监听文本内容的修改。
})
```

在`nextTick`中 `MutationObserver`的作用就如下图所示。在监听到DOM更新后，调用回调函数。

![MutationObserver作用](https://upload-images.jianshu.io/upload_images/3985563-3d5f3707101ce1a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/432/format/webp)

## 总结

- 在同一事件循环中，当所有的同步数据更新执行完毕后，才会调用nextTick
- 在同步执行环境中的数据完全更新完毕后，DOM才会开始渲染。
- 在同一个事件循环中，若存在多个nextTick，将会按最初的执行顺序进行调用。
- 每个异步的回调函数执行后都会存在一个独立的事件循环中，对应自己独立的nextTick
- vue DOM的视图更新实现，，使用到了ES6的`Promise`及HTML5的`MutationObserver`，当环境不支持时，使用`setTimeout(fn, 0)`替代。上述的三种方法，均为异步API。其中`MutationObserver`类似事件，又有所区别；事件是同步触发，其为异步触发，即DOM发生变化之后，不会立刻触发，等当前所有的DOM操作都结束后触发。



#### 参考链接

[Vue官方文档-异步更新队列](https://cn.vuejs.org/v2/guide/reactivity.html#%E5%BC%82%E6%AD%A5%E6%9B%B4%E6%96%B0%E9%98%9F%E5%88%97)

[Ruheng:简单理解Vue中的nextTick](简单理解Vue中的nextTick)

> 个人Github：(Reaper622)[https://github.com/Reaper622]
>
> 欢迎学习交流