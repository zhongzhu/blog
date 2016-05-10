---
title: 用Phaser和PhoneGap来开发iOS和Android游戏 - 4. Phaser基础知识
date: 2016-03-28 14:19:51
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
[上一篇博客](http://zhongzhu.github.io/2016/03/25/%E7%94%A8Phaser%E5%92%8CPhoneGap%E6%9D%A5%E5%BC%80%E5%8F%91iOS%E5%92%8CAndroid%E6%B8%B8%E6%88%8F-3-%E7%94%A8Tiled-Map-Editor%E6%9D%A5%E7%94%9F%E6%88%90%E6%B8%B8%E6%88%8F%E5%9C%BA%E6%99%AF/)我们用[Tiled](http://www.mapeditor.org/)这个工具来生成了游戏场景。今天终于要开始写代码了，我们将开始使用Phaser把游戏场景文件（tiles.png, level.json）读出来，显示在手机上。

下面用到的所有图片和代码都可以在[我的GitHub](https://github.com/zhongzhu/learn_to_build_a_game_with_phaser.git)找到。

## 目标 ##

1. 用Phaser把游戏场景显示在手机浏览器上

## 准备工作 ##
在写代码前，需要准备的东西：

- Windows PC
- 在Windows PC上安装
	- [Google Chrome](https://www.google.com/chrome/)浏览器
	- [Python 2.7.x for Windows](https://www.python.org/downloads/)

<!-- more -->

Chrome浏览器用来模拟手机。
	
安装Python不是因为要写Python代码。Phaser工程其实就是一个网站，所以在调试程序的时候需要一个HTTP Server才能把Phaser工程跑起来。Python内置的SimpleHTTPServer非常好用，零配置，推荐用它。

你可以用任何文本编辑器写代码，我用的是[Sublime Text](https://www.sublimetext.com/3)。

## Phaser工程的目录结构 ##
请先到[我的GitHub](https://github.com/zhongzhu/learn_to_build_a_game_with_phaser.git)下载源代码，下面的讲解将基于这个源代码。

Phaser工程的主目录如下：

![phaser project folder](1.png)

- `part1.html, part2.html` - 游戏的HTML网页。为了简化我把JavaScript代码写在HTML文件里了。每个HTML文件都是本教程的一个阶段性`MVP(Minimum Viable Product)`，可以用来运行和展示。随着教程的进行，还会增加part3.html、part4.html等等
- `index.html` - 现在还用不到它。等整个教程结束，我会做一次重构，把part1.html/part2.html/...的JavaScript代码提取出来放入`js目录`，剩下的真正的HTML的内容放入index.html
- `js目录` - 所有的JavaScript文件
- `assets目录` - 图片，tile map文件，声音

## Phaser工程的代码结构 ##
现在打开文件`learn_to_build_a_game_with_phaser/part1.html`，我们来看看代码，这是一个很常见的HTML5网页:

```HTML
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="user-scalable=no, initial-scale=1, maximum-scale=1, minimum-scale=1, width=device-width, height=device-height" />
  <title>跑完一百米！</title>
  <style type="text/css"> body { margin: 0; }</style>
  <script src="js/phaser.min.js"></script>
</head>

<body>
  <!-- the game logic 游戏逻辑-->
  <script>
    var Play = function () {};

    Play.prototype = {
      preload: function () {},
      create: function() {},
      update: function() {}
    };
  </script>

  <!-- the main app 主程序-->
  <script>
    var targetWidth = 840;
    var targetHeight = 560;
    var newWidth = (window.innerWidth/window.innerHeight) * targetHeight;

    var game = new Phaser.Game(newWidth, targetHeight, Phaser.AUTO);
    game.state.add('Play', Play);
    game.state.start('Play');
  </script>
</body>
</html>
```

第8行引入了Phaser的库文件，就一个文件`phaser.min.js`，那就是Phaser所有的东西啦，够简单吧？

和Phaser相关的全部代码都放在`<body></body>`之间。我用两个`<script></script>`来放两段不同功能的JavaScript代码。这样有点啰嗦，不过好处是看起来清晰，而且最后重构的时候代码很容易分模块。

## 初始化Phaser的Game对象 ##

先来看23行-32行。这几行是程序的入口，类似C语言的main函数。

第29行初始化了Phaser的`Game对象`。

```JavaScript
var game = new Phaser.Game(newWidth, targetHeight, Phaser.AUTO);
```

Game()函数的第三个参数指定Phaser用什么Web技术来画游戏的界面。Phaser支持HTML [Canvas](http://www.w3schools.com/html/html5_canvas.asp)或者[WebGL](https://en.wikipedia.org/wiki/WebGL)两种方式。Canvas几乎所有的手机浏览器都支持；WebGL动画性能更好，不过有些浏览器不支持。AUTO模式让Phaser自己决定用前面两种方式中的一种。

Game()函数的前两个参数指定游戏界面的宽和高。我们的游戏界面为`840X560`，由于各种手机的屏幕宽高比不一样，第27行根据手机浏览器实际宽`window.innerWidth`高`window.innerHeight`比算出了游戏界面的新的宽`newWidth`，游戏的高`targetHeight`保持560px不变。

```JavaScript
var targetWidth = 840;
var targetHeight = 560;
var newWidth = (window.innerWidth/window.innerHeight) * targetHeight;
```

根据手机实际宽高比重新计算游戏的宽高非常重要，如果不这样做，你的游戏在不同的手机屏幕上显示就会失真（图形拉伸，或者被压缩变形）。

第30行，用game.state加载了一个叫做`Play`的`State`。加载只是把State放入内存中，第31行start()才真正让这个State运行起来。

```JavaScript
game.state.add('Play', Play);
game.state.start('Play');
```

那什么是State呢？下面我们详细讲讲State这个很有用的概念。

## Phaser的State对象 ##
Phaser的`State`可以被翻译成状态机里的`状态`。State是个很有用的概念。

在写代码的时候，有面向对象编程经验的人都会想到把把代码封装成类（Class）：主菜单界面一个类，游戏实际的运行一个类，游戏结果的显示（You Win! You Loose!）一个类。要让这些类的对象相互访问，而且能访问Phaser的实时信息，你恐怕得多做一些公共代码的工作。而Phaser的State对象帮你把这些都实现了：

- 状态的加载`game.state.add()`
- 状态的运行/转换`game.state.start()`
- 直接在状态对象里用`this.xxx`的方式访问Phaser的核心对象：`game`, `input`, `camera`, `sound`, `physics`, 等等

如何才能生成一个State对象呢？代码第14-20行定义了一个叫做`Play`的State对象，及其它的原型框架。

```JavaScript
var Play = function () {};

Play.prototype = {
  preload: function () {},
  create: function() {},
  update: function() {}
};
```

很简单，State就是一个函数对象，它有几个约定的方法可以在State的原型(prototype)里实现:

- `preload` Phaser会在游戏初始化阶段调用它，一般我们用它来加载游戏资源(assets)。如：图片、声音等等
- `create` preload被调用结束后，Phaser会调用create。我们一般在这个方法里把内存里的图片放置到游戏界面里，设置好Player和enemy
- `update` 每秒钟这个update会被Phaser调很多次，所以所有游戏精灵们之间的互动都在这里处理。比如：Player跳跃，Player撞到Enemy，等等

通常，一个Phaser游戏工程里会写多个State对象，比如：Load state用来把图片、sprite sheet、声音加载到内存，同时显示一个逐渐变长的progress bar。Menu state用来显示游戏的主菜单，让玩家选关或者设置。Play state用来运行游戏逻辑。End state用来显示You Win! You Loose!之类的东西。

为了简化，我们就一个Play state。

## Play.preload方法 ##
现在打开文件`learn_to_build_a_game_with_phaser/part2.html`，我们来看看preload()方法里面写了些什么。

```Javascript
var Play = function () {};

Play.prototype = {
  preload: function () {
    this.scale.scaleMode = Phaser.ScaleManager.SHOW_ALL;
    this.scale.pageAlignHorizontally = true;
    this.scale.pageAlignVertically = true;

    this.load.tilemap('tilemap', 'assets/level.json', null, Phaser.Tilemap.TILED_JSON);
    this.load.image('tiles', 'assets/tiles.png');
    this.load.atlas('texture-atlas', 'assets/texture-atlas.png', 'assets/texture-atlas.json');
  },
  create: function() {},
  update: function() {}
};
```

第1行，声明了Play state函数对象。

第5行，设置了本游戏界面的缩放模式`scaleMode`为`Phaser.ScaleManager.SHOW_ALL`。

第6-7行是让游戏界面居中显示。

第9-11行，把游戏需要的图片和tile map资源加载到内存。

短短十来行代码，我花了近10天才搞清楚了要这样写。为什么呢？因为这里牵涉了Phaser游戏开发的三个知识点：

1. 手机屏幕适配
2. Tile Map
3. Texture Atlas

下面详细讲讲这三个知识点。

## 手机屏幕适配 ##
一个游戏当然要在各种手机的屏幕上都显示正常，要不然别人怎么会玩你的游戏？这个要求很正常，不过分吧？

答案是：太过分了。

鉴于现在手机厂家多如牛马，每个厂家几个月就出一款长相奇特的新机，要做到一个游戏适配所有手机屏幕基本是没戏。我也是花了差不多一个星期才搞清楚了应该怎么做：太复杂了，别妄想了，干脆就用最简单的方法适配`大部分手机`就好。

下图就是本游戏做手机屏幕适配的方法：

![game scale](4.png)

统共分4步:

1. 给游戏预设一个界面大小840x560，所以宽高比为3:2
2. 游戏代码里动态获得手机实际的宽高比（宽：window.innerWidth，高：window.innerHeight）。比如iPhone5，宽高568x320,宽高比16:9
3. 修正预设的大小，高度不变560，高度按16:9换算成1493。现在我们的游戏有新的界面大小：1493x560，宽高比16:9
4. 虽然宽高比我们和iPhone5一致了，但1493x560比568x320大很多啊。没有关系，用Phaser的缩放模式`this.scale.scaleMode = Phaser.ScaleManager.SHOW_ALL`把1493x560保持宽高比的缩小，直到能放入iPhone5的屏幕里。

看似简单的4步，我花了5个晚上的时间google加看帮助才理清了这样是可行的。

## Tile map ##
Tile map的概念在[上一个博客](http://zhongzhu.github.io/2016/03/25/%E7%94%A8Phaser%E5%92%8CPhoneGap%E6%9D%A5%E5%BC%80%E5%8F%91iOS%E5%92%8CAndroid%E6%B8%B8%E6%88%8F-3-%E7%94%A8Tiled-Map-Editor%E6%9D%A5%E7%94%9F%E6%88%90%E6%B8%B8%E6%88%8F%E5%9C%BA%E6%99%AF/)已经讲过了。

Tile map包括两部分：一个是包含所有小tile图片的大图片tile.png，另一个是描述哪种小tile图片应该放在哪个小格子的描述文件level.json。我们需要用下面的`this.load.tilemap()`和`this.load.image()`方法把这两个资源加载到内存。

这两个函数很类似，第一个参数是给被加载的资源取个名字。以后再访问这个资源用这个名字就好了。第二个参数是资源的文件名。

```Javascript
this.load.tilemap('tilemap', 'assets/level.json', null, Phaser.Tilemap.TILED_JSON); 
this.load.image('tiles', 'assets/tiles.png');  
```

## Texture Atlas ##
Tile Map用的tile.png图片是`sprite sheet`，它把大小一样的小tile图片全部放入了一个大图片文件tile.png中了。

能不能把大小不一样的图片也像`sprite sheet`那样放到一个大图片文件里来提高性能？可以，就是用`Texture Atlas`方式。

Texture Atlas类似Tile Map也包括两部分：一个当然是那个大图片文件，在本游戏就是texture-atlas.png；另一个就是用来描述哪个小图片放入了texture-atlas.png的描述文件texture-atlas.json。texture-atlas.json里面包含了每个小图片的名字，我们后面将用这些名字来访问texture-atlas.png里的各个小图片。

下图就是texture-atlas.png:

![texture-atlas](texture-atlas.png)

下图是texture-atlas.json的内容，每个`filename`后面跟着的就是小图片的名字。

![texture-atlas.json](5.png)

用如下代码就可以把Texture Atlas加载到内存：

```Javascript
this.load.atlas('texture-atlas', 'assets/texture-atlas.png', 'assets/texture-atlas.json')
```

第一个参数是给Texture Atlas资源取的一个名字。第二个参数是大图片文件名。第三个参数是json文件名。

## 休息一下 ##
这篇博客我们讲了Phaser工程的大概样子，并详细讲了Play.preload()方法及其屏幕适配等，下一个博客我们接着讲Play.create()。