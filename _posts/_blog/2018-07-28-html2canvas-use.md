---
layout: post
title:  "html2canvas-实现前端截图"
date:   2018-07-28 8:00:00 
categories: blog
tags: ['HTML5']
published: true
comments: true
script: [post.js]
---

* Do not remove this line (it will not be displayed)
{: toc}

## 简介

`html2cavnas` 可以帮助你对web页面进行截屏。官网说因为截屏基于DOM，并不能100%将屏幕信息还原。我实际应用过后，发现效果还不错。

### 原理

页面加载完成后，`html2canvas` 对DOM进行遍历。它会收集所有用来构建页面的元素的信息。换句话说，它实际上没有截图(比如我们经常使用的手机截图功能)，而是通过读取DOM的属性信息去在canvas进行绘制，类似构建一个截图区域的镜像。

这样的结果是，它只能正确渲染它能理解的属性，这意味着有一些CSS属性会不生效。你可以查看它[支持的CSS属性](http://html2canvas.hertzen.com/features/)。

### 局限性

`html2canvas` 使用到的图像应该遵守 [“同源策略”](http://en.wikipedia.org/wiki/Same_origin_policy)，除非它使用了代理。同样的，如果页面上其它的 `canvas` 中包含跨域的内容，也不会被  `html2canvas` 正确的识别。

`html2canvas` 不渲染插件内容，如Flash或Java applet。

### 浏览器支持

可以在下列浏览器中正常工作。(可能需要 `Promise` polyfill)

* Firefox 3.5+
* Google Chrome
* Opera 12+
* IE 9+
* Edge
* Safari 6+

## 使用

html2canvas(element, options)

```
npm install html2canvas
```

```
import html2canvas from 'html2canvas';
html2canvas(docuemnt.body).then(function (canvas) {
    document.body.appendChild(canvas);
})
```

### 参数

|名称|默认值|描述|
|--|--|--|
|async|true|是否异步去解析和渲染元素|
|allowTaint|false|是否允许跨域的图片去污染画布|
|backgroundColor|#ffffff|画布的背景色，如果DOM中声明为'none'，设置为 `null`|
|canvas|null|用来绘制的 `canvas` 元素|
|foreignObjectRendering|false|如果浏览器支持，是否使用 `ForeignObject` 渲染 (`ForeignObject`， 一种SVG元素)|
|imageTimeout|15000|加载图片的超时时间。设置为0，禁用超时|
|ignoreElements|(elemnt) => false|设置忽略元素的方法|
|logging|true|启用日志记录，用于调试|
|onclone|null|用于修改渲染结果的回调函数|
|proxy|null|用于加载跨域的图片，如果为 `null`，图片不加载|
|removeContainer|true|是否清楚由 `html2canvas` 创建的副本|
|scale|window.devicePixeRatio|缩放比例，默认为设备像素比|
|useCORS|false|是否尝试从使用了 `CORE` 的服务器加载图片|
|width|Element width|画布的宽度|
|height|Element height|画布的高度|
|x|Element x-offset|截图的x轴坐标|
|y|Element y-offset|截图的y轴坐标|
|scrollX|Element scrollX|渲染元素时的y轴滚动距离|
|scrollY|Element scrollY|渲染元素时的x轴滚动距离|
|windowWidth|Window.innerWidth|渲染元素时的窗口宽度|
|windowHeight|Window.innerHeight|渲染元素的窗口高度|

如果你想获得呈现排除某些元素,可以给该元素添加一个`data-html2canvas-ignore属性` ，`html2canvas` 将他们排除在渲染。

## 遇到的问题

### 1.图片跨域的问题

当引入跨域的图片，使用 `html2canvas` 时，会报错，显示“画布被污染了”。

解决方法：

第一步，设置 `useCORS` 为 `true`

```
useCORS: true
```
第二步，服务端改造，在图片响应的 header 中加入如下信息，表示允许跨域访问。
```
'Access-Control-Allow-Origin': '*'
```



## 实现效果

![](http://47.93.235.47:5001/upload/2018/07/28/024993a1-ecca-417b-8dd6-fda0b94ea18a..gif)

## 参考资料

[html2canvas官网](https://html2canvas.hertzen.com/documentation)

[基于html2canvas实现网页保存为图片及图片清晰度优化](https://segmentfault.com/a/1190000011478657)

[工作手记之html2canvas使用概述](https://segmentfault.com/a/1190000007356836)


{% endpost #9D9D9D %}

