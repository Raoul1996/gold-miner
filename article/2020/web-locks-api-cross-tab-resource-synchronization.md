> * 原文地址：[Web Locks API: 跨 Tab 资源同步](https://blog.bitsrc.io/web-locks-api-cross-tab-resource-synchronization-54326e079756)
> * 原文作者：[Mahdhi Rezvi](https://medium.com/@mahdhirezvi)
> * 译文出自：[掘金翻译计划](https://github.com/xitu/gold-miner)
> * 本文永久链接：[https://github.com/xitu/gold-miner/blob/master/article/2020/web-locks-api-cross-tab-resource-synchronization.md](https://github.com/xitu/gold-miner/blob/master/article/2020/web-locks-api-cross-tab-resource-synchronization.md)
> * 译者：[Raoul1996](https://github.com/Raoul1996)
> * 校对者：

# Web Locks API: 跨 Tab 资源同步

![Image by [MasterTux](https://pixabay.com/users/mastertux-470906/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=3348307) from [Pixabay](https://pixabay.com/?utm_source=link-attribution&utm_medium=referral&utm_campaign=image&utm_content=3348307)](https://cdn-images-1.medium.com/max/3840/1*voZnUOIRnDc4kc_nfm4bEQ.jpeg)

## 什么是锁（Locks）？

随着计算机变得越来越强大，会使用多个 CPU 线程来对数据进行处理。多个线程访问单个资源的时候可能会受同步问题的困扰，也催生出了有关资源共享的新问题。

如果你对线程熟悉的话，那么你自然会知晓锁的概念。锁是一种同步方法，，可强制对线程执行访问限制，防止多个线程同时访问单个资源。还有一种锁的变体，允许多个线程同时访问单个资源，不过仍将访问限制为只读。我强烈建议你去阅读一些材料，理解操作系统中锁的概念。

![单线程和多线程 — 来自 [Dave Kurtz](https://www.cs.uic.edu/~jbell/CourseNotes/OperatingSystems/images/Chapter4/4_01_ThreadDiagram.jpg)](https://cdn-images-1.medium.com/max/2000/0*iW0a4sDyFt4hsBfQ.jpg)

## 什么是 Web Locks API？

Web Locks API 将锁整体应用于 web 应用。这个 API 允许一个脚本异步持有对资源的锁定，直到其处理完成之后再释放。当持有锁时，除一种特殊情况外，其他在同域下的脚本无法获得相同资源的锁。接下来我们就说说这个特殊情况。

#### 执行流程是什么样子的呢？

1.申请锁。
2. 在异步任务中锁定时完成工作。
3. 任务完成时候锁自动释放。
4. 允许其他脚本申请锁。

当资源上有锁时，如果处在相同的执行上下文或者其他 Tab/Work 的脚本请求相同资源的锁的时候，锁请求就会进行排队。当锁释放时候，队列中的第一个请求将被授予锁并可以访问资源。

#### 锁以及其作用域

关于 Web Locks API 的作用域可能会很令人困惑。这仅仅是一个摘要，以供你更好地理解。
The scopes on how the Web Locks API can be quite confusing. Hence here is a summary for you to understand it better.

* 根据文档，锁的作用域限制在同源。

> Tab 从 https://example.com 获得的锁对选项卡从 https://example.org 获得的锁没有影响，因为它们的不同源。

* 浏览器中单独用户配置被视为单独的用户代理，视为在作用域之外。因此，即使他们的同源，也不会共享锁管理器。
* 私有模式的浏览会话（隐身模式）被视为单独的用户代理，视为在作用域之外。因此，及时他们的同源，也不会共享锁管理器。
* 在同源且同一个上下文中的脚本视为在作用域之内，并共享锁管理器。例如，一个网页上的两个函数尝试对同一资源加锁。
* 打开在同一个用户代理的同源页面和 Workers（agents）共享锁管理器，即使他们不在相关的浏览上下文中。

假设如果 `脚本 A` 属于 `锁管理器 A`，`脚本 B` 属于 `锁管理器 B`。`脚本 A` 尝试获得 `资源 X` 的锁，成功获得锁并执行异步任务。同时，`脚本 B` 也尝试获得 `资源X` 的锁，它将会成功，因为两个脚本属于不同的锁管理器。但是如果他们属于同一个锁管理器，那么将会有一个队列来获取对 `资源 X` 的锁。希望这个点我已经说明白了。

#### 这些资源是什么？

好吧，它们代表了一种抽象资源。它只是我们想出的一个名称，指的是我们想要保留的资源。它在调度算法之外没有任何意义。

换言之，在上面的例子中，我们可以将 `资源 X` 认为成存储我数据的数据库，或者可以是 `localStorage`。


## 为什么资源协调很重要？

在简单的Web应用程序中很少需要进行资源协调。但是，更复杂，使用大量 JavaScript 的 Web 应用程序可能需要进行资源协调。

如果你使用跨多个 Tab 的应用程序并且可以执行 CRUD 操作，你将必须保持选项卡同步以避免问题。如果用户在一个 Tab 上打开了文本编辑的 Web 应用程序，而忘记了另一个 Tab 打开同一应用程序。现在，他具有正在运行的同一应用程序的两个 Tab。如果他在一个 Tab 上执行一项操作，并尝试在另一 Tab 上执行完全不同的操作，则当同一资源上被两个不同的进程操作时，服务器上可能会发生冲突。在这种情况下，建议获取对资源的锁定并进行同步。

此外，可能存在用户打开了股票投资Web应用程序的两个 Tab 的情况。 如果用户使用其中一个打开的 Tab 购买了一定数量的股票，则两个 Tab 必须保持同步，以避免出现客户错误地再次进行交易的情况。 一个简单的选择是一次只允许应用程序的一个 Tab 或窗口。 但是请注意，可以通过使用私人浏览会话来绕过这个限制。

尽管可以使用诸如 SharedWorker，BroadcastChannel，localStorage，sessionStorage，postMessage，卸载 handler 之类的 API 来管理选项卡通信和同步，但它们各自都有缺点，并且需要变通办法，这降低了代码的可维护性。 Web Locks API 试图通过引入更标准化的解决方案来简化此过程。

## 使用 Web Locks API

这个 API 使用起来比较直接了当，但是你必须要确定浏览器支持该 API。`request()` 方法经常用来请求对资源的锁定。

方法接收三个参数。

* 资源名称（必须传入的第一个参数）—— 字符串
* 回调（必须传入的最后一个参数）—— 当请求成功时候会被调用的一个回调。建议传递 `async` 回调，这样它会返回一个 Promise。及时你没有传入异步回调，它也会包进一个 Promise 中。
* 选项（回调之前传递的可选第二个参数）—— 具有特定属性的对象，我们将在稍后讨论。

这个方法返回的 promise 会在资源被获得后 resolve 掉， 你可以使用 `then..catch ..` 方法，或者选择 `await` 方法来用同步的方式写异步代码。

```JavaScript
const requestForResource1 = async() => {
 if (!navigator.locks) {
  alert("你的浏览器不支持 Web Locks API");
  return;
 }
  
try {
  const result = await navigator.locks.request('resource_1', async lock => {
   // 拿到了锁
   await doSomethingHere();
   await doSomethingElseHere();
   // 可选的返回值
   return "ok";
   // 现在锁被释放
  });
 } catch (exception) {
  console.log(exception);
 }
  
}
```
我相信上面的代码片段是不言自明的。但是还有一些其他参数可以传递给 `request()` 函数。让我们来看看它们。

## 可选参数

#### 模式

对锁发起请求时可以选用两种模式。

* 互斥 (默认)
* 共享

当在资源上请求互斥锁，且如果该资源已经持有了“互斥”或“共享”锁，锁将不会被授予。 但是，如果在持有“共享”锁的资源上请求“共享”锁，则该请求将被批准。 但是，当持有的锁是“互斥”锁时，情况就不会如此。请求将由锁管理器排队。下表总结了这一点。

![Source: [Shun Yan Cheung](http://www.mathcs.emory.edu/~cheung/Courses/554/Syllabus/7-serializability/compat-mat.html)](https://cdn-images-1.medium.com/max/2000/0*6-VW3zuoya_NHYsP.gif)

#### 信号

信号属性传入一个 [中止信号](https://dom.spec.whatwg.org/#interface-AbortSignal)。这允许一个在队列中的锁请求被中止。如果在特定时间段内未批准锁定请求，则可以使用超时来中止锁定请求。

```JavaScript
const controller = new AbortController();
setTimeout(() => controller.abort(), 400); //最多等待 400ms
try {
  await navigator.locks.request('resource1', {signal: controller.signal}, async lock => {
    await doSomethingHere();
  });
} catch (ex) {
  // |ex| 如果计时器触发，将是一个 DOMException，错误名称为 ”AbortError“。
}
```

#### 如果可用

这个属性是一个默认值为 `false` 的布尔值。如果是 true，则锁请求仅在不需要排队时才会被授予。换句话说，在没有任何其他等待的情况下，锁请求才会被授予，否则将返回 `null`。

但是请注意，当返回 `null` 时，该函数将不同步。而是回调将接收值 `null`，值可以由开发者进行处理。

```js
await navigator.locks.request('resource', {ifAvailable: true}, async lock => {
  if (!lock) {
    // 没有获得锁，自行处理
    return;
  }
  
  // 如果获得锁，走这里
});
```

## 用锁的风险

There are several risks associated with the implementation of locks. These are more or less due to the concept of locks themselves, not because of any bugs in the API.

#### 死锁

The concept of deadlock is associated with concurrency. A deadlock happens when a process can no longer make progress because each component is waiting for a request that can not be met.

![Illustration by Author](https://cdn-images-1.medium.com/max/2840/1*DQI5-XUHL9L8e-32d_ufxg.png)

In order to avoid deadlocks, it is essential that we follow a strict pattern when acquiring locks. There are several techniques used such as avoiding nesting locks, make sure locks are well ordered, or even use the signal optional parameter to timeout the lock request. But a carefully crafted API design would be the optimal solution to avoid deadlocks. You can read more about deadlock prevention over [here](https://en.wikipedia.org/wiki/Deadlock_prevention_algorithms).

#### Tabs 无响应

There can be situations where you discover one of your tabs has become unresponsive. If this happens while the tab is holding a lock, you can fall into a tricky situation. A `steal` boolean was introduced as a part of the options argument we previously spoke about. Although `false` by default, if it is passed as `true`, any held locks over a resource will be released and this new lock request will be granted immediately, regardless of the number of requests in the queue for the resource.

But keep in mind that this controversial feature is supposed to be used only in exceptional circumstances. You can read more about this feature over [here](https://github.com/WICG/web-locks/issues/23).

#### Debug 困难

Since there is a possibility of hidden states involved, there can be problems related to debugging. To fix this, a `query` method was introduced. This method returns the state of the lock manager at that specific instant. The result would contain details of pending and currently held locks. It would also contain details of the type of lock, resource in which the lock is held/requested and the clientId of the request.

The `clientId` is simply the corresponding value to the unique context (frame/worker) the lock was requested from. This is the same value used in Service Workers.

```JSON
{
    "held": [
      {
        "clientId": "da2deeaa-8bac-4d1d-97e7-6b1ee46b6730",
        "mode": "exclusive",
        "name": "resource_1"
      }
    ],
    "pending": [
        {
            "clientId": "da2deeaa-8bac-4d1d-97e7-6b1ee46b6730",
            "mode": "shared",
            "name": "resource_1"
        },
        {
            "clientId": "76384678-c5b6-452e-84f0-00c2ef65109e",
            "mode": "exclusive",
            "name": "resource_1"
        },
        {
            "clientId": "76384678-c5b6-452e-84f0-00c2ef65109e",
            "mode": "shared",
            "name": "resource_1"
        }
    ]
}
```

You must also note that this method should not be used by the application to make decisions on locks being held/requested as this output contains the state of the lock manager at that **specific instant** as I’ve mentioned before. This means that a lock could be released by the time your code makes a decision.

## 浏览器兼容性

One let down of this API is browser compatibility. Although there was a polyfill for unsupported browsers, it was later removed.

![Source: [MDN Docs](https://developer.mozilla.org/en-US/docs/Web/API/Web_Locks_API#Browser_compatibility)](https://cdn-images-1.medium.com/max/2010/1*qnGOiC-_s5tFzP0N0moG8A.png)

![Source: [MDN Docs](https://developer.mozilla.org/en-US/docs/Web/API/Web_Locks_API#Browser_compatibility)](https://cdn-images-1.medium.com/max/2006/1*N6uF2s2tQHABBv5SsESNbg.png)

The Web Locks API is a highly useful feature with several use cases that make it a very important addition. Its limited support, however, can discourage developers from learning and implementing it. But with the level of impact this API can have on modern web applications, I personally believe it is essential for web developers to know their way around this new feature. Furthermore, since this API is experimental, you can expect changes in the future.

Head over to this simple [demo](https://mahdhir.github.io/Web-Locks-API-demo/) to get a hands-on experience on how this works. You can view the [source code](https://github.com/Mahdhir/Web-Locks-API-demo) here.

**Resources**

- [MDN Docs](https://developer.mozilla.org/en-US/docs/Web/API/Web_Locks_API)
- [Web Locks Explainer](https://github.com/WICG/web-locks/blob/main/EXPLAINER.md)

> 如果发现译文存在错误或其他需要改进的地方，欢迎到 [掘金翻译计划](https://github.com/xitu/gold-miner) 对译文进行修改并 PR，也可获得相应奖励积分。文章开头的 **本文永久链接** 即为本文在 GitHub 上的 MarkDown 链接。

---

> [掘金翻译计划](https://github.com/xitu/gold-miner) 是一个翻译优质互联网技术文章的社区，文章来源为 [掘金](https://juejin.im) 上的英文分享文章。内容覆盖 [Android](https://github.com/xitu/gold-miner#android)、[iOS](https://github.com/xitu/gold-miner#ios)、[前端](https://github.com/xitu/gold-miner#前端)、[后端](https://github.com/xitu/gold-miner#后端)、[区块链](https://github.com/xitu/gold-miner#区块链)、[产品](https://github.com/xitu/gold-miner#产品)、[设计](https://github.com/xitu/gold-miner#设计)、[人工智能](https://github.com/xitu/gold-miner#人工智能)等领域，想要查看更多优质译文请持续关注 [掘金翻译计划](https://github.com/xitu/gold-miner)、[官方微博](http://weibo.com/juejinfanyi)、[知乎专栏](https://zhuanlan.zhihu.com/juejinfanyi)。
