---
title: 地震学习笔记 | Python读写segy数据 | segyio示例
date: 2024-02-27 23:11:00
tags: [Study,Soft]
categories: [Study]
mathjax: false
---

如果你读过[Anaconda + VScode | Python环境安装与管理](https://cocklebur0924.github.io/2024/01/21/Python01_install/), 相信你已经配置好了python环境，本文介绍利用Python读写segy数据。

[segyio](https://segyio.readthedocs.io/en/latest/)是一个可以读写segy文件的python库。其详细说明可以参考[官方文档](https://segyio.readthedocs.io/en/latest/index.html)或[Github主页](https://github.com/equinor/segyio)。

这里仅介绍一个segyio使用示例。


### 1. 准备数据

本文以Marmousi模型为例，读取segy文件。

首先，下载[marmousi模型示例数据](https://wiki.seg.org/wiki/Open_data#2D_synthetic_seismic_data)。



### 2. 安装segyio

首先, 打开Anaconda Prompt, 创建一个环境seismic (也可以在之前创建好的环境/base环境中直接安装) ：

```
conda create -n seismic python=3.11
```

激活该环境：

```
conda activate seismic 
```

安装segyio：

```
pip install segyio
```

安装画图用的matplotlib库:

```
pip install matplotlib #或conda install matplotlib
```

### 3. 读取segy文件

打开vscode，创建.ipynb文件，选择已安装好segyio的环境seismic。

```
import segyio
import matplotlib.pyplot as plt
import numpy as np
```

```
def read_segy(data_dir,shotnum=0):
    with segyio.open(data_dir,'r',ignore_geometry=True) as f:
        sourceX = f.attributes(segyio.TraceField.SourceX)[:]
        shot_p = sourceX.copy()
        trace_num = len(sourceX) #number of trace, The sourceX under the same shot is the same character.
        if shotnum:
            shot_num = shotnum 
        else:
            shot_num = len(set(sourceX)) #shot number 
        len_shot = trace_num//shot_num   #The length of the data in each shot data
        time = f.trace[0].shape[0]
        '''
        The data of each shot is read separately
        The default is that the data dimensions collected by all shots in the file are the same.
        Jump=1, which means that the data of all shots in the file is read by default. 
        When jump=2, it means that every other shot reads data.
        '''
        print('start read segy data')
        data = np.zeros((shot_num,time,len_shot))
        for j in range(0,shot_num):
            data[j,:,:] = np.asarray([np.copy(x) for x in f.trace[j*len_shot:(j+1)*len_shot]]).T
        return data
```

读入segy文件（前面下载好的marmousi模型）

```
segyfile = '../elastic-marmousi-model/model/MODEL_P-WAVE_VELOCITY_1.25m.segy'
marmousi = read_segy(segyfile,shotnum=1)
print(marmousi.shape)
```

画图：

```
plt.figure(figsize=(15,3))
plt.imshow(marmousi[0])
plt.colorbar()
```

![Marmousi model](/images/Soft/segy_marmousi_model.png)

试一下另一个文件：

```
segyfile1 = '../elastic-marmousi-model/processed_data/SEGY-Depth/SYNTHETIC.segy'
marmousi_depth = read_segy(segyfile1,shotnum=1)
print(marmousi_depth.shape)
plt.figure(figsize=(10,5))
plt.imshow(marmousi_depth[0],vmin=-0.01,vmax=0.01,aspect=1)
```

![Marmousi data](/images/Soft/segy_marmousi_depth.png)

### 4. 写入segy文件

创建一个简单的一维数组（可以当成速度），并扩展到二维，画图：

```
velo = np.arange(1000,5000,100)

def one_to_2d(v_1d, nx =1000):
    return np.ones((v_1d.shape[0],nx))*v_1d.reshape(v_1d.shape[0],1)

velo2d = one_to_2d(velo)
plt.figure(figsize=(8,3))
plt.imshow(velo2d,aspect=10)
plt.colorbar()
```

定义写入segy的函数：
```
def creat_velocity_segy(vel, dir, dx = 10, dt = 10):
    seis = vel
    spec = segyio.spec()
    spec.samples = list(np.int32(np.arange(vel.shape[0])* dt))
    spec.format = 5
    spec.tracecount =vel.shape[1]
    with segyio.create(dir, spec) as f:
    ## fill the file with data
        for i in range(vel.shape[1]):
            t = i 
            f.trace[t] = seis[:,i]
            f.header[t][segyio.TraceField.FieldRecord] = 0
            f.header[t][segyio.TraceField.ShotPoint] = 0
            f.header[t][segyio.TraceField.TraceNumber] = i+1
            f.header[t][segyio.TraceField.ReceiverGroupElevation] = 10
            f.header[t][segyio.TraceField.GroupX] = i*dx
            f.header[t][segyio.TraceField.SourceX] = i*dx
            f.header[t][segyio.TraceField.TRACE_SAMPLE_INTERVAL] = 10
```

写入segy文件：

```
savepath = './savepath/velomodel.segy'
creat_velocity_segy(velo2d,dir=savepath,dx=10,dt=10)
```

读取刚刚写的segy文件，画图检查是否正确：

```
modelcheck = read_segy(savepath,shotnum=1)
plt.figure(figsize=(8,3))
plt.imshow(modelcheck[0],aspect=10)
plt.colorbar()
```

![model check](/images/Soft/segy_model_check.png)


