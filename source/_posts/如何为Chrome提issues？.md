---
title: 如何为Chrome提issues？
date: 2021-05-12 23:38:01
tags:
---
## 背景
在`H5`上传文件业务开发过程中，上传功能需要让用户上传 `图片/pdf` 文件，我们很容易想到，只需要改变`input`的`accept`属性就好了，我们使用的`accept`属性如下。


`<input accept="image/jpeg,image/jpg,image/png,application/pdf" type="file" />`

但是在安卓真机`Chrome`上测试时，发生了一些问题。


<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c6fd1c00f5245d8b88dd239bdcd3390~tplv-k3u1fbpfcp-watermark.image" width="200" />

可以看到，我们点击`Choose File`之后，出现了2个不应该出现的按钮，`Camcorder` 和 `Recorder`。
通过排查，我判断是`Chrome`自身的问题，那么如何为`Chrome`提issues呢？

## 在哪里为Chrome提Issues？
https://bugs.chromium.org/p/chromium/issues/list

可能这里有人会说，这不是`chromium`的bug区吗，并不是`Chrome`。

这里简单科普一下：

- `Chromium`是谷歌的一个开源项目，所有的开发者们都可以去共同改进它。

- `Chrome`不是开源项目，谷歌会把`Chromium`的东西更新到`Chrome`中。所以`Chromium`更像是体验版，而`Chrome`是正式版。

## 提issues步骤
#### 进入Issues页面
我们打开 https://bugs.chromium.org/p/chromium/issues/list 页面
![0.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97d0e38dfe0b49649d6a18af44e0409a~tplv-k3u1fbpfcp-watermark.image)

#### 填写机型环境信息
这里`Chrome versions`，网站会自动帮我们进行识别。由于是在PC上填写，我们需要填写自己的移动设备系统与版本。
<div align="center">
    <img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4810e733663846fcbba5643e422019ef~tplv-k3u1fbpfcp-watermark.image" width="500" />
</div>

#### 选择issues类型
由于我们是开发者，直接选择`Web developer`

上述问题属于浏览器`API`的问题，我们选择 `API`
<div align="center">
    <img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/82c67571c8f34f059cf2fd28e10414f1~tplv-k3u1fbpfcp-watermark.image" width="500" />
</div>

#### 详细描述我们的问题
这里建议上传源代码，或者提供一个`codesandbox`。这里由于我的问题比较简单，我只上传了一个`html`页面。
为了让`Chrome`维护人员更好的复现bug，我上传了4张图片，也提供了在`firefox`的表现（在firefox上是正常的，没有bug）
<div align="center">
    <img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e11f9836724e48ae9196244f4444041d~tplv-k3u1fbpfcp-watermark.image" width="500" />
</div>

为`Chrome`提issues的讲解到这里就结束啦，

以下为我提的issues链接，欢迎一起围观：

https://bugs.chromium.org/p/chromium/issues/detail?id=120435

------
以下为问题详细描述

## 上文具体Case分析
### 关于h5上传popup出现录音、摄像按钮的bug。

以下是我们的最简复现页面：
<div align="center">
    <img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/30f8418d7bba473e974cffb8f283e333~tplv-k3u1fbpfcp-watermark.image" width="320" />
</div>

点击第一行的`<input id="1" accept="image/jpeg,image/jpg,image/png" type="file" />`按钮

可以看到`Chrome`的表现是正常的

<div align="center">
    <img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9b0c022af0d48908de0b8abaa1fdcc9~tplv-k3u1fbpfcp-watermark.image" width="320" />
</div>

点击第二行的`<input accept="image/jpeg,image/jpg,image/png,application/pdf" type="file" />`按钮

可以看到弹出了4个按钮，但是多出了`Camcorder`和`Recoder`，这就不正常了，我们并没有在accept中声明要求用户上传这2种格式的文件

<div align="center">
    <img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c6fd1c00f5245d8b88dd239bdcd3390~tplv-k3u1fbpfcp-watermark.image" width="320" />
</div>

那么在Firefox的表现呢？点击第二行的按钮，可以看到Everything is fine。。所以我们可以确定该bug为Chrome自身的问题。
<div align="center">
    <img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6f1ff58477d64e4898f057fd8421ccb6~tplv-k3u1fbpfcp-watermark.image" width="320" />
</div>

### 如何自行兼容？
在`input`的`onChange`回调中根据`files`进行自行判断，如果用户上传的类型不对，就舍弃掉该文件，给用户`tips`提示。

---
**以下为我提的issues**

欢迎大家一起围观：

https://bugs.chromium.org/p/chromium/issues/detail?id=1204359