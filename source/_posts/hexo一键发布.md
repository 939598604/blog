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

## hexo发布

### 一.配置git秘钥

**1.在bash shell命令行执行**

```
$ ssh-keygen -t rsa -C "939598604@qq.com"
```


直接Enter就行。然后，会提示你输入密码，不需要输入密码：
Enter passphrase (empty for no passphrase): [Type a passphrase] 
 Enter same passphrase again: [Type passphrase again]
完了之后，大概是这样。
Your identification has been saved in /home/you/.ssh/id_rsa. 
 Your public key has been saved in /home/you/.ssh/id_rsa.pub. 
 The key fingerprint is:  01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db your_email@youremail.com
这样。你本地生成密钥对的工作就做好了。

**2.添加公钥到你的github帐户**

**(1)查看你生成的公钥：大概如下：**

```
$ cat ~/.ssh/id_rsa.pub  

ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAklOUpkDHrfHY17SbrmTIpNLTGK9Tjom/BWDSU GPl+nafzlHDTYW7hdI4yZ5ew18JH4JW9jbhUFrviQzM7xlE

LEVf4h9lFX5QVkbPppSwg0cda3 Pbv7kOdJ/MTyBlWXFCR+HAo3FXRitBqxiX1nKhXpHAZsMciLq8V6RjsNAQwdsdMFvSlVK/7XA t3FaoJoAsncM1Q9x5+3V

0Ww68/eIFmb1zuUFljQJKprrX88XypNDvjYNby6vw/Pb0rwert/En mZ+AW4OZPnTPI89ZPmVMLuayrD2cE86Z/il8b+gw3r3+1nKatmIkjn2so1d01QraTlMqVSsbx NrRFi9wrf+M7Q== schacon@agadorlaptop.local
```

**(2)登陆你的github帐户。然后 Account Settings -> 左栏点击 SSH Keys -> 点击 Add SSH key**
**(3)然后你复制上面的公钥内容，粘贴进“Key”文本域内,，点击 Add key**

这样，就OK了。然后，验证下这个key是不是正常工作。

```
$ ssh -T git@github.com
 Attempts to ssh to github
```

如果，看到：
`Hi username! You've successfully authenticated, but GitHub does not  provide shell access.`
就表示你的设置已经成功了。

###二. hexo配置

1.安装发布插件

```
npm install --save hexo-deployer-git
```

2.在_config.yml文件加入_

```
deploy:
  type: git
  repository: git@github.com:939598604/939598604.github.io.git
  branch: master
```

3.在bash shell执行命令

```bash
$ hexo d
```

报错信息

```
ERROR Plugin load failed: hexo-deployer-git
SyntaxError: Unexpected token ...
```

修改hexo-deployer-git版本为1.0.0

```
{
  "name": "hexo-site",
  "version": "0.0.0",
  "private": true,
  "hexo": {
    "version": "3.9.0"
  },
  "dependencies": {
    "hexo": "^3.9.0",
	"hexo-deployer-git": "^1.0.0",
    "hexo-generator-archive": "^0.1.5",
    "hexo-generator-category": "^0.1.3",
    "hexo-generator-index": "^0.2.1",
    "hexo-generator-tag": "^0.2.0",
    "hexo-renderer-ejs": "^0.3.1",
    "hexo-renderer-marked": "^1.0.1",
    "hexo-renderer-stylus": "^0.3.3",
    "hexo-server": "^0.3.3"
  }
}
```

4.再执行

```
$ hexo d
```

### 三.发布多余的md文件

将其他md文件放到source/\_posts文件目录下即可

### 四.一键发布

新建一个deploy.bat结尾的文档，在创建的.bat文档中输入以下内容 

```
call  hexo clean
call  hexo g
call  hexo d
pause
```







