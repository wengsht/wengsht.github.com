---
layout: post
title: 将PyQt4+python2.7转换为exe程序
abstract: PyQt转换成exe，便于在没有环境的windows下执行程序

tags:
- python
- qt
- exe

type: programming
---

## 简介 ##
<hr/>

PyQt开发桌面软件非常快，虽然在linux和mac下PyQt安装非常简单，但是大部分windows不可能都愿意点开安装图标。所以如果能将python脚本打包成exe的话就非常方便了。

网上搜到了三种软件:

* py2exe
> 看起来名字挺靠谱，不过网上介绍对python2.7之后支持不够

* pyInstaller
> 对python2.7支持

* cx_freeze
> 好吧我唯一试成功的一个

所以就只介绍cx_freeze的用法吧

## 安装 ##
前往[这里](http://cx-freeze.sourceforge.net/)下载对应版本的cx-freeze，windows下直接安装即可。

## 使用 ## 
进入待转换的python脚本目录下，比如`C:\main.pyw`就进入`C:\`，然后新建文件setup.py,键入以下内容
    

    from cx_Freeze import setup, Executable  

    setup(  
            name = "PyQtSort",  
            version = "0.1",  
            description = "PyQtSort",  
            executables = [Executable("main.pyw", base = 'Win32GUI')]) 

将`main.pyw`替换成你的脚本名

保存退出

打开`cmd`, 进入`C:`（python脚本所在目录）,运行`python setup.py build`

自动生成目录下`build/exe.win32-x.x/main.exe`

完成。

## 可能遇到的问题 ##
### ImportError ###

`Import: NoModule named atexit`

虽然不知道atexit此物是干嘛的，不过很明显在main.pyw中加入一句`import atexit`就解决问题了！

### AttributeError ###

`AttributeError: 'Module' object has no attribute 'QtSort'`

其中QtSort是我自己写的一个模块，此处出错在程序中的一句：

`from lib.sort_algo import *`

而包lib.sort_algo.__init__.py中__all__包含了QtSort，照理来说不应该有问题。

解决办法:

把import * 改成 import 具体模块就可以了

## 后记 ## 
如果出现其他问题建议google吧，不知道问什么PyQt相关的问题比较少，虽然很简单的问题也得自己纠结。
