---
layout: post
title: "踩到过的坑"
description: ""
category: 
tags: []
---

> 相信我们每个人在自己的学习和工作中，都会遇到许多的bug，即使再牛X的人，也会跟bug打交道。我在工作就遇到很多的问题，这里记录下来。俗话说“吃一堑，长一智。”，不说长 一智，别在同一个地方摔倒两次就行。

#### 1.缓存key被覆盖
---
    按照常理来说，开发环境和线上的正式环境是分开的，线上的程序连接线上的库，测试的程序连接开发机的库。但是保证不了有的“新司机”不按常理出牌。我就是一枚新手。

    最近在开发的过程中，使用到了memcache缓存，由于开发微信公众号的功能，就把测试号的代码放在线上环境进行开发，线上开发也没啥，memcache服务器还是连的本机，结果杯具了，测试公众号的缓存key，把正式公众号的缓存给覆盖了。正式号出来都变成了测试数据。 

    大家以后可得注意了，在相同的环境中使用缓存，至少得变一下key的前缀，这样就不会冲突，最好是测试环境与正式环境相分离。

---

#### 2.线上代码备份
----
    由于公司人员更换，最终别人的代码几经周折还是落在了我的头上。

    我不知道以前同事是如何上线代码的，公司也没有一条正式的发布机制，大部分代码都是通过svn提交到线上。更有甚者，直接操作线上的代码（这简直要命啊！！！）。

    刚交接到我头上，就接到刚需，那没办法，硬着头皮看别人的代码，梳理的差不多了，改动的代码都从svn提交到线上。结果情况不妙，越改越乱，最终发现，线上代码跟svn荡到本地的代码不太一样，究其原因，以前同事是直接改动的线上的代码，么有通过svn提交的代码，我再本地通过svn荡下来的代码不是线上最新的代码，结果我修改后提交到线上，就会覆盖以前线上的代码。最终导致顾此失彼。我真想说一句……

    幸好改动之前，把线上代码通过ssh下载到了本地，功能才得以恢复。再次提醒猴子们，再动线上的代码之前先的在本地保存一份原始代码。在出现问题以后，至少能还原成当前的样子。

----

#### 3.数据库字段设计
----
    一个有经验的老司机肯定都知道，在设置数据库字段的时候，都不允许为NULL，都给字段设置上默认值。再踩过这个坑以后，我深深的感受到了这样做的好处。

    1. 如果有的字段是NULL，并且没有默认值，那么如果出现空数据，该字段是用不到索引的。
    2. 如果字段为NULL，在前台程序没有做好相应判断的时候，总会有意外的错误发生。

    也是一次修改别人的老项目，在原有的代码基础上新加功能，在测试机上都调试好啦，上线就出问题了，测试机的数据库的表结构与线上不一样，线上以前建的表有很多为NULL 的字段，并且没有默认值，在前端程序没有做充分判断的情况下，就报了很多意外的错误。

    解决办法就是，导出线上的表结构到一个文本文件，然后使用正则去替换这些为NULL 的字段，设置为NOT NULL, 然后重新导入到线上库里面。（你也可以一张一张的修改线上表结构，估计累个半死……）

----

#### 4.多台服务器代码同步问题
----
    以前做后台功能的时候，遇到一问题，线上后台编辑文章以后，给一预览功能，预览的效果跟前台是一样的。

    由于线上是多台服务器跑着，每次编辑都是修改的主库，然后保存在其他的从库。当预览的时候，都是读取的从库，主从数据还没同步，预览不了，打开404？如何给编辑修复“预览”这一功能？

    1.缩小主从同步的时间，但是也会存在时间差。
    2.给编辑的电脑，设置一个HOST， 直接指向线上的主服务器。这样每次都能访问到主库。就不会存在主从同步的问题了。

----

#### 5.编码问题
----
    在使用iconv()函数进行编码转换的时候，有时候会出现数据为空的情况。

    1.发现iconv函数在转换字符“-”到gb2312的时候会报错，如果没有ignore参数，所有该字符后面的字符串都无法保存了。
    2.mb_convert_encoding不会出现这个问题，还可以指定多种转换编码，但是，效率要比iconv差太多。
    3.一般情况下，还是用iconv函数，只有当遇到无法确定原编码是何种编码，或者iconv转化后无法正常显示时才用mb_convert_encoding 函数 。
    
----
{% include JB/setup %}