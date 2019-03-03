## 优化内容效率

网络应用通常需要获取几十个（有时甚至是几百个）不同的资源，所有这些资源加起来的数据量高达几兆字节，并且必须在短短几百毫秒内汇聚起来，以实现我们想要达到的即时网络体验。

方法：避免不必要的下载、通过各种压缩技术优化每个资源的传送编码以及尽可能利用缓存来避免多余的下载。

### 避免不必要的下载

最快速并且优化最充分的资源是不需要发送的资源。从应用中消除不必要的资源。
* 清点页面自有资源和第三方资源。
* 评估每个资源的表现：其价值及其技术性能。
* 确定这些资源是否提供了足够的价值。

对隐式和显式的假设提出问题并定期重检:
* 网页上一直包含资源 X，但它提供给用户的价值能否抵消下载并显示它的开销呢？是否能够评估并证明其价值？
* 该资源（特别是第三方资源）能否保持稳定的性能？该资源是否处于或是否需要处于关键路径中？如果该资源处于关键路径中，是否会成为我们网站的单点故障？也就是说，如果该资源不可用，是否会影响网页的性能和用户体验？
* 该资源是否需要或具有 SLA(service level agreement)？该资源是否遵循性能最佳做法：压缩、缓存等？

针对网页上的每个资源定期清点和重新评估上述问题。

### 优化基于文本的资源的编码和传送大小

通过优化和压缩资源来最大限度减小总下载大小。

#### 数据压缩基础知识

根据资源类型（文本、图像、字体等），有若干不同的技术可供使用：可在服务器上启用的通用工具、适用于特定内容类型的预处理优化以及需要开发者输入的资源特定优化。

* 压缩就是使用更少的位对信息进行编码的过程。
* 消除不必要的数据总是会产生最好的结果，例如注释。
* 有许多种不同的压缩技术和算法，例如数据转换、高效编码。
* 利用各种技术来实现最佳压缩。

#### 源码压缩：预处理和环境特定优化

* 建立一个不同内容类型的清单
* 进行哪些类型的内容特定优化来减小其大小
* 加入内部版本和发行版本流程来自动执行这些优化

#### 通过 GZIP 压缩文本

* GZIP 对基于文本的资源的压缩效果最好：CSS、JavaScript 和 HTML。
* 所有现代浏览器都支持 GZIP 压缩，并且会自动请求该压缩。
* 服务器必须配置为启用 GZIP 压缩。
* 某些 CDN 需要特别注意以确保 GZIP 已启用。

GZIP 是一种可以作用于任何字节流的通用压缩程序。它会在后台记忆一些之前看到的内容，并尝试以高效方式查找并替换重复的数据片段。

所有现代浏览器都支持并自动协商将 GZIP 压缩用于所有 HTTP 请求。确保服务器得到正确配置，能够在客户端请求时提供压缩过的资源。

将文件压缩源码后（产生文件名中包含“.min”的文件），再使用 GZIP 进行压缩，可进一步提高压缩率。
1. 先应用内容特定优化：CSS、JS 和 HTML 压缩源码程序。
2. 采用 GZIP 对压缩源码后的输出进行压缩。

`HTML5 Boilerplate`项目包含所有最流行服务器的[配置文件样例](https://github.com/h5bp/server-configs)。

了解 GZIP 的实用效果：打开 Chrome DevTools，然后检查“Network”面板中的“Size / Content”列：“Size”表示资源的传送大小，“Content”表示资源的未压缩大小。

> 注意：
>
> GZIP 有时会增加资源的大小。服务器需要指定最小文件大小阈值。
> 使用CDN 需要确保资源确实得到压缩

### 图像优化

#### 消除和替换图像

* 消除多余的图像资源
* 尽可能利用 CSS3 效果
  * CSS 效果（渐变、阴影等）和 CSS 动画可用于产生与分辨率无关的资源，这些资源在任何分辨率和缩放级别下始终都能清晰地显示，并且需要的字节数往往只是图片文件的几分之一。
* 使用网页字体取代在图像中进行文本编码
  * 网页字体可以在保留选择文本、搜索文本和调整文本大小能力的同时使用漂亮的字体，大大提高了易用性。

接下来应该考虑是否存在某种替代技术，能够以更高效的方式实现所需的效果：

#### 矢量图像与光栅图像

实现
* 矢量图形使用线、点和多边形来表示图像。
* 光栅图形（例如 GIF、PNG、JPEG 或 JPEG-XR 和 WebP 等某种较新的格式）通过对矩形格栅内的每个像素的值进行编码来表示图像。

区别
* 矢量图像最适用于包含几何形状的图像
* 矢量图像与缩放和分辨率无关
* 光栅图像应用于包含大量不规则形状和细节的复杂场景，当放大光栅图像时，图形会出现锯齿并且模糊不清。因此需要在不同分辨率下保存多个版本的光栅图像，以便为用户提供最佳体验。

#### 高分辨率屏幕的含义

* 高分辨率屏幕的每个 CSS 像素包含多个设备像素
* 高分辨率图像需要的像素数和字节数要多得多
* 任何分辨率都采用相同的图像优化方法

像素有2种：CSS 像素和设备像素。单个 CSS 像素可能包含多个设备像素。设备像素越多，屏幕上所显示内容的细节就越丰富。分辨率越高，单个 CSS 像素包含的设备像素越多。

尽可能优先使用矢量图像，如果需要使用光栅图像，使用[img标签的srcset属性](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/img)和[picture标签](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/picture)提供并优化每个图像的多个变体。

#### 优化矢量图像

SVG 是一种基于 XML 的图片格式

优化
* 使用类似[svgo](https://github.com/svg/svgo)的工具进行缩减，以减小其大小
* 使用 GZIP 进行压缩

#### 优化光栅图像

* 光栅图像是2 维像素栅格，例如，100x100 像素的图像是 10,000 个像素的序列
* 每个像素都编码了颜色和透明度信息，即“RGBA”值
  * (R) 红色通道、(G) 绿色通道、(B) 蓝色通道和 (A) alpha（透明度）通道。
  * 浏览器为每个通道分配 256 个值（色阶），也就是每个通道 8 位 (2 ^ 8 = 256)，每个像素 4 个字节（4 个通道 x 8 位 = 32 位 = 4 个字节）。因此，如果我们知道栅格尺寸，就能轻易计算出文件大小（W\*H\*4/1024）。
* 图像压缩程序使用各种方法来减少每个像素所需的位数，以减小图像的文件大小
  * 将图像的“位深”从每个通道 8 位减少为更小的调色板
  * 使用增量编码，不存储像素值，存储相邻元素差异值

#### 无损图像压缩与有损图像压缩

* 由于人眼的工作方式的缘故，对图像进行有损压缩是不错的选择
* 图像优化依赖有损和无损压缩来实现
* 图片格式上的差异是由于优化图像时使用的有损和无损算法的差异和使用方式的差异所致
* 并不存在任何适用于所有图像的最佳格式或“质量设置”：每个特定压缩程序与图像内容的组合都会产生独特的输出

图像和文本不同，提供原始数据的“近似”也能够接受。

典型的图像优化过程由两个高级步骤组成：
1. 使用“有损”过滤器处理图像，去除某些像素数据，可选，具体算法取决于特定的图片格式
2. 使用“无损”过滤器处理图像，对像素数据进行压缩

有损和无损优化的“最佳”配置取决于图像内容、标准、环境，并不存在任何通用的设置。

#### 选择正确的图片格式

* 首先选择正确的通用格式：GIF、PNG、JPEG
* 通过试验选出每一种格式的最佳设置：质量、调色板大小等
* 考虑为现代化客户端添加 WebP 和 JPEG XR 资源

不同的图片格式支持不同的功能。根据所需视觉效果与功能要求相结合选择正确的格式。

|格式|透明度|动画|浏览器|
|---|---|---|---|---|
|GIF|支持|支持|所有|
|PNG|支持|不支持|所有|
|JPEG|不支持|不支持|所有|
|JPEG|XR|支持|支持|IE|
|WebP|支持|支持|Chrome、Opera、Android|

1. 是否需要动画？如果需要，GIF 是唯一的通用选择。
  * GIF 将调色板限制为最多 256 色，这对大多数图像而言都不是好的选择。况且，对于调色板较小的图像，PNG-8 的压缩效果更佳。因此，只有需要动画时，GIF 才是正确的选择。
2. 是否需要使用最高分辨率保留精细的细节？请使用 PNG。
  * 除了选择调色板的大小外，PNG 不采用任何有损压缩算法。因此，它能生成最高质量的图像，但代价是文件大小要比其他格式大得多。请谨慎使用。
  * 如果图像资源包含由几何形状组成的图像，请务必考虑将其转换成矢量 (SVG) 格式！
  * 如果图像资源包含文本，考虑使用文本替换。图像中的文本无法选择、搜索或“缩放”。如果需要表现一种自定义外观，请改用网页字体。
3. 是否要优化照片、屏幕截图或类似的图像资源？请使用 JPEG。
  * JPEG 组合使用有损和无损优化来减小图像资源的文件大小。尝试几种 JPEG 质量级别，为资源找到最佳的质量与文件大小平衡点。

* 考虑增加一个以 WebP 和 JPEG XR 格式编码的变体。
  * 均为新格式，尚未得到所有浏览器的普遍支持
  * 显著降低文件大小

#### 工具和参数调优

根据图像的内容及其视觉以及其他技术要求来挑选格式及其设置。

|工具|说明|
|--|--|
|gifsicle|创建和优化GIF图像|
|jpegtran|优化JPEG图像|
|optipng|无损PNG优化|
|pngquant|有损PNG优化|

#### 提供缩放的图像资源

* 提供缩放的资源是最简单并且最有效的优化之一
  * 图像优化可归结为两个标准：优化编码每个图像像素所使用的字节数，和优化总像素数：图像的文件大小就是总像素数与编码每个像素所使用字节数的乘积。
  * 确保提供的像素数恰好是在浏览器中按预期尺寸显示资源所需的像素数
  * 在 Chrome DevTools 中将光标悬停在图像元素上，可同时显示该图像资源的原始尺寸和显示尺寸。
* 密切关注较大的资源，因为它们会产生大量开销
* 通过将图像缩放到其显示尺寸，减少多余的像素数

#### 技巧和方法总结

* 首选矢量格式：矢量图像与分辨率和缩放无关，是多设备和高分辨率情况的完美选择。
* 缩小和压缩 SVG 资源：大多数绘图应用程序生成的 XML 标记包含可以移除的多余元数据；配置服务器对 SVG 资源采用 GZIP 压缩。
* 挑选最佳光栅图片格式：确定功能要求并选择适合每个特定资源的格式。
* 通过试验为光栅格式找到最佳质量设置：不要害怕调低“质量”设置，调低后的效果通常很不错，并且字节数的缩减很显著。
* 移除多余的图像元数据：许多光栅图像都包含多余的资源元数据：地理信息、相机信息等。请使用合适的工具删除这些数据。
* 提供缩放的图像：调整服务器上的图像尺寸，并确保图像的“显示”尺寸尽可能接近其“自然”尺寸。尤其要密切注意较大的图像，因为在调整尺寸时，它们占用的开销最大！
* 自动化：使用自动化工具和基础设施，确保所有图像资源始终得到优化。

### 自动化图像压缩

[自动化图像压缩](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/automating-image-optimization/)

使用[imagemin](https://github.com/imagemin/imagemin)或[libvps](https://github.com/jcupitt/libvips)进行自动化构建过程。

使用[Imageflow](https://github.com/imazen/imageflow)或[Thumbor](https://github.com/thumbor/thumbor)实现自托管方案。

### 用视频替换动画GIF

[用视频替换动画GIF](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/replace-animated-gifs-with-video/)

### JavaScript启动优化

提供更少的JavaScript可以减少网络传输的时间，减少解码代码的时间，减少解析和编译JavaScript的时间。

#### 网络

考虑JavaScript的下载和执行成本。

降低JavaScript的网络传输成本的方式：
* 只发送用户需要的代码。
  * 使用代码分割将JavaScript分解为关键和非关键。例如`webpack`的代码分割功能。
  * 懒加载非关键代码。
* 缩小
  * 使用`UglifyJS`缩小ES5代码。
  * 使用`babel-minify`或`uglify-es`缩小ES6代码。
* 压缩
  * 使用`gzip`压缩基于文本的资源。
  * 其它压缩工具，Brotli~q11。 Brotli在压缩比方面优于gzip。
* 删除未使用的代码
  * 使用Chrome v59+的`DevTools`的`Show Coverage`查看代码覆盖率，识别可以删除或延迟加载的代码。
  * 使用`babel-preset-env`和`browserlist`避免现有浏览器中已有的功能。分析webpack包，修剪不需要的依赖项。
  * 对于剥离代码，参考[tree-shaking](https://webpack.js.org/guides/tree-shaking/)。
* 缓存代码以最小化网络开销
  * 使用HTTP缓存来确保浏览器有效地缓存响应。确定脚本（max-age）的最佳生命周期并提供验证令牌（ETag）以避免传输未更改的字节。
  * Service Worker缓存可以使应用程序网络具有弹性，并可以访问V8代码缓存等功能。
  * 使用长期缓存以避免重新获取未更改的资源。如果使用Webpack，请参阅[文件名哈希](https://webpack.js.org/guides/caching/)。

#### 解析/编译

JavaScript下载后的成本是JS引擎解析/编译此代码的时间。

Chrome DevTools的Performance面板可以查看解析/编译时间：
* 解析和编译是Performance面板中黄色“脚本”时间的一部分。
* Bottom-Up和Call Tree选项卡显示精确的解析/编译时序。
  * 启用V8的`Runtime Call Stats`，可以看到在Parse和Compile等阶段花费的时间
  * Runtime Call Stats的性能面板支持目前是实验性的。 要启用，请转到`chrome://flags/#enable-devtools-experiments`->重启Chrome->打开DevTools->Settings->Experiments->按`shift`键6次 - >检查`Timeline: V8 Runtime Call Stats on Timeline`项、然后关闭再重新打开DevTools。

花费很长时间解析/编译代码会严重延迟用户与网站交互的时间。发送的JavaScript越多，在网站进行交互之前解析和编译它所需的时间就越长。

浏览器处理JavaScript的成本要高于同样大小的图像或Web字体。
* 同样大小的JavaScript和图像网络`传输`时间花费一样
* JavaScript`解析和编译`时间比图像`解码`时间长
* JavaScript`执行`时间比图像`绘制`时间长
* JavaScript解析、编译、执行会阻塞主线程，而不像不会

设备和网络环境同样重要
* 普通用户可能使用CPU和GPU速度较慢的手机，没有L2 / L3缓存，甚至可能受内存限制。
* 网络功能和设备功能并不总是匹配。
* 最快的手机和普通手机之间解析/编译代码的时间差异为2-5倍。

所以针对用户拥有的设备和网络条件进行优化。

删除非关键JavaScript可以减少传输时间，CPU密集型解析和编译以及潜在的内存开销。

#### 执行时间

JavaScript执行（解析/编译后运行代码）必须在主线程上执行。 较长的执行时间会推迟用户与网站的交互时间。

必须在主线程执行的操作：
* JavaScript解析、编译、执行
* DOM构建
* 布局
* 输入的处理

如果脚本执行时间超过50毫秒，则交互时间因JS下载、编译和执行所需的全部时间而延迟。
* JavaScript分小块执行，避免锁定主线程。 
* 探索是否可以减少执行期间完成的工作量。

#### 其他开销

JavaScript可以通过其他方式影响页面性能：
* 内存。 由于浏览器执行GC（垃圾回收）导致JS执行暂停，页面可能会出现频繁抖动或暂停。避免内存泄漏和频繁的GC暂停，以避免页面抖动或暂停。
* 在运行时长时间运行的JavaScript会阻止主线程导致页面无响应。将工作分成小块（使用[requestAnimationFrame()](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame)或[requestIdleCallback()](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestIdleCallback)进行调度）可以最大限度地减少响应问题。

#### 降低JavaScript传输成本的模式

有些模式像基于路由的分块或PRPL可以帮助降低JavaScript传输成本。

##### PRPL

PRPL（Push, Render, Pre-cache, Lazy-load）是一种通过积极的代码分割和缓存优化交互性的模式：
* 用户请求时，为初始路由`Push`最小化代码
  * Push/Preload关键脚本
    * `Render`路由，实现页面可交互
* 导航到下个路由
  * 使用Service Workers实现`Pre-cache`，缓存资源
* 导航到另一个路由
  * `Lazy-load`异步（分割）路由

##### 渐进式引导

渐进式引导：发送一个最小功能页面（仅包含当前路由所需的HTML / JS / CSS）。 随着更多资源的到来，该应用程序可以延迟加载和解锁更多功能。

#### 结论

传输大小对于低端网络至关重要。 解析时间对CPU绑定设备很重要。 

考虑架构决策有多少JS“空间”可以让我们留下app逻辑。

如果构建一个面向移动设备的网站，请尽力在代表性硬件上进行开发，保持较低的JavaScript解析/编译时间并采用性能预算，以确保能密切关注其JavaScript成本。

### 加载第三方JavsScript

第三方JavaScript指可以直接从第三方供应商嵌入任何站点的脚本。包括广告，分析，小部件和其他脚本，使Web更具动态性和交互性。

#### Chrome DevTools检查第三方脚本

Chrome DevTools支持在“Network”面板中突出显示第三方。

启用第三方检查，需要打开Chrome DevTools的任意面板，然后按CMD + Shift + P显示命令菜单。接下来输入“Show third party badges”。

#### 如何衡量第三方脚本对页面的影响？

##### Lighthouse启动时间审计

Lighthouse JavaScript启动时间审计突出显示解析、编译或评估时间花费太大的脚本。 这对于发现CPU密集型第三方脚本非常有用。

##### Lighthouse网络负载审计

Lighthouse Network Payloads审计识别可能会减慢页面加载时间的网络请求（包括来自第三方的请求）。

##### Chrome DevTools阻止网络请求

通过阻止网络请求，Chrome DevTools可以查看当特定脚本、样式表或其他资源不可用时页面的行为方式，帮助衡量阻止页面中特定第三方资源的影响。

启用阻止请求，请右键单击“Newwork”面板中的任何请求，然后选择“Block Request URL”。 “Request blocking”选项卡可以管理已阻止的请求。

##### Chrome DevTools性能面板

Chrome DevTools中的“Performance”面板可帮助识别网页性能问题。 单击录制按钮并加载页面会显示一个瀑布，表示网站花费时间。 

导航到“Performance”面板的底部的“Summary”，选项卡，选择“Bottom-up”。选择“Group by product”选项，按照花费的时间对第三方进行分组，有助于确定哪些第三方产品成本最高。

衡量第三方脚本影响的良好工作流程是：
* 使用“Network”面板测量加载页面所需的时间。
  * 建议启用网络限制和CPU限制模拟真实条件。
* 阻止认为存在问题的第三方脚本的URL或域。
* 重新加载页面，不加载被阻止的第三方脚本，并重新测量页面所需的时间。
  * 进行3次或更多次测量并查看中位数。

##### 使用Long Tasks检测代价高昂的iframe

第三方iframe中的的长耗时脚本会阻止主线程，并延迟运行其他任务。 这些长耗时任务会导致负面的用户体验，导致事件处理缓慢或丢帧。

使用JavaScript PerformanceObserver API并观察longtask条目来检测真实用户监控（RUM）的长耗时任务。 由于这些条目包含归属属性，因此可以跟踪哪个帧上下文负责该任务。

```js
<script>
    const observer = new PerformanceObserver((list) => {

        for (const entry of list.getEntries()) {

            // Attribution entry including "containerSrc":"https://example.com"
            console.log(JSON.stringify(entry.attribution));

        }

    });

    observer.observe({ entryTypes: ['longtask'] });
</script>

<!-- Imagine this is an iframe with expensive long tasks -->
<iframe src="https://example.com"></iframe>
```

#### 如何有效地加载第三方脚本？

* 使用`async`或`defer`属性加载脚本以避免阻止文档解析。
* 如果第三方服务器运行缓慢，请考虑自托管脚本。
* 如果脚本没有明确的价值，请考虑删除该脚本。
* 考虑资源提示，如`<link rel = preconnect>`或`<link rel = dns-prefetch>`，以便为托管第三方脚本的域执行DNS查找。

##### 使用异步或延迟

JavaScript执行会打断浏览器解析HTML。当浏览器遇到脚本时，必须暂停DOM构造，将其交给JavaScript引擎并允许脚本执行，然后再继续构建DOM。

`async`和`defer`属性会更改此行为。
* async：浏览器会在继续解析HTML文档时异步下载脚本。 当脚本完成下载时，在脚本执行时阻止解析。
* defer：浏览器在继续解析HTML文档时异步下载脚本。 在解析完成之前，脚本不会运行。

始终对第三方脚本使用async或defer（除非执行关键呈现路径依赖该脚本）：
* async：如果在加载过程中让脚本早期运行很重要
* defer：不太重要的资源

##### 使用资源提示可减少连接设置时间

建立与第三方来源的连接可能会花费大量时间 - 尤其是在慢速网络上。许多步骤可能会导致延迟，包括DNS查找，重定向以及可能需要多次往返每个第三方服务器来处理请求。

使用`dns-prefetch`为托管第三方脚本的域执行DNS查找。当最终发出对它们的请求时，因为已经执行了DNS查找所以节省了时间。

```html
<link rel="dns-prefetch" href="http://example.com">
```

如果第三方域使用HTTPS，执行DNS查找并解决TCP往返并处理TLS协商。其他步骤非常慢，因为涉及查看SSL证书以进行验证，因此如果发现第三方设置时间成为问题，使用`preconnect`。

```html
<link rel="preconnect" href="https://cdn.example.com">
```

##### 使用iframe处理脚本

将第三方脚本直接加载到iframe，不会阻止主页的执行。

>注意
>
> 此方法阻止onload事件，因此不要将关键功能附加到onload。

##### 自托管第三方脚本

自托管第三方脚本可以更好地控制脚本的加载。

警告：
* 脚本可能会过时。
* 由于API更改，自托管的脚本无法获得自动更新。

替代方法：使用Service Workers来缓存。

##### 懒加载第三方资源

懒加载用于仅在必要时加载嵌入的资源
* 用户交互后才加载脚本
* 在主页面内容加载之后但在用户可能以其他方式与页面交互之前加载内容

使用Intersection Observer提升懒加载效率

通常侦听`scroll`或`size`事件，然后使用像`getBoundingClientRect()`这样的DOM API计算元素相对于视口的位置。

问题：检测元素在视口中是否可见来懒加载脚本容易出错，通常会导致浏览器变得迟缓。

解决方法：[IntersectionObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)是一个浏览器API，有效地检测被观察元素何时进入或退出浏览器的视口。

#### 使用第三方脚本应避免的模式

##### 避免使用document.write()

不使用document.write()注入脚本。从Chrome 53开始，Chrome DevTools会将使用有问题的document.write()警告记录到控制台。

Lighthouse突出显示使用document.write()的任何第三方脚本。

##### 标签管理器

“标签”是一段代码，允许收集数据、设置cookie或将第三方内容集成到网站中。这些标记会降低页面的加载性能：额外的网络请求，繁重的JavaScript依赖关系，标记本身可能引入的图像和资源。

随着标签的增多，为了保持快速体验，使用标记管理器：
* 在一个单独的地方管理第三方嵌入代码
* 尝试最小化第三方标记的数量

Google Tag Manager (GTM) ：

* 优点
  * 异步代码，执行时不会阻止其他元素在页面上呈现
  * 异步部署通过GTM部署的其他代码，慢加载标签不会阻止其他跟踪代码。
* 风险
  * 拥有凭据和访问权限的任何人都可以轻松添加更多标签，导致生成和执行过多的昂贵HTTP请求。通过仅允许一个用户发布版本来最小化。
  * 任何人都可以配置太多标记管理器自动事件侦听器。每个自动事件监听器都需要执行，而且代码和网络请求越多，页面完全加载所需的时间就越长。在50毫秒内响应事件解决。

##### 避免脚本污染全局域

未知的第三方脚本会加载自己的JavaScript依赖项，可能污染全局域并导致页面意外崩溃。

仔细审核第三方脚本，完成自我测试，子资源完整性和安全传输第三方代码。

#### 缓解策略

最小化第三方脚本对性能和安全性的影响的策略：
* `HTTPS`是必须的。HTTPS网站不应该加载使用HTTP的脚本。
* 考虑iframe上的[sandbox attribute](https://developer.mozilla.org/en/docs/Web/HTML/Element/iframe)。从安全角度来看，允许限制iframe中可用的操作。限制包括allow-scripts控制上下文是否可以运行脚本。
* 考虑`Content Security Policy (CSP)`。通过服务器响应中的HTTP标头，定义页面中受信任的行为。 CSP可用于检测和减轻某些攻击的影响，例如跨站点脚本（XSS）。

CSP特别强大，因为它包含诸如script-src之类的指令，用于指定JavaScript的允许来源。以下是如何在实践中使用它的示例：

```
// Given this CSP header

Content-Security-Policy: script-src https://example.com/

// The following third-party script will not be loaded or executed

<script src="https://not-example.com/js/library.js"></script>
```

#### 结论

结论

不要忽略第三方脚本性能：
* 熟悉最有效的第三方脚本优化方法，例如只加载支持异步加载模式的标记。
* 了解如何识别和修复第三方脚本加载的问题。 
* 优化之后，对脚本进行持续的实时性能监控

### 字体优化

字体可以实现良好的设计、用户体验和性能。

通过规定字体在网页加载和应用的方式，可以减小总大小、缩短渲染时间。

#### 字体基本知识

* Unicode 字体可能包含数千种字形。
* 字体格式有四种：WOFF2、WOFF、EOT 和 TTF。
* 某些字体格式需要使用 GZIP 压缩。

##### 字体格式

兼容性
* EOT只有IE支持
* TTF部分IE支持
* WOFF兼容性最好，但某些
* WOFF2许多浏览器尚未实现

使用方式
* 将WOFF 2.0提供给支持它的浏览器。
* 将WOFF提供给大多数浏览器。
* 将TTF提供给旧 Android（4.4 版以下）浏览器。
* 将EOT提供给旧 IE（IE9 版以下）浏览器。

##### 压缩字体大小

* EOT 和 TTF 格式默认不进行压缩，但在使用时服务器配置 GZIP 压缩。
* WOFF 具有内建压缩。
* WOFF2 采用自定义预处理和压缩算法，提供的文件大小压缩率比其他格式高大约 30%。

某些字体格式包含附加的元数据，删除这些数据可以进一步压缩字体大小。

#### @font-face定义字体系列

* 利用 format() 提示指定多种字体格式。
* 对大型 Unicode 字体进行子集内嵌以提高性能。使用 Unicode-range 子集内嵌，并为较旧的浏览器提供手动子集内嵌回退。
* 减少风格字体变体的数量以改进网页和文本渲染性能。

##### 选择格式

* font-family: 字体系列名称
* font-style: 字体样式;
* font-weight: 字体粗细;
* src: 指定字体资源位置优先级
  * local: 引用、加载和使用安装在本地的字体
  * url('/fonts/awesome.woff2') format('woff2'): url加载外部字体，包含可选的format指示字体格式

```css
@font-face {
  font-family: 'Awesome Font';
  font-style: normal;
  font-weight: 400;
  src: local('Awesome Font'),
       url('/fonts/awesome.woff2') format('woff2'),
       url('/fonts/awesome.woff') format('woff'),
       url('/fonts/awesome.ttf') format('truetype'),
       url('/fonts/awesome.eot') format('embedded-opentype');
}

@font-face {
  font-family: 'Awesome Font';
  font-style: italic;
  font-weight: 400;
  src: local('Awesome Font Italic'),
       url('/fonts/awesome-i.woff2') format('woff2'),
       url('/fonts/awesome-i.woff') format('woff'),
       url('/fonts/awesome-i.ttf') format('truetype'),
       url('/fonts/awesome-i.eot') format('embedded-opentype');
}
```

浏览器确定字体规则：按指定顺序循环访问提供的资源列表，并尝试加载相应的资源

1. 浏览器执行页面布局并确定需要使用哪些字体变体来渲染网页上的指定文本。
2. 对于每一种必需字体，浏览器会检查字体文件是否位于本地。
3. 如果字体文件不在本地，则浏览器会遍历外部定义：
    * 如果存在格式提示，则浏览器会在启动下载之前检查其是否支持。如果浏览器不支持此提示，则会前进到下一格式提示。
    * 如果不存在格式提示，则浏览器会下载资源。

##### Unicode-range 子集内嵌

@font-face 规则还可以定义各资源支持的 Unicode 代码点集。可将大型 Unicode 字体拆分成较小的子集，并且只下载在特定网页上渲染文本所需的字形。

使用unicode-range 描述符指定一个用逗号分隔的范围值列表，使用形式：
* 单一代码点（例如 U+416）
* 间隔范围（例如 U+400-4ff）：表示范围的开始代码点和结束代码点
* 通配符范围（例如 U+4??）：“?”字符表示任何十六进制数字

```css
@font-face {
  font-family: 'Awesome Font';
  font-style: normal;
  font-weight: 400;
  src: local('Awesome Font'),
       url('/fonts/awesome-l.woff2') format('woff2'),
       url('/fonts/awesome-l.woff') format('woff'),
       url('/fonts/awesome-l.ttf') format('truetype'),
       url('/fonts/awesome-l.eot') format('embedded-opentype');
  unicode-range: U+000-5FF; /* Latin glyphs */
}

@font-face {
  font-family: 'Awesome Font';
  font-style: normal;
  font-weight: 400;
  src: local('Awesome Font'),
       url('/fonts/awesome-jp.woff2') format('woff2'),
       url('/fonts/awesome-jp.woff') format('woff'),
       url('/fonts/awesome-jp.ttf') format('truetype'),
       url('/fonts/awesome-jp.eot') format('embedded-opentype');
  unicode-range: U+3000-9FFF, U+ff??; /* Japanese glyphs */
}
```

unicode-range并不是所有浏览器都支持，对于不支持的浏览器，必须提供包含所有必要子集的单一字体资源，并向浏览器隐藏其余子集。

* 如何确定需要哪些子集？
  * 如果浏览器支持 unicode-range 子集内嵌，则自动选择正确的子集。网页只需提供子集文件并在 @font-face 规则中指定相应的 unicode-range。
  * 如果浏览器不支持 unicode-range 子集内嵌，则网页需要隐藏所有多余的子集，即开发者必须指定需要的子集。
* 如何生成字体子集？
  * 使用开源的 pyftsubset 工具对字体进行子集内嵌和优化。
  * 某些字体服务允许通过自定义查询参数手动内嵌子集，利用这些参数手动指定网页需要的子集。

##### 字体选择和合成

每个字体系列都由多个样式变体（常规、加粗、倾斜）和适用于每个样式的多个粗细组成。

例如只提供粗细400和700的系列字体，在使用时指定其它粗细，浏览器会自动映射到最接近的字体。
* 如果未找到精确字体匹配项，浏览器将以最接近的匹配项替代。
* 如果未找到样式匹配项（例如倾斜），则浏览器将合成其自己的字体变体。

为获得最好的一致性和视觉效果，不应该依赖字体合成，而应最大限度减少使用的字体变体的数量并指定其位置。

#### 优化加载和渲染

* 在构建渲染树之前会延迟字体请求，可能会导致文本渲染延迟。
* 可以通过 Font Loading API 实现自定义字体加载和渲染策略，以替换默认延迟加载字体加载。
* 可以通过字体内联替换较旧浏览器中的默认延迟加载字体加载。

网页字体包括可能不需要的所有样式变体，加上可能不会使用的所有字形，很容易就会产生几兆字节的下载。为解决此问题，使用 @font-face CSS 规则将字体系列拆分成一个由 unicode 子集、不同样式变体等资源组成的资源集合。浏览器会确定需要的子集和变体，并下载渲染文本所需的最小集。

##### 网页字体和关键渲染路径

浏览器在构建渲染树（依赖 DOM 和 CSSOM 树）后才知道需要使用哪些字体资源来渲染文本。因此，字体请求的处理将远远滞后于其他关键资源请求的处理，并且在获取资源之前，可能会阻止浏览器渲染文本。

1. 浏览器请求 HTML 文档。
2. 浏览器开始解析 HTML 响应和构建 DOM。
3. 浏览器发现 CSS、JS 以及其他资源并分派请求。
4. 浏览器在收到所有 CSS 内容后构建 CSSOM，然后将其与 DOM 树合并以构建渲染树。
   * 在渲染树指示需要哪些字体变体在网页上渲染指定文本后，将分派字体请求。
5. 浏览器执行布局并将内容绘制到屏幕上。
   * 如果字体尚不可用，浏览器可能不会渲染任何文本像素。
   * 字体可用之后，浏览器将绘制文本像素。

不同浏览器之间实际行为有所差异：

* Safari 会在字体下载完成之前延迟文本渲染。
* Chrome 和 Firefox 会将字体渲染暂停最多 3 秒，之后他们会使用一种后备字体。并且字体下载完成后，他们会使用下载的字体重新渲染一次文本。
* IE 会在请求字体尚不可用时立即使用后备字体进行渲染，然后在字体下载完成后进行重新渲染。

##### 通过 Font Loading API 优化字体渲染

Font Loading API 提供了一种脚本编程接口来定义和操纵 CSS 字体，追踪其下载进度，以及替换其默认延迟下载行为。

##### 通过内联优化字体渲染

使用 Font Loading API 的简单替代策略是将字体内容内联到 CSS 样式表内：

* 浏览器会使用高优先级自动下载具有匹配媒体查询的 CSS 样式表，因为需要使用它们来构建 CSSOM。
* 将字体数据内联到 CSS 样式表中会强制浏览器使用高优先级下载字体，而不等待渲染树。即它起到的是手动替换默认延迟加载行为的作用。

内联策略不够灵活，不允许为不同的内容定义自定义超时或渲染策略，但不失为是一种适用于所有浏览器并且简单而又可靠的解决方案。为获得最佳效果，请将内联字体分成独立的样式表，并为它们提供较长的 max-age。这样一来，在更新 CSS 时，就不会强制访问者重新下载字体。

缺点：
* 延迟加载避免下载多余的字体变体和子集
* 增加 CSS 的大小

##### 通过 HTTP 缓存优化字体重复使用

字体资源通常是不会频繁更新的静态资源。因此，它们非常适合较长的 max-age 到期 - 确保为所有字体资源同时指定了条件 ETag 标头和最佳 Cache-Control 策略。

无需在 localStorage 中或通过其他机制存储字体，其中的每一种机制都有各自的性能缺陷。 

浏览器的 HTTP 缓存与 Font Loading API 或 webfontloader 内容库相结合，实现了最佳并且最可靠的机制来向浏览器提供字体资源。

#### 总结

* 审核并监控字体使用：不要在网页上使用过多字体，并且对于每一种字体，最大限度减少使用的变体数量。
* 对字体资源进行子集内嵌：许多字体都可进行子集内嵌，或者拆分成多个 unicode-range 以仅提供特定网页需要的字形。减小文件大小，并提高资源的下载速度。不过，在定义子集时要注意针对字体重复使用进行优化。
* 向每个浏览器提供优化过的字体格式：每一种字体都应以 WOFF2、WOFF、EOT 和 TTF 格式提供。对 EOT 和 TTF 格式应用 GZIP 压缩，因为默认情况下不会对它们进行压缩。
* 指定重新验证和最佳缓存策略：字体是不经常更新的静态资源。确保服务器提供长期的 max-age 时间戳和重新验证令牌，以实现不同网页之间高效的字体重复使用。
* 使用 Font Loading API 优化关键渲染路径：默认延迟加载行为可能导致文本渲染延迟。可以通过 Font Loading API 为特定字体替换这一行为，以及为网页上的不同内容指定自定义渲染和超时策略。对于不支持该 API 的较旧浏览器，可以使用网页字体加载程序 JavaScript 内容库或使用 CSS 内联策略。