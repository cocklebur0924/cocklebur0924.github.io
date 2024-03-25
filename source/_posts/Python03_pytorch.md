---
title: Pytorch深度学习环境配置 | windows
date: 2024-03-24 12:17:00
tags: [Study,Soft]
categories: [Study]
mathjax: False
---

首先，配置好Anaconda环境，可参考[Anaconda + VScode | Python环境安装与管理](https://cocklebur0924.github.io/2024/01/21/Python01_install/)。在此基础上，本文介绍Pytorch深度学习环境配置。

### Pytorch安装

打开Anaconda Prompt, 查看电脑支持的CUDA版本：

```
nvidia-smi
```

![nvidia-smi](/images/Soft/python03_nvidia-smi.png)

可以看到，我的显卡型号为NVIDIA GTX1660 SUPER，支持的CUDA版本最高到12.2

记得切到你想下载pytorch的环境：

```
conda activate seismic
```

到[Pytorch官网](https://pytorch.org/)选择低于你电脑支持的cuda版本，复制下载命令，例如我的电脑支持到12.2，这里我选择了11.8：

```
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```

![Pytorch](/images/Soft/python03_pytorch.png)

然后漫长的等待,等到出现successful installed，就安装好啦。

![](/images/Soft/python03_ok.png)

### 检查是否安装成功

打开一个jupter，选择刚刚安装好pytorch的那个环境，import torch 检查是否安装成功。

![](/images/Soft/python03_check.png)

### The End

最后，如果你的电脑木有显卡，可以试试在线租卡，比如[AutoDL](https://www.autodl.com/login?url=/home)。

