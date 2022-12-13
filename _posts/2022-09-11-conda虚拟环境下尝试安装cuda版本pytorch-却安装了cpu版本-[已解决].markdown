---
title: conda虚拟环境下尝试安装CUDA版本PyTorch 却安装了CPU版本 [已解决]
description: （其实就是卸了个PyTorch默认安装的类似儿童锁的包
date: 2022-09-11T15:23:28.792Z
preview: ""
tags:
  - conda
  - cuda
  - pytorch
  - wsl
  - linux
categories:
  - issue
  - envconf
layout: single
lastmod: 2022-09-12T07:15:33.200Z
---

**太长不看版：康康你conda虚拟环境里有没有个叫`cpuonly`的包，有的话卸了它**

本机环境：

+ OS：Ubuntu 22.04.1 LTS (Windows Subsystem for Linux 2)
+ GPU：NVIDIA GeForce RTX 2060 Max-Q
+ GPU Driver：GeForce Game Ready Driver 516.94
+ CUDA Version: 11.7
+ conda: 4.14.0 (miniconda3)

事情是这样的。

搞个ML的科研项目，需要用到PyTorch。

因为实在不想在Win本机上整这种活（感觉conda这玩意容易和pip出优先级问题，日后恶心自己）。

所以就配置了个WSL2可用的CUDA (可惜据说性能损耗相较正常Linux系统低了12%)：

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/11.7.1/local_installers/cuda-repo-wsl-ubuntu-11-7-local_11.7.1-1_amd64.deb
sudo dpkg -i cuda-repo-wsl-ubuntu-11-7-local_11.7.1-1_amd64.deb
sudo cp /var/cuda-repo-wsl-ubuntu-11-7-local/cuda-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install cuda
```

害得配置下环境变量：

```bash
vi ~/.bashrc
```

在.bashrc最后加上：

```bash
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
```

测试：

```bash
$ source ~/.bashrc
$ nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2022 NVIDIA Corporation
Built on Wed_Jun__8_16:49:14_PDT_2022
Cuda compilation tools, release 11.7, V11.7.99
Build cuda_11.7.r11.7/compiler.31442593_0
```

奈斯。

直接全局pip装了torch试了一下：

```bash
$ pip install torch
$ python -c "import torch; print(torch.__version__); print(torch.cuda.is_available())"
1.12.1+cu102
True
```

完美。

但我这也不能直接用啊，虽说是个子系统吧。。最后还是安了个miniconda，然后conda开了个虚拟环境尝试安装PyTorch套件。

```bash
conda config --add channels conda-forge #这个被PyTorch依赖的channel必须要先加上
conda config --set channel_priority strict #而且要设置成strict模式，否则会优先从默认channel里找，找不到才会去conda-forge里找
conda create -n PyTorch python=3.9 #创建虚拟环境
conda activate PyTorch #激活虚拟环境
conda install pytorch torchvision torchaudio cudatoolkit=11.7 -c pytorch -c conda-forge #安装PyTorch
```

一路顺风顺水，直到我测试了一下：

```bash
$ python -c "import torch; print(torch.__version__); print(torch.cuda.is_available())"
1.10.2
False
```

直接给我整不会了。
在这之后就是相当nt的网络检索和无意义debug，又重启了几次，False照常。

直到万念俱灰的我`conda list`了一下，发现了一个可疑的玩意，叫`cpuonly`，版本是`2.0`，而且是`pytorch`这个channel里的。

cpu only？属实好活，我这块2060压根用不上呗。

又看了一眼`pytorch`版本，瞳孔地震，居然是1.10.2的cpu版本。

上网，关键词检索：`conda 虚拟环境 torch cpuonly` 果然有人遇到过这个问题，说是这个叫`cpuonly`的包会强制把`pytorch`的`cuda`支持关掉，而且是在`pytorch`安装的时候就会被默认安装上去。

无一例外地，解决方案都是直接`conda uninstall cpuonly`把这玩意卸掉。

我试了一下，居然还真成了，它甚至还自动把我的`pytorch`包和相关包都更到CUDA版了。

又测试了一遍：

```bash
$ python -c "import torch; print(torch.__version__); print(torch.cuda.is_available())"
1.12.1+cu116
True
```

问题解决，泪目。

2022-09-12更：有佬称这方法是瞎猫碰死耗子，咱也不太确定，但能用就不动它了。
链接（是天杀的CSDN）→[佬发现的](https://windses.blog.csdn.net/article/details/125910538)
