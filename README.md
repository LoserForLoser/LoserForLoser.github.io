# LoserForLoser.github.io

本文讲述如何用 jekyll + GitHub 搭建一个像我这样的个人博客

整个过程操作简单上手快非常适合智商不够如我的朋友们来学习一下

以下是整个流程顺序（Mac 环境）

+ 下载ruby
+ 安装jekyll
+ 安装bundler
+ 建立你的第一个静态博客
+ 开启jekyll服务器
+ 写一篇自己博文
+ 用github pages 展示你的博客
  + 创建个人仓库
  + 克隆仓库到一个指定的文件目录
  + 把你本地的第一个博客文件里的所以文件复制到这个克隆下来的文件
  + 把这些文件push到远程仓库
  + 查看你的博客网站


#### 1.下载ruby

什么是ruby：Ruby，一种简单快捷的面向对象（面向对象程序设计）脚本语言，安装Jekyll需要电脑上安装Ruby，以下是安装步骤：

+ Mac 系统下，本人使用的是 [Ruby China](https://ruby-china.org/) 的镜像并随时更新的最新版，之前看了很多 Jekyll 的教程也建议下载2.3以上的新版。（具体流程请参照[官网教程](https://gems.ruby-china.com/)， 或自行百度其他路子解决）
+ 安装后，输入ruby -v和gem -v来看看是否安装成功，看到版本号就说明成功。


#### 2.下载jekyll

jekyll：jekyll是一个简单的免费的Blog生成工具，类似WordPress。但是和WordPress又有很大的不同，原因是jekyll只是一个生成静态网页的工具，不需要数据库支持。但是可以配合第三方服务,例如Disqus。最关键的是jekyll可以免费部署在Github上，而且可以绑定自己的域名。（注：我自己的没有绑定域名）
我们使用gem来安装jekyll，在命令行中输入

 `gem install jekyll`

所有的jekyll的gem依赖包都会被自动安装。

#### 3.下载bundle

在命令行输入

 `gem install bundler`

bundler：就是一个打包机，他会连接rubygems.org（或者其他你声明的源），然后列出所有你指定的符合你需要的 gem。因为所有你在Gemfile里的依赖有它们自己的依赖，所以基于上面的Gemfile运行bundle install会安装相当多的的 gem。（我也不太了解，自己可以百度）

#### 4.建立自己的第一个博客

首先看看你想把你的博客建在哪里，我的是直接扔在桌面了：

 `cd Desktop/:`

然后输入要创建的博客

`jekyll new blog  //blog为你的博客文件名`

控制台可以看见（创建的地址有所不同）New jekyll site installed in C : /blog。你的C盘的文件夹下也会出现相应的blog文件。

#### 5.开启jekyll内置服务器

实现转入blog的目录，输入：

`cd blog//一定要进入创建的对应blog目录，否则服务无法开启`

然后输入：

`jekyll serve  //开启服务器，开启后命令行里自动提示‘按ctrl+c停止’`

Jekyll服务器默认端口是4000，所以打开浏览器输入：http://localhost:4000 就可以看到生成的博客页面。

（题外话，作为一个移动端开发的智障，当我运行服务器后还天真的以为要等命令运行结束才能运行，然后白痴般白白等了一下午。（特做此题记警己示众））

#### 6.使用jekyll写博文

你可能喜欢markdown或html来写博文，都可以，但是博文文件的命名规则要服从下面的规则：

 year-month-title.markup //markup为你的文件格式的后缀名

在你的文章头部添加yaml头信息

```
---
layout: post
title:  "Jekyll+Github搭建个人博客"
date:   2018-07-24 15:03:25
categories: original
---
```

写上自己的博文内容，将这个文件保存在blog里面的_posts目录里面即可。在重启jekyll内置服务器，刷新页面：http://localhost:4000，如果没有，可以先输入：

`jekyll build `

重新生成页面，在启动服务器，这样就可以在页面看到自己添加的博文的标题了。
这就是在本地搭建jekyll和写博文的大致过程了，相信还有其他的搭建方法，但是估计都是大同小异吧。

#### 7.用github 展示你的博客

接下来的操作都是 git 的了。你们既然能看这里就证明你们至少有 git 账号了，我既不废话了。

   1.创建个人仓库
    就是建立一个新的仓库，但是这个仓库的名字必须为你的github的名字+github+io，例： 我的 LoserForLoser.github.io

   2.切换到你想要放 GitHub 博客的文件目录，在这个目录 git clone 结束后把之前的 jekyll new 的文件下的所有文件复制到里面。

   （题外话，之前看了另外一篇帖子的各种骚操作后反倒无法展示，我只能说因人而异了，因此就不在这贴人家了）

   3.然后把里面的所有文件push到刚刚建的远程仓库，步骤我就不写了。
    约几分钟后（因人而异），在浏览器里面输入网址：http://yourname.github.io 就可以看你的个人博客网站了，这就是你的博客网站的地址了。
    前面所说的yourname指的是你的 GitHub 账号名字。

   4.嗯，接下来你就可以查看你的博客网站了。其中还可以在 GitHub 的 settings 中选择你的博客主题。我也还在选主题中。

这就是用 jekyll + GitHub 搭建个人博客的大概过程了，在搭建过程中遇到的种种的问题，就百度吧。
（本文复写自 [博客园 美美王子 的 《jekyll+github搭建个人博客》](https://www.cnblogs.com/yehui-mmd/p/6286271.html) 由于他的是 Windows 般的，相对 Mac 的 Unix 要麻烦些许，请 Windows 的小伙伴自行移步）
