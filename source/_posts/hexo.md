---
title: hexo
date: 2019-03-09 11:29:24
author: 陈锦华
password: 123
toc: true
categories: hexo
tags:
  - hexo
---

## hexo

### 一、 写博客

1.定位到我们的项目的根目录，执行命令：

```
hexo new 'hexo'
```

2.hexo会帮我们在`_posts`下生成相关md文件  hexo.md

```
title: hello_hexo
date: 2019-03-09 11:29:24
tags:
```

3.创建一个页面

```bash
$ hexo new "My New Post"
```

4.运行服务器

```bash
$ hexo server
```

5.生成文件

```bash
$ hexo generate
```

6.部署服务

```bash
$ hexo deploy
```

一般完整格式如下：

```
title: postName #文章页面上的显示名称，一般是中文
date: 2013-12-02 15:30:16 #文章生成时间，一般不改，当然也可以任意修改
categories: 默认分类 #分类
tags: [tag1,tag2,tag3] #文章标签，可空，多标签请用格式，注意:后面有个空格
description: 附加一段文章摘要，字数最好在140字以内，会出现在meta的description里面
```

那么`hexo new page 'hexo'`命令和`hexo new 'hexo'`有什么区别呢？ 

`hexo new page "hexo" ` 生成路径如下

```
项目\source\_posts\hexo.md
```

最终部署时生成：`\项目\public\hexo\index.html`，但是它不会作为文章出现在博文目录。 

3.如果文件夹下文章很多的时候，找起来就会很不方便。 

Hexo作者也给出来解决办法： 

在Hexo目录下的scripts目录中创建一个JavaScript脚本文件。如果没有这个scripts目录，则新建一个。 scripts目录新建的 js 脚本文件可以任意取名。 

```
var spawn = require('child_process').exec;
// Hexo 2.x 用户复制这段
//hexo.on('new', function(path){
//  spawn('start  "Typora编辑器绝对路径.exe" ' + path);
//});
//C:\Program Files (x86)\Typora\Typora.exe 是MakdownPad编辑器在我本地的路径！
// Hexo 3 用户复制这段
hexo.on('new', function(data){
  spawn('start  "C:\Program Files (x86)\Typora\Typora.exe" ' + data.path);
});
```

然后再创建文件输入命令之后就会自动打开Typora编辑器来编辑了 

### 二、更换 Hexo 主题及样式

1、默认主题
我们通过 hexo init 初始化的网站，使用的是 hexo 默认的主题样式 lanscape，位于 xxx/themes/lanscape 目录，我们可以到官网的 主题市场  根据自己的喜好更换其它的主题。
2、更换主题
我目前使用的是 Next 主题，非常简约风。在此以 Next 主题为例，说明如何把 Hexo 的默认主题 lanscape 更换成 next 主题。
3、下载新主题

进入你本地创建的网站的 themes 目录中，在这里打开 Git Bash 命令行，使用 git clone 下载 next 主题：

```
git clone https://github.com/iissnan/hexo-theme-next next
```

这是将 Next 主题下载到本地 themes 目录下的 next 文件夹中。

4 、修改站点配置文件打开站点的配置文件 项目/_config.yml ，找到 theme 配置项，把值从 landscape 改为 next。

```
theme: next
scheme: Pisces
```

5 、修改 next 主题的样式next 主题提供了几个不同的样式可供选择。打开主题的配置文件 blog/themes/next/_config.yml ，找到 Schemes 模块，我用的是 5.1.4 版本的主题，包含有下面 4 种样式：

Muse：这是默认的样式，是 NexT 最初的版本，黑白主调，大量留白；
Mist： Muse 的紧凑版本，整洁有序的单栏外观；
Pisces： 双栏 Scheme，小家碧玉似的清新；
Gemini： 左侧是网站信息及目录，块 + 片段结构布局；

去掉 scheme 前面的注释符 # 即可启用该样式，你可以依次试用一下，选择某一个样式之后执行 hexo g 、hexo d 重新发布即可看到新样式了，我使用的是第三种 Pisces

6、默认语言是英语。

```
language: en
# language: zh-Hans
# language: zh-hk
# language: zh-tw
# language: ru
# language: fr-FR
# language: de
# language: ja
# language: id
# language: pt
# language: pt-BR
# language: ko
# language: it
# language: nl-NL
```

7、在站点配置文件`_config.yml`中可以将语言切换成中文

```
language: zh-Hans
```







