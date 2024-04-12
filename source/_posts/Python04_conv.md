---
title: 深度学习 | 卷积、转置卷积输出特征大小计算器 | python
date: 2024-04-12 23:07:00
tags: [Study,Soft]
categories: [Study]
mathjax: False
---


注意：本文仅计算输出特征大小（即边长），并不真正进行卷积运算！


## 卷积输出特征大小公式

Output_size = (Input_size + 2*Padding - Kernalsize)/Stride + 1

其中，Input_size: 输入特征大小
Padding：Padding（边缘补充大小）
Kernalsize：卷积核大小
Stride：步长

对于长和宽，计算公式是一样的。

## 转置卷积输出特征大小公式

转置卷积是卷积的反过程，可以推算一下：

Output = (Input - 1) * Stride + Kernalsize - 2 * Padding


## python代码实现
```
def cnn(input_size, kernelsize, stride, padding):
    if isinstance(input_size, int):
        input_size = (input_size,)
    if isinstance(kernelsize, int):
        kernelsize = (kernelsize,) * len(input_size)
    if isinstance(stride, int):
        stride = (stride,) * len(input_size)
    if isinstance(padding, int):
        padding = (padding,) * len(input_size)

    output = tuple((i - k + 2 * p) // s + 1 for i, k, s, p in zip(input_size, kernelsize, stride, padding))
    print(output)
    return output

def transcnn(input_trans, kernelsize, stride, padding):
    if isinstance(input_trans, int):
        input_trans = (input_trans,)
    if isinstance(kernelsize, int):
        kernelsize = (kernelsize,) * len(input_trans)
    if isinstance(stride, int):
        stride = (stride,) * len(input_trans)
    if isinstance(padding, int):
        padding = (padding,) * len(input_trans)

    output_trans = tuple((i - 1) * s + k - 2 * p for i, k, s, p in zip(input_trans, kernelsize, stride, padding))
    print(output_trans)
    return output_trans
```


## 测试

例如，输入数据维度为（270，61），期望输出维度为（650，801），通过不断的trial-and-error，确定需要的卷积层和转置卷积层(该例中转置卷积层后拼了一个卷积层):

```
input_size = (270, 61)
print('Input size = ', input_size)

print('----------- Start CNN -----------')
cnn1 = cnn(input_size, kernelsize=(3), stride=(1), padding=(0))
cnn2 = cnn(cnn1, kernelsize=(3), stride=(1), padding=(0))
cnn3 = cnn(cnn2, kernelsize=(3), stride=(2), padding=(0))
cnn4 = cnn(cnn3, kernelsize=(3), stride=(1), padding=(0))
cnn5 = cnn(cnn4, kernelsize=(3), stride=(1), padding=(0))
cnn6 = cnn(cnn5, kernelsize=(3), stride=(2), padding=(0))
cnn7 = cnn(cnn6, kernelsize=(3), stride=(1), padding=(0))
cnn8 = cnn(cnn7, kernelsize=(3), stride=(1), padding=(0))
cnn9 = cnn(cnn8, kernelsize=(3), stride=(2), padding=(0))
cnn10 = cnn(cnn9, kernelsize=(3), stride=(1), padding=(0))
output_cnn = cnn(cnn10, kernelsize=(3,1), stride=(1), padding=(0))
print("Output of cnn:", output_cnn)

print('---------- Start TransCNN -------')
transcnn1 = transcnn(output_cnn, kernelsize=(5,3), stride=(3,3), padding=(0))
transcnn1 = cnn(transcnn1, kernelsize=(3), stride=(1), padding=(1))
transcnn2 = transcnn(transcnn1, kernelsize=(5), stride=(2,3), padding=(0))
transcnn2 = cnn(transcnn2, kernelsize=(3), stride=(1), padding=(1))
transcnn3 = transcnn(transcnn2, kernelsize=(5,3), stride=(2,3), padding=(0))
transcnn3 = cnn(transcnn3, kernelsize=(3), stride=(1), padding=(1))

transcnn4 = transcnn(transcnn3, kernelsize=(5,3), stride=(2,3), padding=(0))
transcnn4 = cnn(transcnn4, kernelsize=(3), stride=(1), padding=(1))

transcnn5 = transcnn(transcnn4, kernelsize=(3), stride=(1,2), padding=(0))
transcnn5 = cnn(transcnn5, kernelsize=(3), stride=(1), padding=(1))

transcnn6 = transcnn(transcnn5, kernelsize=(3), stride=(1,2), padding=(0))
transcnn6 = cnn(transcnn6, kernelsize=(3), stride=(1), padding=(1))

transcnn7 = transcnn(transcnn6, kernelsize=(3,3), stride=(1,2), padding=(0))
transcnn7 = cnn(transcnn7, kernelsize=(3), stride=(1), padding=(1))

transcnn8 = transcnn(transcnn7, kernelsize=(6,3), stride=(1,1), padding=(0))
transcnn8 = cnn(transcnn8, kernelsize=(3), stride=(1), padding=(1))

transcnn9 = transcnn(transcnn8, kernelsize=(5,3), stride=(1,1), padding=(0))
transcnn9 = cnn(transcnn9, kernelsize=(3), stride=(1), padding=(1))

output_transcnn = cnn(transcnn9, kernelsize=(3), stride=(1), padding=(0))
print("Output of transcnn:", output_transcnn)

print("Output size = ", output_transcnn)

```
