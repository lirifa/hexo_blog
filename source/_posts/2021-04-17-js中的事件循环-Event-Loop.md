---
title: js中的事件循环(Event Loop)
date: 2021-04-17 15:01:38
tags: [前端, js]
categories: 技术
---

事件循环是JavaScript处理异步操作的机制，是浏览器渲染主线程的工作方式。
<!-- more -->

# 一、在了解js的事件循环之前，我们先了解一下浏览器有哪些进程线程：
首先要明确一点：**浏览器是一个多进程多线程的应用程序**。 浏览器内部工作极其复杂。为了避免相互影响，为了减少连环崩溃的几率，当启动浏览器后，它会自动启动多个进程，每个进程又包含有多个线程。
[![pZEfpuD.png](https://s41.ax1x.com/2025/11/28/pZEfpuD.png)](https://imgchr.com/i/pZEfpuD) 
> 可以在浏览器的任务管理器中查看当前的所有进程 其中，最主要的进程有：
1. 浏览器进程
- **功能：**主要负责用户界面的显示和用户交互，包括地址栏、书签、标签页等。
- 线程：浏览器进程内部通常会有多个线程，例如：
    + **UI线程：**负责处理用户的输入和界面更新。
    + **定时器线程：**处理定时器事件。
    + **文件I/O线程：**处理文件读写操作。

2. 网络进程
- **功能：**专门负责网络请求和资源加载，包括从服务器获取HTML、CSS、JavaScript、图片等资源。通过将网络操作与渲染分离，可以提高浏览器的响应速度。
- 线程：网络进程内部也会启动多个线程来处理不同的网络任务，例如：
    + **请求线程：**负责发送HTTP请求和接收响应。
    + **缓存线程：**处理缓存的读写操作，以提高资源加载速度

3. 渲染进程（核心）
- **功能：**渲染进程是浏览器的核心部分，负责将网页内容呈现给用户。每个标签页通常会有一个独立的渲染进程，以确保标签页之间的隔离，避免一个标签页的崩溃影响到其他标签页。
- 线程：
    + **渲染主线程：**负责执行HTML、CSS和JavaScript代码，进行DOM树的构建、样式计算、布局、绘制等。
    + **合成线程：**负责将渲染结果合成到屏幕上，处理图层的合成和动画效果。
    + **工作线程：**用于处理一些异步任务，如Web Worker。

# 二、渲染主线程是如何工作的
## 渲染主线程是浏览器中最繁忙的线程，需要它处理的任务包括但不限于：
- 解析 HTML
- 解析 CSS
- 计算样式
- 布局
- 处理图层
- 每秒把页面画 60 次
- 执行全局 JS 代码
- 执行事件处理函数
- 执行计时器的回调函数
- ...
> 关于渲染进程为什么不使用多线程来处理这么多任务，js为什么被设计为单线程
1. 简化执行环境与开发模型
   单线程模型避免了多线程编程中常见的‌死锁、‌竞态条件等复杂同步问题，降低了学习门槛和运行时错误风险。代码顺序执行特性使开发者更容易预测程序行为，提升开发效率和稳定性

2. 保障DOM操作一致性
    作为浏览器脚本语言，JavaScript需频繁操作‌DOM。若采用多线程，可能出现线程A添加内容与线程B删除同一节点的冲突，导致浏览器无法确定最终状态。单线程通过顺序执行DOM操作，确保状态一致性。‌‌

3. 历史设计与异步处理优化
   JavaScript诞生之初旨在实现页面动态交互，单线程设计简化了语言实现。通过‌事件循环机制，耗时操作（如网络请求）被放入任务队列，主线程空闲时处理，避免阻塞用户界面。现代异步方案（如‌Promise、‌async/await）进一步提升了可读性。‌‌

为了利用多核CPU的计算能力，HTML5提出Web Worker标准，允许JavaScript脚本创建多个线程，但是子线程完全受主线程控制，且不得操作DOM。所以，这个新标准并没有改变JavaScript单线程的本质。

## 要处理这么多的任务，主线程遇到了一个前所未有的难题：如何调度任务？ 比如：
- 我正在执行一个 JS 函数，执行到一半的时候用户点击了按钮，我该立即去执行点击事件的处理函数吗？
- 我正在执行一个 JS 函数，执行到一半的时候某个计时器到达了时间，我该立即去执行它的回调吗？
- 浏览器进程通知我“用户点击了按钮”，与此同时，某个计时器也到达了时间，我应该处理哪一个呢？
- ......
渲染主线程想出了一个绝妙的主意来处理这个问题：排队。
[![pZE4EX8.png](https://s41.ax1x.com/2025/11/28/pZE4EX8.png)](https://imgchr.com/i/pZE4EX8)
1. 在最开始的时候，渲染主线程会进入一个无限循环
2. 每一次循环会检查消息队列中是否有任务存在。如果有，就取出第一个任务执行，执行完一个后进入下一次循环；如果没有，则进入休眠状态。
3. 其他所有线程（包括其他进程的线程）可以随时向消息队列添加任务。新任务会加到消息队列的末尾。在添加新任务时，如果主线程是休眠状态，则会将其唤醒以继续循环拿取任务，这样一来，就可以让每个任务有条不紊的、持续的进行下去了。
**整个过程，被称之为事件循环（消息循环）**

# 三、何为异步
代码在执行过程中，会遇到一些无法立即处理的任务，比如：
- 计时完成后需要执行的任务 —— <font color="#dd0000">setTimeout</font>、<font color="#dd0000">setInterval</font>
- 网络通信完成后需要执行的任务 – <font color="#dd0000">XHR</font>、<font color="#dd0000">Fetch</font>
- 用户操作后需要执行的任务 – <font color="#dd0000">addEventListener</font>

如果让渲染主线程等待这些任务的时机达到，就会导致主线程长期处于「阻塞」的状态，从而导致浏览器「卡死」
[![pZE7MQI.png](https://s41.ax1x.com/2025/11/29/pZE7MQI.png)](https://imgchr.com/i/pZE7MQI)
**渲染主线程承担着极其重要的工作，无论如何都不能阻塞！** 因此，浏览器选择**异步**来解决这个问题
[![pZE7lOP.png](https://s41.ax1x.com/2025/11/29/pZE7lOP.png)](https://imgchr.com/i/pZE7lOP)
使用异步的方式，**渲染主线程永不阻塞**

# 四、任务有优先级吗
我们都知道事件循环的过程中包含宏任务和微任务的说法，经常讲，事件循环往往从宏任务开始，但是在执行下一个宏任务前，我们需要先将本轮宏任务产生的微任务执行完毕。 那么任务有优先级吗？答案是**没有**。 **但是任务队列有**。 根据 W3C 的最新解释:
> 每个任务都有一个任务类型，同一个类型的任务必须在一个队列，不同类型的任务可以分属于不同的队列。 在一次事件循环中，浏览器可以根据实际情况从不同的队列中取出任务执行。
> 浏览器必须准备好一个微队列，微队列中的任务优先所有其他任务执行 <https://html.spec.whatwg.org/multipage/webappapis.html#perform-a-microtask-checkpoint>

# 五、宏任务队列已经无法满足当前的浏览器要求了
在目前 chrome 的实现中，至少包含了下面的队列：
- 延时队列：用于存放计时器到达后的回调任务，优先级「中」
- 交互队列：用于存放用户操作后产生的事件处理任务，优先级「高」
- 微队列：用户存放需要最快执行的任务，优先级「最高」
- .....

以下是模拟浏览器任务调度的伪代码(由豆包生成)，重点体现了三种队列的优先级关系和处理流程：
```js
// 模拟浏览器的三种任务队列
const queues = {
  microtasks: [],         // 微任务队列（最高优先级）
  inputQueue: [],         // 交互队列（用户输入等）
  timerQueue: [],         // 延时队列（setTimeout/setInterval）
  renderingQueue: []      // 渲染队列（额外补充，用于完整模拟）
};

// 任务调度器状态
let isProcessing = false;

// 模拟浏览器的任务处理主循环
function browserMainLoop() {
  // 持续运行的事件循环
  while (true) {
    // 1. 先处理所有微任务（最高优先级）
    processAllMicrotasks();
    
    // 2. 检查是否需要渲染（通常在微任务后考虑）
    if (shouldRender()) {
      processRenderingTasks();
    }
    
    // 3. 处理高优先级任务：交互队列（用户输入优先于定时器）
    if (queues.inputQueue.length > 0) {
      processNextTask(queues.inputQueue);
      continue;
    }
    
    // 4. 处理延时队列任务（优先级低于交互）
    if (queues.timerQueue.length > 0) {
      // 只处理已到期的定时器任务
      const now = getCurrentTime();
      const readyTimers = queues.timerQueue.filter(task => task.expires <= now);
      if (readyTimers.length > 0) {
        // 按到期时间排序，先处理最早到期的
        readyTimers.sort((a, b) => a.expires - b.expires);
        processNextTask(readyTimers);
        continue;
      }
    }
    
    // 5. 若没有任务，进入休眠等待新任务
    waitForNewTasks();
  }
}

// 处理所有微任务（执行到队列为空）
function processAllMicrotasks() {
  while (queues.microtasks.length > 0) {
    const microtask = queues.microtasks.shift();
    executeTask(microtask);
  }
}

// 处理单个任务队列中的下一个任务
function processNextTask(queue) {
  if (queue.length === 0) return;
  
  isProcessing = true;
  const task = queue.shift();
  executeTask(task);
  isProcessing = false;
  
  // 执行完一个任务后，再次检查微任务（微任务会在当前任务后立即执行）
  processAllMicrotasks();
}

// 执行任务的具体逻辑
function executeTask(task) {
  try {
    task.callback();  // 执行任务的回调函数
  } catch (error) {
    reportError(error);  // 处理任务执行中的错误
  }
}

// 辅助函数：检查是否需要渲染
function shouldRender() {
  // 简化逻辑：根据浏览器刷新频率（如60Hz约16ms一次）判断是否需要渲染
  return getCurrentTime() - lastRenderTime > 16;
}

// 模拟添加任务的API（对应浏览器提供的API）
const browser = {
  // 添加微任务（如Promise.then）
  queueMicrotask(callback) {
    queues.microtasks.push({ callback });
  },
  
  // 添加延时任务（如setTimeout）
  setTimeout(callback, delay) {
    const expires = getCurrentTime() + delay;
    queues.timerQueue.push({ callback, expires });
  },
  
  // 添加交互任务（如click事件）
  addInputTask(callback) {
    queues.inputQueue.push({ callback });
  },
  
  // 添加渲染任务
  requestAnimationFrame(callback) {
    queues.renderingQueue.push({ callback });
  }
};
```

# 六、核心优先级规则说明：
1. **微任务队列（microtasks）优先级最高** - 无论其他队列是否有任务，当前执行栈空闲时会先清空所有微任务 - 对应API：<font color="#dd0000">Promise.then、queueMicrotask、async/await</font>等
2. **交互队列（inputQueue）次之** - 用户输入（点击、键盘等）任务优先级高于定时器，保证用户操作的响应速度 - 浏览器会优先处理用户交互，避免界面卡顿感
3. **延时队列（timerQueue）优先级较低** - <font color="#dd0000">setTimeout/setInterval</font>任务仅在没有交互任务时才会被处理 - 定时器的实际执行时间可能比设定时间晚（受队列阻塞影响）
4. **渲染任务（renderingQueue）适时执行** - 通常在微任务处理后、其他任务执行前检查是否需要渲染 - 遵循显示器刷新率（如60fps），避免过度渲染消耗性能

这个伪代码简化了浏览器的实际实现，但核心逻辑符合现代浏览器（包括Chrome）的任务调度原则：**优先保证用户交互响应速度，其次处理定时任务，而微任务则始终在当前任务周期内立即完成。**

实际浏览器中，任务调度会更复杂，还会涉及任务优先级动态调整、线程池管理、节能策略等，但上述伪代码已能体现三种队列的核心优先级关系。

# 总结
本文主要从浏览器的渲染进程的视角出发，为大家讲解当前**浏览器环境下**的事件循环是什么样的，如果正在阅读本文的你之前并不了解什么是事件循环，我这里推荐你阅读[这篇文章](https://zhuanlan.zhihu.com/p/9441045906)，我相信读完以后，你一定能对事件循环有一定程度的了解。

# 面试题：说说事件循环机制
1. 先说基本知识点，宏任务、微任务有哪些
2. 说事件循环机制过程，边说边画图出来
3. 说async/await执行顺序注意，可以把 chrome 的优化，做法其实是违法了规范的，V8 团队的PR这些自信点说出来，显得你很好学，理解得很详细，很透彻。
4. 把node的事件循环也说一下，重复1、2、3点，node中的第3点要说的是node11前后的事件循环变动点。
[答案详情](https://juejin.cn/post/6844904079353708557?searchId=202511282107300C1DA29DEDCF4A231E8F)