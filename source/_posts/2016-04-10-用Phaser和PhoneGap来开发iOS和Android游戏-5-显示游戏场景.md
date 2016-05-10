---
title: 用Phaser和PhoneGap来开发iOS和Android游戏 - 5. 显示游戏场景
date: 2016-04-10 19:48:31
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
[上一篇博客](http://zhongzhu.github.io/2016/03/28/%E7%94%A8Phaser%E5%92%8CPhoneGap%E6%9D%A5%E5%BC%80%E5%8F%91iOS%E5%92%8CAndroid%E6%B8%B8%E6%88%8F-4-Phaser%E5%9F%BA%E7%A1%80%E7%9F%A5%E8%AF%86/)讲在Play.preload()中把游戏需要的资源（图片，tile map文件等等）加载到内存中，今天我们将把内存中的资源显示到手机浏览器屏幕上。也就是说，今天我们终于可以看到我们的游戏界面了。(TP:FB6E4FA1)

下面用到的所有图片和代码都可以在[我的GitHub](https://github.com/zhongzhu/learn_to_build_a_game_with_phaser.git)找到。

## 目标 ##

1. 用Phaser把游戏场景显示在手机浏览器上

## 用Chrome来模拟手机浏览器 ##
为了测试方便，我们在PC上用Chrome浏览器来模拟手机浏览器。
<!-- more -->

![chrome to simulate mobile](1.png)

打开Chrome之后，按上图的步骤就可以模拟手机浏览器。

1. 按`f12`打开`开发者工具`
2. 点击图标转换成`Device Mode`
3. 选择`手机类型`。比如iPhone 5/iPhone 6，不同手机类型的分辨率/宽高比不一样，正好可以用来测试上篇博客我们的屏幕适配是不是有效
4. 默认情况显示的手机屏幕是竖屏的，我们的游戏是横屏，所以需要点击`Rotate`图标把屏幕改成横屏
5. 这里的区域就是手机屏幕啦

现在我们试试看能不能把上一个博客写的代码`learn_to_build_a_game_with_phaser/part2.html`在Chrome里面显示出来。要显示网页，我们还需要一个HTTP Server。

## 用Python来做HTTP Server ## 
假设大家已经安装好Python了。我们试试看能不能显示上次的网页`part2.html`。操作如下：

1. 把代码从GitHub clone到本地

	```
	C:\> git clone https://github.com/zhongzhu/learn_to_build_a_game_with_phaser.git
	```

2. 执行如下两行命令

	```
	cd .\learn_to_build_a_game_with_phaser
	python -m SimpleHTTPServer
	```

	如果看到如下提示，就表示HTTP Server起来了，在监听8000端口。

	> Serving HTTP on 0.0.0.0 port 8000 ...

3. 转到Chrome浏览器，输入`http://localhost:8000/part2.html`应该可以看到如下界面

	![part2.html](2.png)
	
为什么是一片黑色？上一篇博客我们只讲到了如何把游戏资源加载到内存还没把它们显示出来，所以屏幕上什么都没有。下面就开始讲代码，看如何显示。

## 代码框架 ##
这次的代码在`learn_to_build_a_game_with_phaser/part3.html`文件里。代码的框架如下

```Javascript
  <!-- the game logic 游戏逻辑-->
  <script>
    var Play = function () {};

    Play.prototype = {
      preload: function () {
        // ...
      },
      create: function() {
        // 设置游戏背景颜色
        // 显示Tile Map
        // 显示Enemy
        // 显示Player
      },
      update: function() {}
    };
  </script>
```

显示的工作都放入了Play.create()，总共有四个任务：

1. 设置游戏背景颜色
2. 显示Tile Map
3. 显示Enemy
4. 显示Player

## 设置游戏背景颜色 ##
很简单，就是把`舞台`stage的backgroundColor属性设为颜色的16进制值就好。

```Javascript
// 设置游戏背景颜色
this.stage.backgroundColor = '#A6E5F5';
```

这里多说两句Phaser的`舞台stage`和`世界world`的关系和区别。

Phaser的`world`指的是整个游戏世界，而玩家能看到的部分都在`舞台stage`里面。你可以想象stage是一个空相框，而world是一张很大的地图，玩家用相框放在大地图上，自己能看到的部分（显示在手机上的部分）就是stage的内容，只要玩家在地图上移动相框，就能慢慢看到所有的地图内容。

当然，有的游戏（比如：2048，珠宝消除）整个游戏世界都显示在手机屏幕上，这种情况下Phaser的stage和world的物理大小是一样的。

## 显示Tile Map ##
如下，显示Tile Map总共就6行代码。

![add tile map](3.png)

第31行，利用add.tilemap()方法生成了tilemap对象this.map。函数参数`tilemap`就是代码第22行的`tilemap`，那个JSON文件。

```Javascript
this.map = this.add.tilemap('tilemap');
```

第32行，把tilemap需要的图片资源加载进来。函数有两个参数，第二个是代码第23行的图片资源`tiles`，第一个是我们在用Tiled Map Editor编辑游戏场景时给tileset取的名字，如下：

![tileset name](4.png)

第34,35行，this.map.createLayer()方法创建了`BackgroundLayer`和`GroundLayer`，并把这两个Tile Map Layer创建并显示在了手机屏幕上。

这两个Layer的名字是我们在Tiled Map Editor里面设置的，如下：

![layer names](5.png)

第36行，setCollisionBetween()设置我们的Player会和`GroundLayer`的哪些tile图片`碰撞`。

```Javascript
this.map.setCollisionBetween(1, 35, true, 'GroundLayer');
```

这个`碰撞`是个很重要的概念，如果不`setCollisionBetween()`，那Player在`重力`的作用下就会像特异功能一样穿过我们的`GroundLayer`(那些草地啊，桥啊，反正Player用来走的)，直接掉下屏幕最下方消失不见。

`setCollisionBetween()`的头两个参数是tile小图片的`下标index`。下面的图就是我们所有的tile小图片了，下标从1开始，从上到下从左到右累加。比如：那个笑脸的绿色不明生物的下标就是4。

![tiles](tiles.png)

`setCollisionBetween()`的第三个参数表示我们要`碰撞`，第四个参数表示这个函数只对`GroundLayer`起作用。`BackgroundLayer`是用来做背景的，所以不需要和Player有什么`碰撞`了。

第38行，我们按`GroundLayer`的实际大小重置了`world`的大小。

```Javascript
this.groundLayer.resizeWorld();
```

前面已经讲过`stage`和`world`的概念。最开始我们生成Game对象的时候，设置的大小是840x560，所以当时`stage`和`world`的大小一样都是840x560。

```Javascript
var game = new Phaser.Game(840, 560, Phaser.AUTO);
```

不过，我们用Tiled Map Editor编辑的游戏场景可比这个大多了，有2310x560这么大。调用`resizeWorld()`后，`stage`也就是我们看世界的窗口依然是840x560，但是我们的世界`world`已经变成2310x560了。

## 显示Enemy ##
游戏里的`敌人Enemy`只有一种：很凶的绿色的盒子。下面的代码介绍了如何生成敌人。

```Javascript
this.enemyGroup = this.game.add.group();
this.enemyGroup.enableBody = true;
this.map.createFromObjects(
  'ObjectLayer', 'box', 
  'texture-atlas', 'blockerMad',
  true, false, this.enemyGroup);
```
第1行，利用add.group()生成了一个this.enemyGroup`组Group`。如下图所示，游戏里有很多的`Enemy`，如果要一个一个去生成，那太麻烦了。而且，每个Enemy和Player之间的关系都一样（Player撞到Enemy就重头来过），所以把所有的Enemy放在一个组里统一对待是个很好的选择。

![enemies](6.png)

第2行，Group的enableBody设为true。这是给Group里的所有Enemy都加上`可以物理碰撞身体（physics body）`。正如前面`GroundLayer`加上`Collision`一样，给Enemy加上`Body`才会让Phaser知道`Player`和`Enemy`可以碰撞并通知游戏程序来处理。假如没有`Body`，`Player`就会径直穿过`Enemy`的身体，看起来就像科幻动画了。

第3-6行，用`Tilemap.createFromObjects()`方法创建所有的`Enemy`对象并显示在手机屏幕上。这个方法有7个参数，我们一个一个来讲讲。

![createFromObjects](7.png)

`'ObjectLayer', 'box'`：看上图，`ObjectLayer`是前面用Tiled Map Editor创建的放各种精灵的层，`ObjectLayer`里的所有Enemy都给取名叫`box`。

`'texture-atlas', 'blockerMad'`： `texture-atlas`是Play.preload()里加载到内存的`texture-atlas.png`的名字。`blockerMad`是Enemy小图片在`texture-atlas.json`里的名字。

这4个参数的意思就是：把tile map里`ObjectLayer`上叫做`box`的对象全部用`texture-atlas.png`里叫`blockerMad`的小图片来绘制到屏幕上。

`this.enemyGroup`：把创建出来的对象都加到enemyGroup组里。

## 显示Player ##

```Javascript
this.player = this.add.sprite(60, 150, 'texture-atlas', 'alienBlue_walk2');
this.player.anchor.setTo(0.5, 0.5);
this.player.animations.add('walking', ['alienBlue_walk1', 'alienBlue_walk2'], 5, true);

this.physics.arcade.enable(this.player);
this.player.body.gravity.y = 1000;
this.camera.follow(this.player);
```

第1行，把`精灵(sprite)`显示在坐标`X轴：60`, `Y轴：150`。我们在`texture-atlas.png`里保存了好几帧和`Player`相关的图片。初始化用的是如下的`alienBlue_walk2`。

![player picture](8.png)

第2行，`anchor`是sprite图片的基准点，如果把anchor设为`(0,0)`表示用图片的左上角做基准点把图片放到`stage`上。设为`(0.5,0.5)`表示用图片的中心作为基准点。下面的图展示了不同anchor值，sprite在stage上摆放位置的不同。

![anchor](9.png)

第3行，`player.animations.add()`给`Player`增加动画效果。动画取名叫`walking`，利用了`alienBlue_walk1`和`alienBlue_walk2`两帧图片循环播放。播放速度`5帧/秒`。这里add()只是生成了动画，还没有正式开始播放。需要后面`player.animations.play()`动画才真正能动起来。

第5行，给`Player`加上物理特性。这样它就会有`Body`及其各种`重力`、`碰撞`、`速度`等等。

第6行，Player的`Y轴重力加速度`为1000。这样Player就会像真实的世界一样，从屏幕的(60,150)坐标往下落。当然，他不会落到底，他会和`GroundLayer`碰撞，然后站在GroundLayer上。

第7行，让游戏的`镜头(camera)`跟随Player。前面讲到world很大，stage只是玩家能看到的一个小相框里的东西，Player在游戏里奔跑时我们就得横着移动小相框让玩家能看到Player当前所在地点的地图。在Phaser里，没有移动相框的函数，而是移动镜头，让镜头跟随Player。

## 试试我们的代码 ##
现在打开Chrome浏览器，输入`http://localhost:8000/part3.html`应该可以看到如下界面：

![collision not working](10.png)

等等，为什么Player还是穿过`GroundLayer`掉到屏幕下方去了？其实我们还少写列两行代码。

## 最后的两行代码 ##
现在让我们来加入最后的两行代码。

![fix collision issue](11.png)

第26行，让游戏用`Arcade`物理系统。Phaser支持好几种物理系统，比如`Arcade`、`P2`和`Ninja`。其中以Arcade最简单，它只能判断方形物体的碰撞。如果要做`愤怒小鸟`之类的游戏，就牵涉多边形/圆形/物体旋转时的碰撞，那就得需要更高级的P2或者Ninja物理系统才行了。我们的碰撞都是方形的，用Arcade简单实用。

第59-61行，我们实现了一个新的方法`Play.update()`。一般讲游戏的性能都说`FPS(Frame Per Second)`，每秒多少帧。如果游戏FPS=30，那Phaser每秒钟就会调用`Play.update()`30次。我们一般会在update()里加入各种碰撞测试和碰撞成功后的回调函数。

第60行，`physics.arcade.collide()`检测`Player`和`GroundLayer`之间的碰撞。如果检测到了碰撞，Player就会站立在GroundLayer上，而不是掉到屏幕下方去。

## 再试试我们的代码 ##
现在打开Chrome浏览器，输入`http://localhost:8000/part3.html`应该可以看到如下界面：

![collision issue fixed](12.png)

看来我们已经成功了，不是吗？Player稳稳地站在了草地上，耶！

## 今天就到这里了 ##
这篇博客我们讲了如何把游戏场景显示在手机浏览器上，下次我们讲如何让Player跑和跳。