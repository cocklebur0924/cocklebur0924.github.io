---
title: Anaconda + VScode | Python环境安装与管理
date: 2024-01-21 10:11:00
tags: [Study,Soft]
categories: [Study]
mathjax: False
---


本文介绍利用Anaconda进行python环境安装与管理，利用VScode进行代码编辑。


## Anaconda安装与使用

### Anaconda 安装

Anaconda是一款强大的包管理软件，可以配置独立的项目环境，因此在不同项目用到不同python版本的时候就很方便。无需单独下载python！

在[Anaconda官网下载地址](https://www.anaconda.com/download)下载相应的版本。


![Anaconda下载](/images/Soft/anaconda_install_01.png)

<center><img src=/images/Soft/anaconda_install_02.png width=100% /></center>
<center>Anaconda官网下载</center>

如果网速太慢，可以试试[清华镜像下载地址](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)。


下载完成后，双击exe文件，安装。

![Anaconda下载](/images/Soft/anaconda_install_03.png)

一路next，注意如果有勾选环境变量配置的要选上，不过新版本好像已经默认加入环境变量了。
直到下面这一步，安装完成。

![Anaconda下载](/images/Soft/anaconda_install_04.png)

勾上Lauch之后点击Finish，就会自动打开Anaconda Navigator了。浏览器也会弹出来建议你注册获取更多资源的信息，可以不用管。

![Anaconda下载](/images/Soft/anaconda_install_05.png)

到这里，说明Anaconda已经安装成功了。

如果是windows用户，在开始栏就可以看到新安装的这些东东了。
![Anaconda下载](/images/Soft/anaconda_install_06.png)

### Jupyter Notebook 简单使用

下面简单介绍利用默认安装的Jupyter Notebook工具进行python编程。
在打开的Anaconda Navigator里可以看到，这里 JupyterLab 和 Jupyter Notebook 已经是安装好的。这里以 Jupyter Notebook为例，点击Lauch运行，浏览器就会跳出来jupyter页面了。

![Anaconda下载](/images/Soft/anaconda_install_07.png)

在 Jupyter Notebook 页面，右上角点击new python3，就可以打开一个jupyter格式的脚本，开始简单地编程了。

![Anaconda下载](/images/Soft/anaconda_install_08.png)

每个单元格内运行代码的快捷键是 ctrl+enter 或 shift+enter 。

![Anaconda下载](/images/Soft/anaconda_install_09.png)

当然，也可以通过在Anaconda powershell prompt 输入Jupyter notebook直接召唤他。

![Anaconda下载](/images/Soft/anaconda_install_10.png)


### Anaconda 环境创建


通过交互式界面（Anaconda Navigator）和命令行（Anaconda powershell prompt）都可以创建环境，并进行包管理。

(1) 利用 Anaconda Navigator 创建（不推荐）

在Environments界面点击左下角Create即可创建，输入环境名称，选择相应的python版本，点击Create就好了。

![Anaconda下载](/images/Soft/anaconda_install_11.png)

相应的包管理也可以通过点点点的方式实现，然而还是利用命令行方便一些，所以下面主要介绍利用Anaconda powershell prompt 创建并管理环境的一些常用常用命令。

(2) ** 利用 Anaconda powershell prompt 创建 **

打开Anaconda powershell prompt，输入以下命令行即可创建。

```
conda create --name py311 python=3.11  
```

或

```
conda create -n py311 python=3.11 #创建名为py311的环境，其python版本为3.11
```

如果想删除这个环境：

```
conda remove -n py311 --all
```

查看所有环境：

```
conda info -e  #或conda info --envs 或conda env list
```

激活环境：

```
conda activate py311  #conda activate+环境名
```

查看当前环境下所有的包：

```
conda list
```

下载包（以matplotlib为例）

```
conda install matplotlib
```

卸载包

```
conda uninstall matplotlib
```

退出环境，默认回到base环境

```
conda deactivate py311
```

查看环境位置等信息
```
conda info
```

以上，就可以使用python进行简单地编程了，然而为了使编程过程更加丝滑，还是推荐使用VScode~

## VScode 安装与使用

### VScode 安装


在[vscode官方下载地址](https://code.visualstudio.com/Download)下载需要的版本。

![vscode下载](/images/Soft/vs_01.png)

一路next下载，这一步可以勾选添加到Windows上下文菜单，这样可以右键文件通过vscode打开。

![vscode下载](/images/Soft/vs_02.png)

![vscode下载](/images/Soft/vs_021.png)

直到这一步，完成安装~

![vscode下载](/images/Soft/vs_03.png)


### VScode 使用

VScode界面如下，New file 即可新建py格式或ipynb格式的文件，平时同一个项目我通常放在一个文件夹中，Open Folder即可打开项目文件夹。

![vscode下载](/images/Soft/vs_04.png)


这里演示新建了一个ipynb格式文件，右上角可以选择相应的配置环境（**上面介绍的Anaconda已经配置好的环境**），左下角可以打开terminal。每个单元格内运行的快捷键也是**Ctrl+Enter**。

![vscode下载](/images/Soft/vs_05.png)

如果是py格式，配置环境的解释器选择在右下角。

![vscode下载](/images/Soft/vs_06.png)


### 可能遇到的问题

如果vscode中选不到配好的anaconda环境，可能是anaconda没有添加进环境变量，需要手动添加环境变量：
右键此电脑——属性——高级系统设置——环境变量——系统变量Path——编辑——新建，添加你的anaconda下载路径。


♥ The end ♥