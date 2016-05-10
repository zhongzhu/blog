---
title: 用Phaser和PhoneGap来开发iOS和Android游戏 - 6. 让Player动起来
date: 2016-05-09 15:20:45
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
[上一篇博客](http://zhongzhu.github.io/2016/04/10/%E7%94%A8Phaser%E5%92%8CPhoneGap%E6%9D%A5%E5%BC%80%E5%8F%91iOS%E5%92%8CAndroid%E6%B8%B8%E6%88%8F-5-%E6%98%BE%E7%A4%BA%E6%B8%B8%E6%88%8F%E5%9C%BA%E6%99%AF/)我们把游戏场景显示出来了，Player也能稳稳的站在`GroundLayer`草地上。今天我们来让Player动起来。

所有的代码都在文件`learn_to_build_a_game_with_phaser/part4.html`里。你可以在[我的GitHub](https://github.com/zhongzhu/learn_to_build_a_game_with_phaser.git)找到源代码和用到的图片资源。

## 目标 ##

1. Player走
2. Player跳
3. Player撞到Enemy后重新开始

<!-- more -->

## 让Player走起来 ##
为了让游戏简单易玩，我们的游戏只有一种控制方式：玩家点击屏幕让Player跳起来躲开Enemy。

`Player走`这个动作就不需要玩家控制了，当Player站在`GroundLayer`上时，我们给它设一个速度。代码如下：

```Javascript
update: function() {
  this.physics.arcade.collide(this.player, this.groundLayer);

  if (this.player.body.blocked.down) {
    this.player.body.velocity.x = 260;
    this.player.animations.play('walking');
  }
}
```

第4行，`this.player.body.blocked.down`用来检测Player是否站站在`GroundPlayer`上。只有当Player站着的时候，我们才让它自动走。

第5行，`player.body.velocity.x`给Player设了一个X轴的速度`260`。也就是让Player能以`260像素/秒`的速度往右走。

第6行，`player.animations.play('walking')`让Player在走的时候播放动画。这个叫`walking`的动画是我们在[上一篇博客](http://zhongzhu.github.io/2016/04/10/%E7%94%A8Phaser%E5%92%8CPhoneGap%E6%9D%A5%E5%BC%80%E5%8F%91iOS%E5%92%8CAndroid%E6%B8%B8%E6%88%8F-5-%E6%98%BE%E7%A4%BA%E6%B8%B8%E6%88%8F%E5%9C%BA%E6%99%AF/)定义的。定义的代码如下：

```Javascript
this.player.animations.add('walking', ['alienBlue_walk1', 'alienBlue_walk2'], 5, true);
```

## 试试我们的代码 ##
现在打开Chrome浏览器，输入`http://localhost:8000/part4.html`应该可以看到如下界面：

![player walsking](1.png)

很神奇，对吧？Player从左一直往右走，走的时候能看到`walking`动画。Player穿过所有的Enemy，一直走到游戏场景的最右边。穿过所有的Enemy肯定不对的，后面我们会处理。现在我们先让Player跳起来。

## 让Player跳起来 ##
Player跳起来的代码如下。更改主要在Play.create()里，此外还需增加一个自定义函数playerJump()：

```Javascript
create: function() {
  // 设置游戏背景颜色
  // 显示Tile Map
  // 显示Enemy

  // 显示Player
  this.player = this.add.sprite(60, 150, 'texture-atlas', 'alienBlue_walk2');
  this.player.anchor.setTo(0.5, 0.5);
  this.player.animations.add('walking', ['alienBlue_walk1', 'alienBlue_walk2'], 5, true);

  this.physics.arcade.enable(this.player);
  this.player.body.gravity.y = 1000;
  this.camera.follow(this.player);

  this.input.onDown.add(this.playerJump, this);
},

playerJump: function() {
  if (this.player.body.blocked.down) {
    this.player.animations.stop();
    this.player.frameName = 'alienBlue_walk2';
    this.player.body.velocity.y = -550;
  }
},
```

第15行，当玩家点击屏幕或者点击鼠标时，调用函数`this.playerJump`。`input.onDown`是一个Phaser信号，当鼠标点击或者屏幕点击时会触发。

第18-23行，我们定义了函数`this.playerJump`。

第19行，仅当Player站在`GroundLayer`地面上时，Player才能跳。这是防止Player跳到控制后，玩家点击屏幕，造成Player可以在`空中连跳`。

第20行，当Player跳起来的时候，停止Player的`walking`动画。防止Player在空中时脚还在交叉走，看起来很怪异。

第21行，Player的`walking`动画被停止了，但还是需要显示一个Player图片的，这里我们选择用图片`alienBlue_walk2`。Player在空中的整个过程都显示这一张图片。

第22行，这是Player能跳起来的关键。我们给Player的Y轴速度一个`550`的负值。Y轴的正值是Player往屏幕`下方走`，而负值是往屏幕`上方走`，也就是跳了。当然Player不是原地跳，前面我们用`this.player.body.velocity.x = 260`给Player一个X轴正值，这样它就可以`向右走，并向上走`，也就是`向右前方跳`了。

## 试试我们的代码 ##
现在打开Chrome浏览器，输入`http://localhost:8000/part4.html`。用鼠标点击游戏界面，就可以看到Player跳起来了。

![player jumping](2.png)

## Player撞到Enemy后重新开始 ##
前面提到Player穿过所有的Enemy是不对的，的确，一般的游戏Player被坏人击中都或失血或者死掉。我们的游戏简单些，撞到Enemy就让Player重头开始跑吧。

先看看如何判断Player`撞到`Enemy:

```Javascript
update: function() {
  this.physics.arcade.collide(this.player, this.groundLayer);
  this.physics.arcade.overlap(this.player, this.enemyGroup, this.playerHit, null, this);

  if (this.player.body.blocked.down) {
    this.player.body.velocity.x = 260;
    this.player.animations.play('walking');
  }
}
```

第3行，Phaser用`this.physics.arcade.overlap()`函数来判断碰撞。头两个参数是碰撞的双方，第三个参数是碰撞之后的回调函数。下面是回调函数`playerHit()`。

```Javascript
playerHit: function(player, enemy) { 
  this.player.animations.stop();
  this.player.reset(60, 150);
  this.player.body.velocity.x = 0;
  this.player.body.velocity.y = 0;

  this.player.body.blocked.down = false;
}
```

第2行，停掉Player的`walking`动画。

第3行，把Player从现在的坐标移回到初始坐标：`X坐标60，Y坐标150`。

第4-5行，Player的X轴和Y轴速度都变为`0`。

第7行，Player撞到Enemy的时候可能是站在地上的，所以`this.player.body.blocked.down === true`。当Player移回到初始坐标(60, 150)的时候是在空中，所有要把`blocked.down`改为`false`。

就这么多代码，我们完成了今天的三个目标。现在试试我们的代码是否工作。

## 试试我们的代码 ##
现在打开Chrome浏览器，输入`http://localhost:8000/part4.html`。用鼠标点击游戏界面，就可以看到Player跳起来，撞到绿色Enemy后Player又回到了原点重新开始。酷～～～，已经像一个完整的游戏了，不是吗？

![player overlap with enemy](3.png)

## 今天就到这里了 ##
下次我们讲游戏信息（Player的生命数，已跑完多少米）的显示，及其`You Win`和`You Lose`界面。