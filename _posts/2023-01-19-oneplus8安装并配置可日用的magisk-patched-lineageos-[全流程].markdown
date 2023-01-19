---
title: OnePlus8安装并配置可日用的Magisk-Patched LineageOS [全流程]
description: 遁入邪教
date: 2023-01-19T08:23:34.259Z
preview: ""
tags:
  - zh-cn
  - android
  - android-root
  - magisk
  - xposed
  - geekery
categories:
  - envconf
  - exp
slug: oneplus8安装并配置可日用的magisk-patched-lineageos-[全流程]
layout: single
keywords:
  - lineageos
  - magisk
  - LOS
lastmod: 2023-01-19T10:55:18.608Z
---

*前言：安装的这套LOS20“可日用”仅为个人观点，刷机有风险，玩机需谨慎。be ware, be ware。不过本人已经将这台LOS机器作为主力机使用了。*
*经本人测试，小而美、Alipay、工行app均可正常使用，支付功能均可用。*

## 碎碎念（跳过，没用）

+ [前往正文](#正文)
+ [我已成功安装LineageOS](#boot修补法安装Magisk)
+ [我已成功安装Magisk](#去你妈的Root检测)

事情是这样的。

本人的手机，一台国版一加8，经常被我十分暴力地折腾。我曾经在一时兴起的情况下从XDA找了个根本不知道可不可靠的基于Android11的氧OS全量包，手动覆盖安装了当年那台机器上的Android10氢OS。就连安装之后我都不知道安的是全球版还是欧版还是印度版，但它能跑还不杀我后台，我就没动它。至于那点Google的trash我干脆用黑域和冰箱解决了，就留了点有用的。也没root，就玩了玩Shizuku。

过了一年半左右又直接本机OTA到了基于Android12的新氧OS，这时候就开始出小问题了。这机器时不时地会触发软重启，通常都是在一键清通知或者后台以后。我当时以为是我哪次瞎鼓捣的ADB指令影响了什么奇奇怪怪的系统级钩子函数，就在termux里用Shizuku那个rish里开logcat然后手动尝试触发这种软重启，结果成功触发是触发了，但log看了半天也看不懂，一个os里的包报的错，什么surface什么玩意的，好像和控制触屏io的东西有关，看log里那玩意一崩系统直接休克了，最后我也没修好，不知道咋下手。

就在一周零几天前氧OS的OTA又给我推了个包，居然是基于Android13的新氧OS。我属实是没想到一加还能给我这两年半以前的机器做新OS的适配。牛逼。一加这人能处。

由于太激动了我当时直接就OTA了，也没备份，直接就安装并重启了。结果遇到了非常邪门的问题，一开机到那个什么同意新条款的页面就给我跳权限管理器频繁停止工作，然后关了这小窗点确定同意条款也没反应，就卡死在那了，过几秒种直接自己重启。完全进不去启动器，不是变砖胜似变砖。

给我气的直接进recovery了，我以为不会有啥事反正也我也没清我system区的东西。谁知道刚进recovery直接告诉我system corrupt了，我一狠心直接启动发现同意新条款界面没了，取而代之的是Google的Welcome界面，电脑开着LAN代理好不容易进桌面了结果发现我原来的软件、数据全没了。啥也没剩。给我干沉默了。这ssd也没法恢复数据。死局。

于是万念俱灰之下老夫决定遁入邪教。

## 正文

### 准备工作

+ 一台足以让机主无路可走的一加8，需打开USB调试。本人测试机型：国行12+256
+ 一台win机器，安装有一加官方adb驱动和全套安卓platform-tools，至少包括adb和fastboot工具
+ 一根可靠的数据线，最好是原装的。
+ 可靠的互联网连接，最好有代理软件，不然在XDA或LOS官网下镜像会奇慢无比。

### 开始

#### LOS & LOS Recovery

win机下载LineageOS官网提供的系统和Recovery镜像文件。对于一加8，项目代号为instantnoodle，链接在这->[一加8LOS20镜像](https://download.lineageos.org/instantnoodle) 本人没有安装TWRP，用的LOS官方Recovery。

先后台下载着这两个偏大的文件，然后按照LOS官方提供的流程进行操作->[LOS官方安装流程](https://wiki.lineageos.org/devices/instantnoodle/install) 建议先全部读一遍再进行操作。流程非常简单，不再赘述。

#### boot修补法安装Magisk

由于本人没有使用TWRP，我用了boot修补法安装了Magisk

##### 从LOS卡刷镜像中提取boot

需要用到一位大佬写的提取工具->[payload-dumper-go](https://github.com/ssut/payload-dumper-go)

在release页面有编译好的版本，下载解压即用。

工具解压后将卡刷包文件（即上文从LOS官网下载的zip文件）解压，将其中的`payload.bin`解压到payload-dumper工具的目录中。然后在工具目录中打开PowerShell，输入：

```bash
./payload-dumper-go.exe payload.bin
```

读条全部完成后从`extracted_`开头的文件夹中找到`boot.img`即为LOS的boot，保留备用。

##### Magisk修补boot

启动手机，访问[Magisk官网](https://themagisk.com)下载并安装Magisk Manager。

打开USB调试连接电脑，用adb测试连接：

```bash
adb devices
```
如果出现手机未识别的提示，则在手机上同意USB调试的权限授权。连接模式可以先选成仅充电。

若连接成功则将上一步提取出的boot文件通过adb传入手机：

```bash
adb push ./boot.img /sdcard/
```
传入成功后boot文件应当出现在手机内部存储根目录。确认已传入手机后，手机打开Magisk Manager，点击Magisk栏的安装按钮，选择“选择并修复一个文件”并点击开始，然后在下个页面选择刚才传入的`boot.img`文件，Magisk Manager将会修补此boot文件并输出一个patched-boot文件到手机Download文件夹。将此修补后的boot文件用adb拉取回电脑：
```bash
adb pull /sdcard/Download/magisk_patched-xxxxxxxx.img ./ #别直接复制，先把xxx改了
```
然后将手机重启至fastboot，再将拉取到的补丁boot文件使用fastboot工具刷入手机：
```bash
fastboot flash boot ./magisk_patched-xxxxxxx.img 
```
然后重启手机，打开Magisk Manager你会发现Magisk已安装成功。

建议提前把Zygisk功能打开，接下来的配置将会重度依赖此功能。

#### 去你妈的Root检测

Bypass Root Detection Of Toxic Apps

##### 准备工作

手机访问以下链接并在releases页面下载最新版模块。注意一定要下载Zygisk版，Riru将会在未来停止支持。

+ [LSPosed](https://github.com/LSPosed/LSPosed/releases)
+ [Shamiko](https://github.com/LSPosed/LSPosed.github.io/releases)

下载后在Magisk Manager底栏最右的模块栏中选择安装本地模块，选择下载的模块zip安装并重启。注意这两个模块每个安装完都要重启，一共要重启两次。

##### Shamiko

安装并开启这两个模块后，在Magisk设置中打开遵守排除列表选项，然后进入配置排除列表并勾选你手机上那些toxic app，比如Alipay、工行app、数字RMB、交管213等等。

勾选后建议立即重启手机使Shamiko生效。理论上Shamiko生效后应用即可正常使用。如果依然不能正常使用请看下一章节。

##### XPosed Module “隐藏应用列表”

LSPosed是一个全新的XPosed框架，安装后你的桌面应该会出现一个LSPosed管理器的图标。

点击进入管理器后在模块仓库搜索“隐藏应用列表”，在附件页下载模块apk，安装后在LSPosed管理器中选择启用，作用域勾选系统框架然后重启。

重启后进入“隐藏应用列表”软件设置，先在应用管理中勾选所有手机上的toxic app，然后在模板管理中选择新建黑名单模板，然后将所有XPosed模块app和Magisk以及其他root相关app都加入此模板，并设置对所有toxic app不可见。然后重启手机即可生效。

#### Optional Work

理论上执行完上面两步，大部分的toxic app就都可以正常运行了。下文还提供了一些额外的可选项以供参考。

##### Work Profile

我本人还用到了Android内置的work profile功能来隔离toxic app到一个单独的沙箱子系统中，这个单独的work profile可以视为一个容器，安装到这个profile的app无法自主地与主用户的app进行任何形式的数据交换，除了系统级priveledge apps如系统自带文件管理器。这个功能有些类似Linux下的lxd，属实好用。

在F-Droid和IzzyOnDroid上能找到的可以提供安卓上work profile管理功能的app主要有两个，一个是大名鼎鼎的炼妖壶Island的FLOSS分支[Insular](https://f-droid.org/zh_Hans/packages/com.oasisfeng.island.fdroid/)，一个是[Shelter](https://f-droid.org/zh_Hans/packages/net.typeblog.shelter/)。安装后跟随应用内guidance即可配置使用。我个人把小而美、Alipay、QQ啥的全放里面了，主用户只安装了各种FOSS，非常安心。

...








