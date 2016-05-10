---
title: 用cytoscape.js展示neo4j网络关系图 - 1. Flask
date: 2016-03-10 16:03:23
tags:
categories:
- neo4j
---
用可视化的方式来展示网络关系图是一件挺有趣的事情，在选定用cytoscape.js来显示neo4j图形数据库的数据后我做了一个原型，并用下面三篇博客来记录了做原型的过程。

- [用cytoscape.js展示neo4j网络关系图 - 1. Flask](2016/03/10/用cytoscape-js展示neo4j网络关系图-1-Flask/)
- [用cytoscape.js展示neo4j网络关系图 - 2. py2neo（这篇博客）](2016/03/11/用cytoscape-js展示neo4j网络关系图-2-py2neo/)
- [用cytoscape.js展示neo4j网络关系图 - 3. cytoscape.js](2016/03/11/用cytoscape-js展示neo4j网络关系图-3-cytoscape-js/)

## 要解决的问题 ##
最近在找一种可视化方案来显示IT网络中的节点（PC，服务器，路由器，人）之间的关系。经过一系列的调研，初步选定如下方案。

## 使用的方案 ##

- **后端**
	- [neo4j](http://neo4j.com/)， 图形数据库用来存储网络节点及节点间的关系
	- Web框架[Flask](http://flask.pocoo.org/)，一个基于Python的Web微框架
	- [py2neo](http://py2neo.org/)，neo4j的Python API包
- **前端**
	- [cytoscape.js](js.cytoscape.org)，显示节点及节点间的关系
	- [jQuery.js](https://jquery.com/), AJAX必须用的库 
- **开发平台**
	- Windows 7, 64-bit
<!-- more -->

整体的架构如下：

![整体的架构](1.png)

## 开发环境搭建 ##
### neo4j ###
首先要去[neo4j](http://neo4j.com/download/)下载community免费版， 我用的是neo4j-community_windows-x64_2_2_1.exe 。

安装后，为了简化开发，我们先把neo4j的用户鉴权关掉。这需要修改`C:\neo4j-community-2.2.1\conf\neo4j-server.properties`，把`dbms.security.auth_enabled=true`改成`dbms.security.auth_enabled=false`。修改后双击`C:\neo4j-community-2.2.1\bin\Neo4j.bat`启动neo4j server。

在浏览器里访问`http://localhost:7474/`，如果看到下图就证明neo4j安装成功了。

![neo4j brower page](2.png)

鉴于我们只是做一个prototyping，就不用真实的IT节点数据，直接用neo4j自带的Movie数据来做原型。照着下面的步骤做完，把Movie数据插入到neo4j库。

  1. 点击“Write Code”
  ![这里写图片描述](3.png)

  2. 点击“Create a graph”
  ![这里写图片描述](4.png)

  3. 按照指示做完第一步，你应该能看到网络图显示出来了
  ![这里写图片描述](5.png)
  ![这里写图片描述](6.png)

### Flask, py2neo ###
首先确认你已经安装了Python 2.7.x。然后还需安装Flask和py2neo。这两个Python包的安装可以用[requirements.txt](https://github.com/zhongzhu/cytoscape_neo4j/blob/master/requirements.txt)文件的方式安装。你可以写一个如下的文件

![这里写图片描述](7.png)

然后用pip命令来安装requirements.txt里面列出的python包。

```
C:\>pip install -r requirements.txt
```

### 前端的cytoscape.js和jQuery.js ###
它们只是些javascript文件，后面会讲到如何下载及把它们放到什么地方。

## 要完成的功能 ##
要把neo4j数据库里面的Movie数据正确的显示到前端，我们需要完成如下的任务。

 1. 搭建基于Flask的简单网站 (这篇博客)
 2. 用py2neo来获取neo4j的节点及关系
 3. 用cytoscape.js来显示网络关系图

## 开始写代码 ##
以上我们完成了开发环境安装及其功能设计，可以开始最开心的coding阶段啦。大家可以在[我的Github](https://github.com/zhongzhu/cytoscape_neo4j.git)上找到下面讲到的所有源代码。

## 搭建基于Flask的简单网站 ##
让我们来搭建一个基于Flask的简单网站。首先创建如下目录：

```
C:\>mkdir cytoscape_neo4j
C:\>mkdir cytoscape_neo4j\templates
C:\>mkdir cytoscape_neo4j\static
C:\>mkdir cytoscape_neo4j\static\js
C:\>mkdir cytoscape_neo4j\static\css
```

完成后的目录结构如下

```
C:\cytoscape_neo4j
+---static
|   +---css
|   \---js
\---templates
```

`cytoscape_neo4j`目录用来放后台Python代码（整个项目就一个python文件，app.py）。`static`目录用来放网站的静态文件，如javascript/css文件。`templates`目录用来存放网页（整个项目只有一个网页，index.html）。

现在来写Flask应用程序`app.py`(源代码： [cytoscape_neo4j/app.py](https://github.com/zhongzhu/cytoscape_neo4j/blob/master/app.py)).

```python
# coding=utf-8
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(debug = True)    
```

6到8行代码的意思是：如果用浏览器访问根目录"/"，app.py就会发送"Hello, World!"到浏览器。让我们来启动我们的Flask应用。在Windows命令行下输入如下命令。如果看到`Running on http://127.0.0.1:5000/`的提示就表示Flask网站运行起来了。

```
C:\>cd cytoscape_neo4j

C:\cytoscape_neo4j>python app.py
 * Running on http://127.0.0.1:5000/
 * Restarting with reloader
```

可以在浏览器里访问`http://localhost:5000`，就能看到"Hello, World!"正常显示出来了。

![这里写图片描述](8.png)

Hello World正常运转后，我们来把它完善一下，做成一个真正满足我们功能要求的网站（也就是添加需要的html/Javascript/css文件了）。网站最后的目录结构是这样的。

```
C:\cytoscape_neo4j
|   app.py
|
+---static
|   +---css
|   |       style.css
|   |
|   \---js
|           code.js
|           cytoscape.min.js
|           jquery-1.11.2.min.js
|
\---templates
        index.html
```

`index.html`是我们唯一的网页。`cytoscape.min.js`是cytoscape必须的Javascript库。AJAX当然少不了`jquery-1.11.2.min.js`。`code.js`是我们的前台程序，它调用cytoscape.min.js在前台生成网络关系图。`style.css`是我们自定义的样式表，cytoscape.min.js会读它来初始化网络关系图的样式（比如：画布的宽度，高度）。`app.py`前面见过了，是Flask应用程序。

你可以在[JQuery的官网下载](http://jquery.com/download/)下载jquery-1.11.2.min.js。

[cytoscape的官网](http://js.cytoscape.org/)可以下载cytoscape.js-2.4.0.zip，解压后我们只需要cytoscape.min.js就好。

现在我们来看看`index.html` (源代码： [cytoscape_neo4j/templates/index.html](https://github.com/zhongzhu/cytoscape_neo4j/blob/master/templates/index.html))

```html
<!DOCTYPE html>
<html>
<head>
    <title>学习Cytoscape.js和neo4j</title>
    <link href="/static/css/style.css" rel="stylesheet" />
    <script src="/static/js/jquery-1.11.2.min.js"></script>
    <script src="/static/js/cytoscape.min.js"></script>
    <script src="/static/js/code.js"></script>
</head>
<body>
  <h1>Movie网络图</h1>
  <div id="cy"></div> 
</body>
</html>
```

代码第5行引入了我们自定义的style.css样式表。第6，7行引入了刚下载的jquery和cytoscape库。第8行引入我们的前台程序code.js.

在11行我们准备在页面上显示“Movie网络图”。比较特殊的是第12行，这个`id="cy"`的div将会被cytoscape用做画布来绘制网络关系图。

`app.py`也需要做一些修改。（源代码： [cytoscape_neo4j/app.py](https://github.com/zhongzhu/cytoscape_neo4j/blob/master/app.py)）

```
# coding=utf-8
from flask import Flask, jsonify, render_template

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

if __name__ == '__main__':
    app.run(debug = True)
```

代码6-8行的意思是，如果访问网站的根目录"/"，Flask会返网页cytoscape_neo4j/templates/index.html。

`style.css`我们将定义画布的大小和背景色。（源代码：[cytoscape_neo4j/static/css/style.css](https://github.com/zhongzhu/cytoscape_neo4j/blob/master/static/css/style.css)）

```css
/* cytoscape graph */
#cy {
    height: 400px;
    width: 500px;
    background-color: #f9f9f9;
}
```

`index.html`里面定了`<div id="cy"></div> `，所以css里用`#cy`来做选择器。`style.css`里我们定义画布的宽度500px，高度400px，颜色是灰色。

`code.js`目前用不上，建一个空文件就行。

现在可以看看我们修改的效果了。按如下步骤启动`app.py`。

```
C:\>cd cytoscape_neo4j

C:\cytoscape_neo4j>python app.py
 * Running on http://127.0.0.1:5000/
 * Restarting with reloader
```

在浏览器里访问`http://localhost:5000`应该可以看到下图，一行标题`Movie网络图`，加下面一个500x400的灰色画布。

![这里写图片描述](9.png)

到此，我们的基本工作都完成了。下一个博客会讲到如何用py2neo来查询neo4j获取需要的节点和节点之间的关系。