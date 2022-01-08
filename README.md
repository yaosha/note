# 简介

使用gitbook构建静态文档，使用如下命令启动项目

```
npm install

//本地运行
npm run start

//生成到docs目录
npm run build
```

## 运行失败

如果遇到如下错误
```js
TypeError: cb.apply is not a function
    at D:\MyProject\note\node_modules\npm\node_modules\graceful-fs\polyfills.js:287:18
    at FSReqCallback.oncomplete (fs.js:193:5)
```

删除`node_modules\npm\node_modules\graceful-fs\polyfills.js`62-64行代码
```js
  fs.stat = statFix(fs.stat)
  fs.fstat = statFix(fs.fstat)
  fs.lstat = statFix(fs.lstat)
```