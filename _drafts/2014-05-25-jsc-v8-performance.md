---
layout: post
title: JavaScriptCore vs. V8性能测试
tags: [Google V8, JavaScriptCore]
keywords: JavaScriptCore V8性能测试对比, JavaScriptCore性能测试, V8性能测试
description: 通过测试数据来分析JavaScriptCore和V8性能差异，并通过JavaScript和V8的具体内部实现原理来讲解二者之间性能差异的原因
comments: true
---

&emsp;&emsp;近几年随着Google V8的崛起，关于JavaScript引擎性能的争论就没有停止过。随着越来越多的应用将JavaScript引擎替换成V8，JavaScriptCore似乎处在一个孤军奋战的状态，无论是从接口易用性还是从性能方面，饱受诟病和摈弃。之前参加ARM的一个关于性能优化的会议的时候，当ARM技术同学听到还有人在使用JavaScriptCore的时候，感到非常的差异。那么JavaScriptCore是不是大家想想中的那么差呢？V8是不是就已经在各个方面超越了JavaScriptCore呢？关于这个话题，我也想了很久，一直都觉得对于JavaScriptCore和V8理解的不够透彻，不敢妄加评判。刚好今天周末杭州下雨，也没有其他的好的去处，打算好好的分析下这个问题。

&emsp;&emsp;虽然JavaScriptCore和V8都分别推出了自己的跑分工具，但我个人认为这个跑分工具意义不是很大，而且各家都针对自己的特性来修改跑分工具。另外，业界也有很多人站出来批判，说单纯的测试JavaScriptCor和V8的性能没有意义，因为这些测试都只是单纯的测试JavaScript在不同引擎上的performance，而对于浏览器而言，渲染以及DOM操作才是真正影响用户体验的。对于这一点，我也不太赞同。因为现在已经有越来越多的应用和框架将JavaScript引擎从浏览器中剥离开来，特别是在Google V8崛起之后，JavaScript引擎开始逐渐独立出来，而不是非要跟浏览器绑定，例如Nodejs。在JavaScript引擎独立方面，JavaScriptCore就显得比较逊色了，目前JavaScriptCore还是跟WebKit绑定的非常紧密，甚至有些重要的特性需要依赖于WebKit，例如JavaScriptCore的GC机制，使得JavaScriptCore剥离出来比较困难，还需要修改源码，而V8则显得非常独立。

&emsp;&emsp;对于JavaScript引擎的性能最终表现在时间和内存消耗。时间的消耗一方面是JS的一些运算逻辑，以及函数调用。通过一些压力测试，对于JavaScriptCore和V8在时间消耗上的性能对比图如下：   


![Alt text](/images/jsc-v8-performance.png)

* empty-loop
空循环100000次(毫秒)

* call-js-func
调用一个空的js函数100000次耗时(毫秒)

* call-native-func
调用native方法100000次耗时(毫秒)

从上面的对比图可以发现，除了调用native方法以外，其他几个V8都要好于JavaScriptCore.