---
title: 有意思的bug（一）
date: 2021-05-24 22:38:25
tags:
- bug
---
## h5 wxsdk分享好友、朋友圈图片不展示
这个bug很有意思，表现是有些图片会展示，有些图片不展示。

于是我想到可能是`webpack`打包的问题

由于项目是通过脚手架搭建的，我们找到`url-loader`，可以看到`options`中有`imageInlineSizeLimit`。

```js
// webpack.config.js
{
    test: [/\.bmp$/, /\.gif$/, /\.jpe?g$/, /\.png$/],
    loader: require.resolve('url-loader'),
    options: {
        limit: imageInlineSizeLimit,
        name: 'static/media/[name].[hash:8].[ext]',
    },
},
```
`const imageInlineSizeLimit = parseInt(process.env.IMAGE_INLINE_SIZE_LIMIT || '10000');`

那么原因就很清晰了，大于10000的会打包成`base64`，微信读取不了base64为封面图，所以无法展示。


## ios无法解析YYYY-MM-DD
事情是这样的，有人说小程序有页面白屏，我是安卓手机，简单排查了一下，没有复现出来。于是看了下日志，大部分机型为ios，于是借了一台iPhone手机，真机一看，还真是白屏。

在`vConsole`中查看报错，看到了`Invalid Date`

问题源头找到了，在苹果系统无法解析`YYYY-MM-DD HH:mm:ss`这种日期格式，当然解决方案很简单，只要将`-`替换为`/`即可
`new Date(date.replace(/\-/g, "/"));`

但实际上在安卓系统上解析这种日期格式完全无问题。 
例如`new Date("2019-03-31 21:30:00");`是可以正常解析的

像这种问题就和经验很有关系，如果遇到一次，下次绝对不会踩坑了

## Chrome弹出自动翻译问题
背景是这样的，一开始在本地进行测试，是没有问题的，在Chrome下不会弹出自动翻译弹窗，但是发到线上之后，一进入网页，就会自动弹出是否要翻译成中文的弹窗，体验很差。

这就很奇怪，莫非是页面的英文字母太多了吗，简单google了一下，并没有查到自动翻译弹出的规则

灵机一动，想到是不是html模板的问题

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0,minimum-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>demo</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
  </body>
</html>
```

重点在这行`<html lang="en">`，设置了`lang="en"`，脚手架用惯了，反而忽视了最基本的东西。

https://www.w3schools.com/tags/ref_language_codes.asp

通过查阅w3c官网，我们设置`lang="zh"`，问题顺利解决。