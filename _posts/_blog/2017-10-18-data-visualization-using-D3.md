---
layout: post
title:  "使用D3进行数据可视化"
categories: blog
tags: ['数据可视化', 'D3']
published: true
comments: true
script: [post.js]
---

## 概述

d3的英文含义是 `Data-Drive Document`，翻译过来就是数据驱动文档。
> 数据驱动，哇，好耳熟！Vue不就标榜是数据驱动吗？React不就改变了前端开发模式，界面的变化由直接dom操作发展为数据驱动。
数据驱动就是界面的表现会跟着数据的变化自动变化。

看看d3官方定义

> D3 (or D3.js) is a JavaScript library for visualizing data using web standards. D3 helps you bring data to life using SVG, Canvas and HTML. D3 combines powerful visualization and interaction techniques with a data-driven approach to DOM manipulation, giving you the full capabilities of modern browsers and the freedom to design the right visual interface for your data.

> D3(或者D3.js)是一个使用web标准由数据驱动的JavaScript库。D3帮助用户使用SVG，Canvas和HTML将数据变得生动形象。D3把强大的可视化和交互技术与数据驱动的DOM操作方法结合起来，为用户提供了现代浏览器的全部功能和为用户的数据设计正确的可视界面的自由。

D3的api和jQuery很像，你在使用中会发现，有选择器，有链式调用，它可以把数据和HTML结构或者SVG文档对应起来，当然现在也支持Canvas。D3有丰富的数学函数来里数据转换和物理计算，擅长操作SVG中的路径(path)和几何图形(circle,ellipse,rect...)。

还有很重要的一点，学习d3，首先要学习一些svg知识。

## 版本差异(v3与v4)

最新的版本是v4，网上的教程多是v3版本的，之间差异比较大。所以你如果使用的是v4，而你学习的示例使用的是v3，就要多加小心了，一模一样的代码运行时很容易报错，你需要查看 [官方文档](https://github.com/d3/d3)/ [中文文档](https://github.com/tianxuzhang/d3.v4-API-Translation#selecting-elements) 来替换相应的方法或属性。

```
// 创建坐标轴
d3.svg.axis().orient('bottom') // v3
d3.axisBottom() // v4

// 创建线性比例尺
d3.scale.linear() // v3
d3.d3.scaleLinear() // v4
```

## 实现柱状图

```
<!doctype>
<html>
    <head>
        <style>
            .bar-item{
                display: inline-block;
                width: 20px;
                background-color: blue;
                margin-right: 3px;
            }
        </style>
    </head>
    <body>
        <div id="bar"></div>
        <script src="d3.v4.js"></script>
        <script>
        
            d3.select('bar')
                .selectAll('div')
                .data([10,20,30,40,50])
                .append('div')
                .attr('class', 'bar-item')
                .style('height', function (d) {
                    return d + 'px'
                })
        
        </script>
    </body>
</html>
```

![](http://jsummer.me/assets/images/posts/bar.png)

通过上面简短的代码，浏览器打开，你会发现一个简单的柱状图已经生成了。这个图使用的是html元素。那用svg呢？

```
<svg width="500" height="300" id="bar-svg"></svg>
```

```
d3.select('#bar-svg')
      .selectAll('rect')
      .data(data)
      .enter()
      .append('rect')
      .attr('x', function (d, i) {
        return (20 + 4) * i 
      })
      .attr('y', function (d, i) {
        return 100 - d
      })
      .attr('width', 20)
      .attr('height', function (d, i) {
        return d
      })
```


具体看下代码，看d3做了什么？

```
d3.select('#bar')  // 选择id为bar的元素
.selectAll('div')  // 返回一个div标签的选集(selection)
.data([10,20,30,40,50])  //选集(selection)使用data方法将数据和div标签绑定
.enter()  //待绑定的数据
.append('div')  // 添加div元素
.attr('class', 'bar-item') // 给div元素添加类名
.style('height', function (d) {
    return d + 'px'
})  //给div元素设置高度
```

这个例子很简单，但体现了d3的数据绑定和元素生成的过程，数据绑定的原理以后分享。

下面有两张网图，一个是必备技能，一个是D3有哪些功能：

__必备技能__
![](http://img.blog.csdn.net/20160527215016188?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

__D3有哪些功能__
![](http://img.blog.csdn.net/20160522221307682?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### D3系列内容，敬请期待

[使用D3开发柱状图]()

[使用D3开发折线图]()

[使用D3开发饼图]()

[使用D3开发桑基图]()

[使用D3开发二叉树]()
