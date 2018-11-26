## 以用户为中心的性能指标

传统性能指标的问题：

* 性能指标确保精度，加载时间因用户而异，具体取决于其设备功能和网络条件。将加载时间显示为单个数字会忽略经历更长时间加载的用户。
加载时间应该是来自每个用户的所有加载时间的集合，并且完全表示该加载时间的唯一方法是使用柱状分布图。x轴的数字显示加载时间，y轴上的条形高度显示在该特定时间段中经历加载时间的用户的相对数量。

* 加载不是一个单一的时刻，不是任何一个指标就可以完全指明。在加载期间有多个时刻可以影响用户是否将其视为“快速”。因此，不应仅使用一个指标来衡量加载，而应测量整个体验中可能会影响用户的加载感知的每个时刻的时间。

* 性能只是加载时间的一个关注点。但现实是性能不佳可能随时发生，而不仅仅是在加载过程中。对点击或点击无法快速响应的应用以及无法顺畅滚动或动画制作的应用可能与加载缓慢的应用一样糟糕。

所以必须专注于与用户体验有任何关系的事物。传统的性能指标（如加载时间或DOMContentLoaded时间）非常不可靠，因为它们发生时可能会或可能不会与用户认为应用程序加载的时间相对应。

因此，为了确保我们不重复这些错误，我们必须回答以下问题：

1.什么指标最准确地衡量人类所感知的性能？
2.我们如何衡量实际用户的这些指标？
3.我们如何解释我们的测量结果以确定应用程序是否“快速”？
4.一旦我们了解了应用程序的真实用户性能，我们将如何防止回归并希望在未来提高性能？

### 性能指标

当用户导航到网页时，他们通常会寻找视觉反馈来向他们保证一切都会按预期工作。

|用户体验问题|明细|指标|
|---|---|---|---|
|它发生了吗？|导航是否成功启动？ 服务器有响应吗？|First Paint (FP) / First Contentful Paint (FCP)|
|它有用吗？|是否有足够的内容呈现给用户可以使用它？|First Meaningful Paint (FMP) / Hero Element Timing|
|可以使用吗？|用户可以与页面进行交互，还是仍在忙于加载？|Time to Interactive (TTI)|
|令人愉快吗？|相互作用是否平滑自然，没有滞后和缺陷？|Long Tasks (technically the absence of long tasks)|

#### First paint(FP) 和 first contentful paint(FCP)

`Paint Timing API`定义了两个指标：FP和FCP。在导航后，当浏览器将像素渲染到屏幕时，立即标记指标点。它回答了这个问题：它是否正在发生？

这两个指标之间的主要区别是FP标记浏览器渲染任何与导航前屏幕上的内容不同的点。相比之下，FCP是浏览器从DOM渲染第一部分内容时的点。

#### First meaningful paint(FMP) 和 hero element timing

FMP是回答了问题：它有用吗？。虽然“有用”的概念很难以适用于所有网页的方式进行规范，但web开发人员自己很容易知道他们的网页的哪些部分将会是对用户最有用。

网页的这些“最重要的部分”通常被称为hero elements。网页几乎总是有比其他部分更重要的部分。 如果页面的最重要部分加载很快，用户甚至可能不会注意到页面的其余部分是否没有。

#### 长耗时任务

浏览器通过将任务添加到主线程上的队列，逐个执行以响应用户输入。 这也是浏览器执行应用程序JavaScript的地方，因此从这个意义上讲，浏览器是单线程的。

在某些情况下，这些任务可能需要很长时间才能运行，如果发生这种情况，主线程将被阻塞，队列中的所有其他任务都必须等待。这是当今网络上不良体验的主要来源。

长任务API将任何超过50毫秒的任务识别为可能存在问题，并将这些任务公开给应用程序开发人员。选择50毫秒是因为应用程序需要满足RAIL在100毫秒内响应用户输入的指导原则。

#### Time to interactive (TTI)

TTI标记应用程序可视化呈现的点，并且能够可靠地响应用户输入。应用程序可能无法响应用户输入的几个原因：

* 尚未加载使页面上的组件依赖的JavaScript。
* 阻塞主线程的任务耗时很长（如上一节所述）。

TTI标识页面的初始JavaScript加载完毕和主线程空闲（没有长任务）的点。

### 在真实用户的设备上衡量这些指标

使用`PerformanceObserver`, `PerformanceEntry`, 和 `DOMHighResTimeStamp`在真实设备上测量这些指标。

```js
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    // entry是PerformanceEntry实例
    console.log(entry.entryType);
    console.log(entry.startTime); // DOMHighResTimeStamp
    console.log(entry.duration); // DOMHighResTimeStamp
  }
});

// 观察关心的entry
observer.observe({entryTypes: ['resource', 'paint']});
```

PerformanceObserver为我们提供了前所未有的能力，即在性能事件发生时订阅它们并以异步方式响应它们。 这将取代旧的PerformanceTiming接口，该接口通常需要轮询以查看数据何时可用。

#### 跟踪FP / FCP

获得特定性能事件的数据后，发送到用于捕获当前用户的度量标准的任何分析服务。 例如，使用Google Analytics（分析），踪首次绘制时间，如下所示：

```html
<head>
  <!-- 先异步加载async Google Analytics代码段 -->
  <script>
  window.ga=window.ga||function(){(ga.q=ga.q||[]).push(arguments)};ga.l=+new Date;
  ga('create', 'UA-XXXXX-Y', 'auto');
  ga('send', 'pageview');
  </script>
  <script async src='https://www.google-analytics.com/analytics.js'></script>

  <!-- 注册PerformanceObserver跟踪绘制计时 -->
  <script>
  const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
      // `name` 可能是 'first-paint' 或 'first-contentful-paint'
      const metricName = entry.name;
      const time = Math.round(entry.startTime + entry.duration);

      ga('send', 'event', {
        eventCategory: 'Performance Metrics',
        eventAction: metricName,
        eventValue: time,
        nonInteraction: true,
      });
    }
  });
  observer.observe({entryTypes: ['paint']});
  </script>

  <link rel="stylesheet" href="...">
</head>
```

#### 使用hero elements跟踪FMP

一旦确定网页上的哪些元素是hero elements，就会想要跟踪用户可以看到的点。

目前还没有FMP的标准化定义（因此也没有performance entry type）。这是因为以通用方式确定所有页面的“有意义”非常困难。

但是，在单个页面或单个应用程序的上下文中，通常最好将FMP视为hero elements在屏幕上可见的时刻。

#### 跟踪TTI

Google提供用于[检测TTI的polyfill](https://github.com/GoogleChromeLabs/tti-polyfill)，可在任何支持Long Tasks API的浏览器中运行。

polyfill提供getFirstConsternallyInteractive()方法，该方法返回使用TTI值解析的promise。可以使用Google Analytics跟踪TTI，如下所示：

```js
import ttiPolyfill from './path/to/tti-polyfill.js';

ttiPolyfill.getFirstConsistentlyInteractive().then((tti) => {
  ga('send', 'event', {
    eventCategory: 'Performance Metrics',
    eventAction: 'TTI',
    eventValue: tti,
    nonInteraction: true,
  });
});
```

注意：与FMP一样，很难指定适用于所有网页的TTI度量标准定义。 在polyfill中实现的版本适用于大多数应用程序，但可能不适用于特定应用程序。 

#### 跟踪长任务

创建PerformanceObserver并观察longtask类型的数据：

```js
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    ga('send', 'event', {
      eventCategory: 'Performance Metrics',
      eventAction: 'longtask',
      eventValue: Math.round(entry.startTime + entry.duration),
      eventLabel: JSON.stringify(entry.attribution),
    });
  }
});

observer.observe({entryTypes: ['longtask']});
```

#### 跟踪输入延迟

将事件的时间戳与当前时间进行比较以检测代码中的输入延迟，如果差异大于100 ms，上报。

```js
const subscribeBtn = document.querySelector('#subscribe');

subscribeBtn.addEventListener('click', (event) => {
  // Event listener logic goes here...

  const lag = performance.now() - event.timeStamp;
  if (lag > 100) {
    ga('send', 'event', {
      eventCategory: 'Performance Metric'
      eventAction: 'input-latency',
      eventLabel: '#subscribe:click',
      eventValue: Math.round(lag),
      nonInteraction: true,
    });
  }
});
```

由于事件延迟通常是长任务的结果，因此可以将事件延迟检测逻辑与长任务检测逻辑结合使用：如果长任务在event.timeStamp的同时阻塞主线程，则可以上报该长任务数据。

### 
