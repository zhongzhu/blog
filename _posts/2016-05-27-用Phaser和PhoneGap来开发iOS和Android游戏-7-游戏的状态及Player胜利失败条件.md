---
title: 用Phaser和PhoneGap来开发iOS和Android游戏 - 7. 游戏的状态及Player胜利失败条件
date: 2016-05-27 10:13:54
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
[上一篇博客](http://zhongzhu.github.io/2016/05/09/%E7%94%A8Phaser%E5%92%8CPhoneGap%E6%9D%A5%E5%BC%80%E5%8F%91iOS%E5%92%8CAndroid%E6%B8%B8%E6%88%8F-6-%E8%AE%A9Player%E5%8A%A8%E8%B5%B7%E6%9D%A5/)我们让Player在游戏里动起来了，它可以跑跳，碰到Enemy后可以重头开始。看起来这已经是一个可以玩的小游戏啦。不过，还是有很多可以提高的地方，比如说：

- Player到底跑多少米了？目前没法知道。
- Player碰到Enemy就重新开始，感觉像无敌模式，玩起来没有一点点压力，不刺激。
- Player就算跑完全程了，也没有显示它胜利了啊。

今天我们就来解决这几个问题。

<!-- more -->

无敌模式不刺激的问题很好解决，我们可以赋予Player几条命，撞到Enemy减一条命，减到0就显示You Lost界面。

目前跑了多少米可以显示在游戏界面上，实时更新。

Player跑完全程后，显示You Win界面，对玩家的成绩做出表扬。

## 目标 ##

1. 显示当前跑了多少米
2. Player跑完全程后，显示You Win界面
3. Player的生命值，及其You Lose界面

所有的代码都在文件`learn_to_build_a_game_with_phaser/part5.html`里。你可以在[我的GitHub](https://github.com/zhongzhu/learn_to_build_a_game_with_phaser.git)找到源代码和用到的图片资源。

## 显示当前跑了多少米 ##
几乎所有的游戏都会在游戏界面显示游戏的当前状态。比如说：Player当前还有几条命、已经吃了几个金币、目前的得分、还剩多少时间等等。这些数据的显示通常会在游戏界面的上方，有点像Windows工具栏的位置。

游戏界还给这种显示取了个好听的名字：`HUD(head-up display)`。维基百科对HUD的解释：

> In video gaming, the HUD (head-up display) or Status Bar is the method by which information is visually relayed to the player as part of a game's user interface.

> The HUD is frequently used to simultaneously display several pieces of information including the main character's health, items, and an indication of game progression (such as score or level).

我们将把目前的米数显示在游戏界面的正上方。代码如下：

```Javascript
create: function() {
  // 设置游戏背景颜色
  // 显示Tile Map
  // 显示Enemy
  // 显示Player

  // 显示当前跑了多少米
  this.metersLabel = this.game.add.text(Math.round(this.game.width/2), 50, 
    '0m', {font: '40px Arial', fill:'#fff'});
  this.metersLabel.anchor.setTo(0.5, 0.5);
  this.metersLabel.fixedToCamera = true;

  // 每秒更新一次当前跑了多少米
  this.time.events.loop(Phaser.Timer.SECOND, function() {
    this.metersLabel.text = Math.round(this.player.x * 100 / this.world.width) + 'm';
  }, this);
}
```

第8-9行，创建`metersLabel`文本控件，位置在X坐标为游戏界面的正中间，Y坐标为50。文本的初始值为`0m`，表示`零米`。字体大小40px，字体颜色白色。

第10行，Anchor设为（0.5, 0.5）用来保证文字居中。

第11行，`fixedToCamera = true`很关键，它可以保证文字是跟着镜头走的。Player不停的往右跑，镜头一直跟随Player。如果文本控件不跟随镜头走，很快Player往右跑远了玩家就会看不见文字了。

第14-16行，利用`time.events.loop()`函数来循环调用一个回调函数。`loop()`的第一个参数是循环时间，`Phaser.Timer.SECOND`表示每秒调用一次。第二个参数是回调函数。回调函数计算当前的米数，并更新`metersLabel`文本控件的值。

本游戏虽然叫做`跑完一百米`，但是游戏场景不可能有一百米宽啦。所以我们用Player的X轴位置`player.x`来除游戏场景总宽度`world.width`，然后乘以100来模拟出当前的米数。

就这些代码，让我们来看看游戏有什么变化。

## 试试我们的代码 ##
现在打开Chrome浏览器，输入`http://localhost:8000/part5.html`可以看到如下界面：

![HUD meters](1.png)

Player在奔跑的同时，游戏界面正上方已经可以实时更新当前跑了多少米了。

## Player跑完全程后，显示You Win界面 ##
Player跑完全程后，我们会显示如下的You Win界面：

![You Win](2.png)

界面里有个`再跑一次`的按钮，玩家点击按钮可以重玩游戏。

### You Win State ###
在Phaser里，游戏的每个界面都用一个`Phaser State`来管理是一种很好的代码模块化方式。下面我们将写一个新的State`YouWin`，来显示上面的界面并完成界面点击的处理。

在`learn_to_build_a_game_with_phaser/part5.html`里加入如下代码：

```HTML
<!-- You Win State -->
<script>
  var YouWin = function () {};

  YouWin.prototype = {
    preload: function () {
      // 什么都不用做
    },
    create: function() {        
    },
  };
</script>

<!-- the main app 主程序-->
<script>
  var targetWidth = 840;
  var targetHeight = 560;
  var newWidth = (window.innerWidth/window.innerHeight) * targetHeight;

  var game = new Phaser.Game(newWidth, targetHeight, Phaser.CANVAS);
  game.state.add('Play', Play);
  game.state.add('YouWin', YouWin);
  game.state.start('Play');
</script>
```

第1-12行，定义`YouWin State`。

第22行，把`YouWin State`加载到`game.state`里备用。

### YouWin.preload() ###
`YouWin State`里需要的图片资源已经在`Play.preload()`里面用如下代码加载过了，所以`YouWin.preload()`里面是什么都不用做。

```Javascript
this.load.atlas('texture-atlas', 'assets/texture-atlas.png', 'assets/texture-atlas.json');
```

### YouWin.create() ###
代码如下：

```Javascript
create: function() {
  this.add.image(Math.round(this.game.width/2), Math.round(this.game.height/2), 
    'texture-atlas', 'youwin_bg').anchor.set(0.5);

  this.add.button(Math.round(this.game.width/2), Math.round(this.game.height/2) + 50, 
    'texture-atlas', 
    function() { this.state.start('Play'); }, 
    this,
    'restart', 'restart', 'restart', 'restart').anchor.set(0.5); 
}
```

第2-3行，显示背景图片`youwin_bg`。

第5-9行，显示按钮图片`restart`。

第7行是点击按钮后的回调函数，点击后用`state.start()`函数把游戏重新返回`Play`状态，也就是重玩游戏。

第9行有4个`restart`，其实这四个参数应该代表按钮在鼠标悬停、鼠标离开、按钮按下、按钮松开四个状态下的图片名字。我这里为了省事（主要还是不会用PS做图），四个状态都用`restart`这一张图片了。

就这些代码。现在打开Chrome浏览器，输入`http://localhost:8000/part5.html`玩玩游戏，当Player跑完全程后，你应该可以看到You Win界面了。点击`再跑一次`又可以重玩一次游戏。

## You Lose界面 ##
如前面所说，我们Player初始会有3条命，撞到Enemy就损失1条，到0就显示You Lose界面。

跟`You Win`一样，`You Lose`界面我们也用一个`YouLose State`来管理，代码如下：

```HTML
<script>
  var YouLose = function () {};

  YouLose.prototype = {
    preload: function () { 
      // 什么都不用做
    },
    create: function() {
      this.add.image(Math.round(this.game.width/2), Math.round(this.game.height/2), 
        'texture-atlas', 'youlose_bg').anchor.set(0.5);

      this.add.button(Math.round(this.game.width/2), Math.round(this.game.height/2) + 50, 
        'texture-atlas', 
        function() { this.state.start('Play'); }, 
        this,
        'restart', 'restart', 'restart', 'restart').anchor.set(0.5); 
    }
  };
</script>  

<!-- the main app 主程序-->
<script>
  var targetWidth = 840;
  var targetHeight = 560;
  var newWidth = (window.innerWidth/window.innerHeight) * targetHeight;

  var game = new Phaser.Game(newWidth, targetHeight, Phaser.CANVAS);
  game.state.add('Play', Play);
  game.state.add('YouWin', YouWin);
  game.state.add('YouLose', YouLose);
  game.state.start('Play');
</script>
```

代码和`YouWin State`几乎一模一样，就不细讲了。主需要注意第10行，背景图片的名字是`youlose_bg`。完成后的界面如下：

![You Lose](3.png)

## Player的生命值 ##
Player生命值的显示和前面跑了多少米的显示很类似。我们需要更新`Play.creaet()`和`Play.playerHit()`里面的代码。

### Play.create() ###
代码如下：

```Javascript
create: function() {
  // 设置游戏背景颜色
  // 显示Tile Map
  // 显示Enemy
  // 显示Player
  // 显示当前跑了多少米
  // 每秒更新一次当前跑了多少米

  // 显示Player头像和生命值
  this.lives = this.add.image(50, 50, 'texture-atlas', 'hud_p2');
  this.lives.anchor.set(0.5);
  this.lives.fixedToCamera = true;

  this.player.health = 3;  // 初始化3条命
  this.livesLabel = this.game.add.text(
    this.lives.x + this.lives.width + 12, this.lives.y,
    'x ' + this.player.health,
    {font: '40px Arial', fill:'#fff'});
  this.livesLabel.anchor.set(0.5);
  this.livesLabel.fixedToCamera = true;
}
```
第10-12行，显示Player头像`hud_p2`。注意第12行的镜头跟随。

第14行，Player初始化有3条命。

第15-20行，显示`livesLabel`文本控件，控件的值为第17行的`'x ' + this.player.health`。注意`x`只是为了显示好看，能显示成`x 3`、`x 2`分别表示还有3条命、两条命。当然，最后需要镜头跟随。

现在打开Chrome浏览器，输入`http://localhost:8000/part5.html`，你应该能看到Player生命值显示出来了。

![Player Lives](4.png)

### Play.playerHit() ###
现在我们来实现`撞到Enemy就损失1条，到0就显示You Lose界面`。需要更改`Play.playerHit()`：

```Javascript
playerHit: function(player, enemy) {
  this.player.animations.stop();
  this.player.x = 60;
  this.player.y = 150;
  this.player.body.velocity.x = 0;
  this.player.body.velocity.y = 0;
  this.player.body.blocked.down = false;

  this.player.health--;
  if (this.player.health > 0) {
    this.livesLabel.text = ' x ' + this.player.health;
  } else {
    this.state.start('YouLose');
  }
}
```

当Player撞到Enemy时，`Play.playerHit()`就会被调用。第9-14行是新加入的代码。

第9行，把player的生命值减一。

第10-14行，如果生命值大于0，更新`livesLabel`文本控件显示当前生命值；如果生命值为0，转到`YouLose State`界面去。

到这里，我们已差不多实现了游戏的基本功能，可以对大家宣布：游戏beta版已经完成了，尽情的玩吧！

## 今天就到这里了 ##
下一个博客讲讲如何加入游戏主菜单界面，还有，还有？还有音效！对啊，哪个游戏会是没有声音的呢 :-)






