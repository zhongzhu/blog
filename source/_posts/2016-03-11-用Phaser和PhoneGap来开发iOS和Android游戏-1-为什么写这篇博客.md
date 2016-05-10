---
title: 用Phaser和PhoneGap来开发iOS和Android游戏 - 1. 为什么写这篇博客
date: 2016-03-11 15:12:31
tags:
- Phaser
- PhoneGap
- Cordova
- iOS
- Android
categories:
- APP开发
- 游戏
---
两个月前看到[Phaser](http://phaser.io)这个很简洁漂亮的HTML5 Javascript游戏框架后便有了在业余时间做一个手游玩玩的念头。一个月前也开始陆陆续续的照着[Phaser官网的教程](http://phaser.io/tutorials/making-your-first-phaser-game)学习Phaser。

为什么要写这篇博客呢？主要是为了治愈目前的拖延，督促自己完成目标。而且：从头到尾设计一个游戏，进行编码并最终在AppStore上线是一件很好玩的事情。不是嚒？“从AppStore下载自己写的游戏来玩”，想想就觉得很酷 :-)

当然，如果你能从中受益，开发出属于你自己的游戏，那就更好了！

## 目标 ##

 1. 用Phaser开发一个网页游戏
 2. 用PhoneGap把游戏打包成Android APP
 3. Android APP在Google Play上线
 4. 用PhoneGap把游戏打包成iOS APP
 5. iOS APP在AppStore上线

<!-- more -->

## Phaser和PhoneGap简介 ##
先讲讲我们要用到的两个重要工具：Phaser和PhoneGap。

`Phaser`是一个游戏框架，可以用来开发桌面或者手机HTML5网页游戏，开发语言支持JavaScript/TypeScript。这里我们用JavaScript，这个Internet时代最流行的语言（我不会TypeScript）。

用Phaser开发出来的其实是一个HTML5网页游戏或者叫做Web APP，要在AppStore或者Google Play上线必须用原生APP的方式。如果要用Objective-C/Swift来写一个iOS APP，然后再用Java写一个Android APP才能上线我们的游戏，那成本太高了，维护两套不同的代码也是很头痛的。于是，我们需要PhoneGap。

`PhoneGap`其实就是一个打包软件，它可以把你用HTML/CSS/JavaScript写的网页应用（Web APP）直接打包成iPhone/Android/WindowsPhone原生APP。一套代码，直接生成多种平台的原生APP，相当的厉害！

## 准备工作 ##
APP开发当然用MacBook最好，因为iOS游戏APP必须在苹果电脑上打包，此外MacBook也可以打包Android APP。如果没有MacBook也无所谓，准备一台能上网的Windows PC就可以了。用Windows PC我们至少可以做到开发Android APP并上线（目标的第3步）。

你需要准备

- Windows PC
- 在Windows PC上安装[Python 2.7.x for Windows](https://www.python.org/downloads/) （用来做HTTP server）

## 要把游戏开发出来，统共分几步？ ##
其实我也不知道总共分几步。

我可以说是个游戏开发门外汉，多年一直开发测试工具，对游戏这行还真是不了解。门外汉也有门外汉的优势：有自知之明，所以肯定会做一个简单得不能再简单的游戏。“简单”有一个致命的优势：也许你真的能把它做出来！

好吧，回到原来的话题：统共分几步？我觉得分三步：

1. **策划**：这是个什么类型的游戏？游戏的故事是什么？
2. **图片，音效**：没有图形和声音就不是游戏了
3. **写代码**

到这里可能有人会说了：“少了一步吧？没有测试！”。作为系统测试部门专门开发测试工具的资深员工怎么会忘记测试？测试包括在“3. 写代码”里面了，写了代码要测试是对程序员的基本要求，DoD(Definition of Done)，不是嚒？:-)

这三步，我觉得最难的是策划。估计策划要花掉30%到50%的开发时间。好了，今天就写到这里，下一篇博客写游戏的策划。