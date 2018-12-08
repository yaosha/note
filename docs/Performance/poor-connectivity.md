## 低带宽和高延迟

了解应用或网站在连接不佳或不可靠时的使用情况，并进行相应的构建。

### 在低带宽和高延迟环境中测试

使用工具模拟低带宽和高延迟环境。

#### 模拟网络限速

Chrome DevTools在Network面板自定义上传/下载速度、来回通讯延迟测试应用。

Xcode中安装硬件IO工具，使用Network Link Conditioner面板模拟网络环境。

使用Android Emulator模拟Android设备的网络状况。

iPhone也可以使用Network Link Conditioner模拟网络环境。

#### 使用不同服务器位置和网络类型测试

以各种网络和主机位置运行性能测试。

在构建流程中使用脚本或通过使用API实现自动化测试。

Fiddler 支持通过 GeoEdge 进行全局代理，并且可以使用其自定义规则来模拟调制解调器的速度。

#### 在连接不佳的网络状态下测试

利用软件和硬件代理，模拟有问题的移动网络状况，例如带宽限制、封包延迟和随机丢包。利用共享代理或欠佳网络，开发团队可以将网络实战测试纳入工作流程。

Facebook 的 Augmented Traffic Control (ATC) 是以 BSD 许可证发布的一套应用，可用于控制流量和模拟连接不良的网络状况。

### 处理不可靠的连接和虚假连接

虚假连接指看似连接而实际未连接的情况。浏览器看似已连接上网络，但由于某种原因，实际并未连接。

利用超时处理时断时续的连接。[sw-toolbox](https://github.com/GoogleChromeLabs/sw-toolbox)使用服务工作线程实现测试。

```js
toolbox.router.get(
  '/path/to/image',
  toolbox.networkFirst,
  {networkTimeoutSeconds: 3}
);
```
