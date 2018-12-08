## 评估加载性能

浏览器提供2个API(网络和资源计时)通过测量资源加载的各个阶段来捕获高分辨率时序，用于测量文档和资源为用户加载的速度。

### API

* 导航计时：收集HTML文档的性能指标
* 资源计时：收集文档相关资源(像样式表，脚本，图像等)的性能指标

这些方法存在于window.performance命名空间中。

#### getEntriesByType

`getEntriesByType`方法根据类型访问存储在缓冲区的性能数据：

```js
performance.getEntriesByType("navigation");
performance.getEntriesByType("resource");
```

#### getEntriesByName

`getEntriesByName`按名称获取性能数据。参数传入文档或资源的URL：

```js
var heroImageTime = performance.getEntriesByName("https://somesite.com/images/hero-image.jpg");
```

#### getEntries

`getEntries`默认情况下获取性能数据缓冲区中的所有内容：

```js
var allTheTimings = performance.getEntries();
但等等，还有更多！ getEntries还可以作为getEntriesByName和getEntriesByType的更详细的替代：

var allTheTimings = performance.getEntries({
  //按名称获取
  "name"："https：//somesite.com/images/hero-image.jpg"，
  //按类型获取
  "entryType"："资源"，
  //按其initiatorType值获取
  "initiatorType"："img"
});
```

### 网络请求各个阶段计时

#### DNS查找

Domain Name System(DNS)用于将用户请求的URL转换为IP地址，根据DNS缓存等因素，衡量时间。导航和资源计时都暴露了两个与DNS相关的指标：

* `domainLookupStart`标记DNS查找何时开始。
* `domainLookupEnd`标记DNS查找结束时。

持续时间计算：

```js
var pageNav = performance.getEntriesByType("navigation")[0];
var dnsTime = pageNav.domainLookupEnd - pageNav.domainLookupStart;
```

注意：不能总是依赖这些指标计算。在某些条件下，两个API中的某些属性将为0。例如，如果该主机未设置正确的Timing-Allow-Origin响应头，则domainLookupStart和domainLookupEnd(以及其他)对于由第三方服务的资源都可以为0。

#### 连接协商

当建立与服务器的连接时，在将资源发送到客户端之前，客户端和服务器会对事务进行排序，从而发生延迟。如果使用HTTPS(这种情况越来越常见)，此过程还包括TLS协商时间。连接阶段包含三个指标：

* `connectStart`：标记客户端打开与服务器的连接。
* `secureConnectionStart`：标记客户端何时开始TLS协商。
* `connectEnd`：在连接协商结束时标记(包括TLS时间)。

在未使用HTTPS(或HTTP连接仍然存在)的情况下，`secureConnectionStart`为0。

```js
var pageNav = performance.getEntriesByType("navigation")[0];
var connectionTime = pageNav.connectEnd - pageNav.connectStart;
var tlsTime = 0; 

if(pageNav.secureConnectionStart> 0){
  tlsTime = pageNav.connectEnd - pageNav.secureConnectionStart;
}
```

#### 请求和响应

影响页面速度的两个因素：

* 外在因素：不受的控制的因素，例如连接延迟和带宽。
* 内在因素：可以控制的内容，例如服务器和客户端架构以及资源大小。

导航和资源计时使用这些指标描述请求和响应：

* `fetchStart`：标记浏览器开始获取资源的时刻。与请求的不同之处在于，它不标记浏览器何时为资源发出网络请求，而是在它开始检查缓存(例如，HTTP和service worker缓存)以查看是否有必要发送网络请求时。
* `workerStart`：标记何时从service worker的FetchEvent获取请求(如果适用)。如果未为当前页面安装service worker，则始终为0。
* `requestStart`：浏览器发出网络请求的时间。
* `responseStart`：响应的第一个字节到达的时间。
* `responseEnd`：响应的最后一个字节到达的时间。

在保持缓存查找时间的同时测量资源下载时间：

```js
//缓存查找加响应时间
var pageNav = performance.getEntriesByType("navigation")[0];
var fetchTime = pageNav.responseEnd - pageNav.fetchStart;

//服务工作者时间加上响应时间
var workerTime = 0;

if(pageNav.workerStart> 0){
  workerTime = pageNav.responseEnd - pageNav.workerStart;
}

//请求加响应时间(仅限网络)
var totalTime = pageNav.responseEnd - pageNav.requestStart;

//仅响应时间(下载)
var downloadTime = pageNav.responseEnd - pageNav.responseStart;

//第一个字节的时间(TTFB)
var ttfb = pageNav.responseStart - pageNav.requestStart;
```

#### 相对不那么重要的指标

##### 卸载

卸载是指浏览器在加载新页面之前进行一些清除处理,包括`unloadEventStart`和`unloadEventEnd`指标。如果在卸载事件处理程序中处理下一页的渲染，则需要量化指标。

注意：卸载指标是Navigation Timing独有的。

##### 重定向

重定向会增加请求的延迟，包括`redirectStart`和`redirectEnd`指标。

##### 文档处理

加载HTML文档时，浏览器需要时间来处理它们。导航计时API通过`domInteractive`，`domContentLoadedEventStart`，`domContentLoadedEventEnd`和`domComplete`指标衡量。

* `domInteractive`：user agent将当前文档从准备就绪设置为交互式之前的时间戳。
* `domContentLoadedEventStart`：user agent在当前文档中触发DOMContentLoaded事件之前的时间戳。
* `domContentLoadedEventEnd`：user agent在当前文档中DOMContentLoaded事件完成之前的时间戳。
* `domComplete`：user agent将当前文档从准备就绪设置为完成之前的时间戳。

注意：文档处理指标仅适用于导航计时。

##### 载入中

当文档及其资源完全加载完毕后，浏览器将触发`load`事件。

 `loadEventStart`和`loadEventEnd`指标度量`load`事件，但是最简单的方法是使用duration属性。

注意：loadEventStart和loadEventEnd指标是Navigation Timing独有的。

##### 文档和资源大小

文档或资源的大小无疑会对加载性能产生影响。幸运的是，这两个API都公开了用于量化资源有效负载的属性：

* `transferSize`是包含HTTP标头的资源的总大小。
* `encodedBodySize`是除HTTP标头之外的资源的压缩大小。
* `decodingBodySize`是资源的解压缩大小(同样，不包括HTTP头)。

计算响应HTTP标头和压缩率：

```js
// HTTP标头大小
var pageNav = performance.getEntriesByType("navigation")[0];
var headerSize = pageNav.transferSize - pageNav.encodedBodySize;

// 压缩率
var compressionRatio = pageNav.decodedBodySize / pageNav.encodedBodySize;
```

### 编码获取性能计时数据

#### 使用API

使用上述API获取性能计时数据。

#### 使用`PerformanceObserver`监听性能数据

导航和资源计时API返回数组，使用循环处理存在问题，原因如下：

* 循环(尤其是包含许多项的数组)占用主线程。
* 循环仅捕获循环运行时可用的性能数据。通过计时器定期轮询性能输入缓冲区是昂贵的，并且与渲染器竞争可能导致问题。

`PerformanceObserver`就是为了解决这些问题，使用观察者模式执行在记录新性能数据时的回调：

```js
//实例化性能观察器
var perfObserver = new PerformanceObserver(function(list，obj){
  //获取到目前为止收集的所有资源数据
  //(你也可以在这里使用getEntriesByType / getEntriesByName)
  var entries = list.getEntries();

  //迭代数据
  for(var i = 0; i <entries.length; i ++){
    //做的工作！
  }
});

//运行观察者
perfObserver.observe({
  //轮询导航和资源计时数据
  entryTypes：[“navigation”，“resource”]
});
```

Internet Explorer和Edge都不支持PerformanceObserver，因此要进行功能检查：

```js
if ("performance" in window) {
  if ("PerformanceObserver" in window) {
  } else {
  }
}
```

#### 陷阱

使用计时可能存在其他棘手的情况。

##### Cross-origins和Timing-Allow-Origin标头

收集跨源资源的度量标准需要设置[Timing-Allow-Origin](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Timing-Allow-Origin)标头。

##### 持久连接会影响时间

持久HTTP连接是指重用连接以传输其他资源。在HTTP / 1中使用`Connection：Keep-Alive`标头时，会启用持久连接。当通过HTTP / 2提供内容时，将使用单个连接来流式传输该源的所有资源。此行为会影响计时。

##### 计时API的浏览器兼容性

不能总是依赖导航和资源计时API。

例如，在Safari的一个奇怪的情况下（从版本11.2开始），支持资源计时，但导航计时不支持。

### 处理性能数据

#### 使用`navigator.sendBeacon`

收集了性能数据，需要发送到某个地方以便稍后进行分析。

通常在`unload`事件期间将数据发送到服务器，因为这是用户已完成页面的时间。 问题是在卸载期间运行代码可能阻塞下一页的下载和渲染。

需要以一种在加载新页面时不会占用浏览器的方式发回数据。 `navigator.sendBeacon`使用非阻塞请求异步发送POST数据：

```js
window.addEventListener("unload", function() {
  // 注意: 如果entries数据量很大，不要使用getEntries
  let rumData = new FormData();
  rumData.append("entries", JSON.stringify(performance.getEntries()));

  // 排列beacon请求，检查失败
  if(!navigator.sendBeacon("/phone-home", rumData)) {
    // sendBeacon失败，使用XHR或fetch替代
  }
}, false);
```

#### `navigator.sendBeacon`不可用时

需要检查`navigator.sendBeacon`浏览器兼容性：

```js
window.addEventListener("unload", function() {
  // 获取性能数据
  let rumData = new FormData();
  rumData.append("entries", JSON.stringify(performance.getEntries()));

  // 检查是否支持sendBeacon
  if("sendBeacon" in navigator) {
    if(navigator.sendBeacon(endpoint, rumData)) {
      // sendBeacon成功
    } else {
      // sendBeacon失败，使用XHR或fetch替代
    }
  } else {
    // 不支持sendBeacon ，使用XHR或fetch替代
  }
}, false);
```
