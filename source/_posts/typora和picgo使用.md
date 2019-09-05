---
title: typora和picgo使用
date: 2019-08-09 20:32:59
tags:  [hexo,typora,picgo] 
series: hexo折腾
category: 我的
---

# typora和picgo使用

## 1.picgo下载

https://github.com/Molunerfinn/PicGo/releases/tag/v2.1.2

## 2.picgo码云插件

- 在picgo插件栏中搜索gitee-uploader插件

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904105711.png)

## 3.设置gitee仓库

- 在gitee新建一个公开的资源仓库

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904105937.png)

- 设置个人token

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904110128.png)



## 4.设置picgo的gitee配置参数

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904110538.png)

repo：仓库地址去掉码云域名  https://gitee.com/chenjinhua_939598604/resources    chenjinhua_939598604/resources  

branch：主干

token：上面设置的个人token

path：是仓库中新建static文件夹

**点击设为默认图床，点击确定**

## 5.在picgo上传区测试上传图片

将本地文件拖拽到上传区，弹出返回的图片上传地址，测试上传成功

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904111322.png)

## 6.设置picgo上传图片的快捷键

点击picgo的设置栏，点击修改上传快捷键后面的设置按钮，然后输入组合键，确定

![](https://gitee.com/chenjinhua_939598604/resources/raw/master/static/20190904111017.png)

## 7.打开Typora测试上传图片

用QQ的ctrl+ALT+A快捷键对需要的区域截图，存储到剪贴板中，然后按下刚才在picgo设置好的上传快捷键ctrl+u组合键上传到gitee，会返回图片的上传地址到剪贴板中，最后在typora中ctrl+v粘贴。

