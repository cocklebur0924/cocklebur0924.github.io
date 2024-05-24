---
title: python 并行计算 | 利用multiprocessing的多进程并行实现
date: 2024-05-24 20:07:00
tags: [Study,Soft]
categories: [Study]
mathjax: False
---

本文安排如下：

一，python多线程与多进程并行介绍

二，多进程并行实现（多输入输出函数）


## python 并行计算介绍

python有两种方式实现并行计算，1是多线程并行（threading），2是多进程并行（multiprocessing）,以下为两种方法的简单介绍：

[From chatgpt]

对于 I/O 密集型任务，threading 可以显著减少执行时间。

对于 CPU 密集型任务，multiprocessing 是更好的选择，因为它能够充分利用多核处理器的能力。

在编写并行代码时，需要考虑任务的类型和并行化的成本，选择合适的并行技术。

Tips:

### I/O密集型

I/O（输入/输出）密集型任务主要依赖于系统的I/O操作，比如读写文件、网络通信、数据库访问等。这些任务的性能瓶颈通常在于I/O操作的速度，而不是CPU的处理能力。

特点：

1. 等待时间长：程序需要等待外部设备（如磁盘、网络）完成操作。

2. CPU利用率低：CPU大部分时间处于等待状态，可以在等待期间执行其他任务。

3. 并行化收益高：由于I/O操作的等待时间较长，使用多线程可以在等待I/O的同时处理其他任务，从而提高效率。


### CPU密集型

CPU密集型任务主要依赖于CPU的处理能力，如计算、数据处理、图像渲染等。这些任务的性能瓶颈在于CPU的计算速度。

特点：

1. 计算量大：需要大量的数学计算和数据处理。

2. CPU利用率高：CPU几乎始终处于忙碌状态。

3. 并行化依赖多核：由于GIL（全局解释器锁）的存在，Python的线程对于CPU密集型任务并不高效。使用多进程可以真正实现并行化，充分利用多核处理器。


## 多进程并行实现

假设需要并行计算的函数（yourfunction）有4个输入，3个输出。

该函数需要计算10*2次（比如每运行10次的结果保存一个文件，共生成2个文件）

如果使用for循环，描述如下：

```
import time
import tqdm
import numpy as np

def yourfunction(parameter1,parameter2, parameter3, parameter4):
    output1, output2, output3 = parameter1*sum(t * t for t in range(10000000)), parameter2, parameter3+parameter4  # 模拟CPU密集型计算
    return output1, output2, output3


if __name__ == '__main__':


    # 输入参数
    parameter1 = 1   
    parameter2 = 2
    parameter3 = 3
    parameter4 = 4  

    start_time = time.time()
    for num in tqdm.tqdm(range(2)): #for num in range(2):
        label = []
        feature = []
        maybeother = []

        for i in tqdm.tqdm(range(10)):
            output1, output2, output3 = yourfunction(parameter1,parameter2, parameter3, parameter4)
            label.append(output1)
            feature.append(output2)
            maybeother.append(output3)


        label = np.array(label,dtype='float32')
        feature = np.array(feature,dtype='float32')
        maybeother = np.array(maybeother, dtype='float32')

        #np.save('../trainset/label_para_%d'%num,label)
        #np.save('../trainset/feature_para_%d'%num,feature)
    end_time = time.time()
    print(f"for循环计算耗时: {end_time - start_time} 秒")

    print("Label shape:", label.shape)
    print("Feature shape:", feature.shape)
    print("maybeother shape:", maybeother.shape)
    

```
运行结果：

```
100%|███████████████| 10/10 [00:59<00:00,  5.94s/it]
100%|███████████████| 10/10 [01:05<00:00,  6.52s/it]
100%|███████████████| 2/2 [02:05<00:00, 62.78s/it]
for循环计算耗时: 126.04252409934998 秒
Label shape: (10,)
Feature shape: (10,)
maybeother shape: (10,)
```

下面使用多进程并行实现：

```
import time
import tqdm
import numpy as np
import multiprocessing


def yourfunction(parameter1,parameter2, parameter3, parameter4):
    output1, output2, output3 = parameter1*sum(t * t for t in range(10000000)), parameter2, parameter3+parameter4  # 模拟CPU密集型计算
    return output1, output2, output3

def generate_data(args):
    i, parameter1, parameter2, parameter3, parameter4 = args    
    output1, output2, output3 = yourfunction(parameter1,parameter2, parameter3, parameter4)
    return output1, output2, output3


if __name__ == '__main__':


    num_cores = multiprocessing.cpu_count()
    num_cores_use = num_cores    # 或者 小于num_cores的一个数值
    print(f"本地计算机有 {num_cores} 个 CPU 核心。")
    print(f"并行计算将使用 {num_cores_use} 个 CPU 核心。")

    # 输入参数
    parameter1 = 1   
    parameter2 = 2
    parameter3 = 3
    parameter4 = 4  

    start_time = time.time()
    for num in tqdm.tqdm(range(2)):
        label = []
        feature = []
        maybeother = []

        # ****************** Parallel compute *************************
        # 构建参数列表
        args_list = [(i, parameter1, parameter2 , parameter3, parameter4) for i in range(10)]   # model number saved in each file 

        
        # 使用多进程池
        with multiprocessing.Pool(processes=num_cores_use) as pool:
            results = []
            for result in tqdm.tqdm(pool.imap_unordered(generate_data, args_list), total=len(args_list)):
                results.append(result)
        
        
        
        # 分离结果
        for output1, output2, output3 in results:
            label.append(output1)
            feature.append(output2)
            maybeother.append(output3)
        

        label = np.array(label, dtype='float32')
        feature = np.array(feature, dtype='float32')
        maybeother = np.array(maybeother, dtype='float32')

    
        #np.save('../trainset/label_para_%d'%num,label)
        #np.save('../trainset/feature_para_%d'%num,feature)
    end_time = time.time()
    print(f"并行计算耗时: {end_time - start_time} 秒")

    print("Label shape:", label.shape)
    print("Feature shape:", feature.shape)
    print("maybeother shape:", maybeother.shape)
    

```
运行结果：

```
本地计算机有 12 个 CPU 核心。
并行计算将使用 12 个 CPU 核心。
100%|█████| 10/10 [00:21<00:00,  2.20s/it]
100%|█████| 10/10 [00:20<00:00,  2.01s/it]
100%|█████| 2/2 [00:44<00:00, 22.31s/it] 
并行计算耗时: 45.013415575027466 秒
Label shape: (10,)
Feature shape: (10,)
maybeother shape: (10,)
```

如果将函数修改为：

```
def yourfunction(parameter1,parameter2, parameter3, parameter4):
    output1, output2, output3 = parameter1, parameter2, parameter3+parameter4  
    return output1, output2, output3
```

会发现，并行计算的时间比普通for循环时间还要长，这是因为对于这个简单的函数，该任务就变成I/O密集型了。

