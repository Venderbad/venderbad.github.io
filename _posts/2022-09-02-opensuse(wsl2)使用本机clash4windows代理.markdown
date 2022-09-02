---
title: OpenSUSE(WSL2)使用本机Clash4Windows代理
description: 小坑一个
date: 2022-09-02T14:56:44.117Z
preview: ""
tags:
  - linux
  - pitfall
  - proxy
  - wsl
  - resolved
categories:
  - misc
layout: post
lastmod: 2022-09-02T14:56:51.642Z
keywords: []
slug: opensuse-wsl2-使用本机clash4windows代理
---
闲来无事，鼓捣了一波win11的WSL2。
发现微软商店里有滚动更新的大蜥蜴可以装，抱着玩玩的心态整了一个。
安装后发现和Kali一样是minimalist setup，遂手动安装了一些必要的包。
发现zypper速度奇慢无比，就换成了清华的zypper源，结果也没快到哪去，装个neovim花了小10分钟。
遂决定用本机clash给大蜥蜴提个速。UWP loopback全开，咱也不知道有没有生效，就试着装了个GitHub上一套NeoVim开箱即用的配置→[LunarVim](https://github.com/LunarVim/LunarVim)。
readme里贴心地给了个oneliner安装脚本↓
```bash
bash <(curl -s https://raw.githubusercontent.com/lunarvim/lunarvim/master/utils/installer/install.sh) -y
```
尝试运行，没反应。
Cc断掉重新跑，没反应。等了十分钟，果不其然个给我跳了个time out，说是接不到`raw.githubusercontent.com`。
嗯，classic。
再来一遍，跑起来了，项目图标出来了，然后卡着不动。此时老夫还没意识到是代理的问题。
又试了几次，跑了小半个点，处理了一众dependency issue，最后批处理安装插件又跳了个fatal error，还是接不到`raw.githubusercontent.com`。
整个安装过程有如阿基里斯追王八，安装成功似乎就在眼前，却可望但不可及。
这时候我终于意识到不对了，就把Google百度和localhost全ping了一遍，啥反应也没有。又试了试这个
```bash
curl myip.ipip.net
```
居然是喜闻乐见的北京鹏博士，可tnnd给我高兴坏了。
百度了一波，发现WSL2使用Hyper-V跑的，接入网络是通过`vEthernet (WSL)`这个虚拟网卡实现的，和本机的网络是隔离的，所以本机的代理对WSL2是无效的。
也就意味着我的UWP loopback压根就没用。好嘛。
于是老夫换了个思路，打开了本机clash的局域网代理，查了一眼WSL2虚拟网卡的IP，`172.xx.xx.1`，然后在WSL2里面设了个全局代理的环境变量。
```bash
export all_proxy="socks5://172.xx.xx.1:7890"
```
重启clash，又curl了一遍`myip.ipip.net`，中国香港，这次对味了。
又跑了一遍安装脚本，半分钟跑完了，心态爆炸。小半天直接白瞎了。

2022-09-02补充：已有更优雅的解决方案→[为 WSL2 一键设置代理](https://zhuanlan.zhihu.com/p/153124468)
这位哥写好了一键配置脚本，可以直接抄作业。