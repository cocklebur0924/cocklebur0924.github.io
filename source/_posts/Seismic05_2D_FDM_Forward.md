---
title: 地震学习笔记 | 二维声波方程有限差分正演 | 2D FDM Seismic Forward
date: 2023-02-14 21:24:57
tags: [Study,Seismic]
categories: [Study]
mathjax: true
---

本文详细推导了二维声波方程有限差分（Finite Difference Method,FDM）格式，提供了空间精度为2-16之间任意偶数阶的c++代码（时间精度2阶）。

## FDM正演公式

### 二维声波方程有限差分格式

在均匀介质中，二维声波方程的表达式为：

$$ \frac{\alpha^2u}{\alpha t^2}=v^2(\frac{\alpha^2u}{\alpha x^2}+\frac{\alpha^2u}{\alpha z^2})+s(t) \tag{1}$$

其中，$s(t)$为震源函数，$v$为声波速度。

令$u_{i,j}^k=u(idx,jdz,kdt)$,即在空间中选定合适的$dx,dz$为步长的空间进行网格化，而在时间上选定合适的$dt$为步长对时间进行离散化。

下面以时间二阶，空间二阶为例，对$u_{i,j}^{k+1}$和$u_{i,j}^{k-1}$利用Taylor公式展开可以得到：

$$u_{i,j}^{k+1} = u_{i,j}^{k}+\frac{\alpha u}{\alpha t}\Delta t+\frac{1}{2!}\frac{\alpha^2u}{\alpha t^2}\Delta t^2+\omicron(\Delta t^2) \tag{2}$$

$$u_{i,j}^{k-1} = u_{i,j}^{k}-\frac{\alpha u}{\alpha t}\Delta t+\frac{1}{2!}\frac{\alpha^2u}{\alpha t^2}\Delta t^2+\omicron(\Delta t^2) \tag{3}$$

将上面两式相加得到$u$对时间$t$的二阶偏导：

$$\frac{\alpha^2 u}{\alpha t^2} = \frac{u_{i,j}^{k+1}+u_{i,j}^{k-1}-2u_{i,j}^{k}}{\Delta t^2}  \tag{4}$$ 


同理，可以得到$u$对$x,z$的二阶偏导：

$$\frac{\alpha^2 u}{\alpha x^2} = \frac{u_{i+1,j}^{k}+u_{i-1,j}^{k}-2u_{i,j}^{k}}{\Delta x^2}  \tag{5}$$ 

$$\frac{\alpha^2 u}{\alpha z^2} = \frac{u_{i,j+1}^{k}+u_{i,j-1}^{k}-2u_{i,j}^{k}}{\Delta z^2}  \tag{6}$$

将公式$(4)-(6)$回代入二维声波方程$(1)$，得到二维声波方程在时间二阶、空间二阶的有限差分格式：

$$\frac{u_{i,j}^{k+1}+u_{i,j}^{k-1}-2u_{i,j}^{k}}{\Delta t^2}=v^2(\frac{u_{i+1,j}^{k}+u_{i-1,j}^{k}-2u_{i,j}^{k}}{\Delta x^2} + \frac{u_{i,j+1}^{k}+u_{i,j-1}^{k}-2u_{i,j}^{k}}{\Delta z^2})+s(t) \tag{7-1}$$

化简：

$$ {u_{i,j}^{k+1}+u_{i,j}^{k-1}-2u_{i,j}^{k}} = \frac{v^2 \Delta t^2}{\Delta x^2}(u_{i+1,j}^{k}+u_{i-1,j}^{k}-2u_{i,j}^{k}) + \frac{v^2 \Delta t^2}{\Delta z^2}(u_{i,j+1}^{k}+u_{i,j-1}^{k}-2u_{i,j}^{k})+s(t) \tag{7-2}$$

$$ u_{i,j}^{k+1} = 2u_{i,j}^{k} - u_{i,j}^{k-1} + \frac{v^2 \Delta t^2}{\Delta x^2}(u_{i+1,j}^{k}+u_{i-1,j}^{k}-2u_{i,j}^{k}) + \frac{v^2 \Delta t^2}{\Delta z^2}(u_{i,j+1}^{k}+u_{i,j-1}^{k}-2u_{i,j}^{k})+s(t) \tag{7-3}$$

同理可得到二维声波方程在时间二阶，空间2N阶的有限差分格式：

$$ u_{i,j}^{k+1} = 2u_{i,j}^{k} - u_{i,j}^{k-1} + \frac{v^2 \Delta t^2}{\Delta x^2}\sum_{p=1}^{N}C_p(u_{i+p,j}^{k}+u_{i-p,j}^{k}-2u_{i,j}^{k}) + \frac{v^2 \Delta t^2}{\Delta z^2}\sum_{p=1}^{N}C_p(u_{i,j+p}^{k}+u_{i,j-p}^{k}-2u_{i,j}^{k})+s(t) \tag{8-1}$$

若化简合并$u_{i,j}^k$，则：

$$ u_{i,j}^{k+1} = 2u_{i,j}^{k} - u_{i,j}^{k-1} + \frac{v^2 \Delta t^2}{\Delta x^2} \sum_{p=0}^{N}C_p(u_{i+p,j}^{k}+u_{i-p,j}^{k} + u_{i,j+p}^{k}+u_{i,j-p}^{k}-2u_{i,j}^{k})+s(t)) \tag{8-2}$$

其中二阶导数的偶数阶精度有限差分系数如下表所示：

| 精度(阶数) | $C_0$ | $C_1$ | $C_2$ | $C_3$ | $C_4$ | $C_5$ | $C_6$ | $C_7$ | $C_8$ |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
|  2  | -2 | 1 |
|  4  | $-\frac{5}{2}$ | $\frac{4}{3}$ | $-\frac{1}{12}$ |
|  6  | $-\frac{49}{18}$ | $\frac{3}{2}$ | $-\frac{3}{20}$ | $\frac{1}{90}$ |
|  8  | $-\frac{205}{72}$ | $\frac{8}{5}$ | $-\frac{1}{5}$ | $\frac{8}{315}$ | $-\frac{1}{560}$ |
|  10  | $-\frac{5269}{1800}$ | $\frac{5}{3}$ | $-\frac{5}{21}$ | $\frac{5}{126}$ | $-\frac{5}{1008}$ | $\frac{1}{3150}$ |
|  12  | $-\frac{5369}{1800}$ | $\frac{12}{7}$ | $-\frac{15}{56}$ | $\frac{10}{189}$ | $-\frac{1}{112}$ | $\frac{2}{1925}$ | $-\frac{1}{16632}$ |
|  14  | $-\frac{266681}{88200}$ | $\frac{7}{4}$ | $-\frac{7}{24}$ | $\frac{7}{108}$ | $-\frac{7}{528}$ | $\frac{7}{3300}$ | $-\frac{7}{30888}$ | $\frac{1}{84084}$ |
|  16  | $-\frac{1077749}{352800}$ | $\frac{16}{9}$ | $-\frac{14}{45}$ | $\frac{112}{1485}$ | $-\frac{7}{396}$ | $\frac{112}{32175}$ | $-\frac{2}{3861}$ | $\frac{16}{315315}$ | $-\frac{1}{411840}$ |

**Notice：在实际编程中，采用式$(8-1)$的形式，因此上表中$C_0$项省略**

### 震源项

采用雷克子波作为震源函数，其在时间域的表达式为：

$$s(t)=e^{-(2\pi f/ r)^2t^2}cos(2\pi ft) \tag{9}$$

对时间进行离散化后：

$$s(k\Delta t)=e^{-(2\pi f/ r)^2k\Delta t^2}cos(2\pi fk\Delta t) \tag{10}$$

关于雷克子波的更多介绍，可参考[地震学习笔记 | 雷克子波的认识 | Matlab代码](https://cocklebur0924.github.io/2022/10/11/Seismic02_RickerWavelet/)。


##  c++程序实现

```cpp
// 2D FDM Seismic Forward (without Absorting Boundary)
// Spatial accuracy: N = 2 - 16 (even number)
// Author：Cocklebur
// 2023 

#include<stdio.h>
#include<math.h>
#include<stdlib.h> 
#include <vector>
#include <iostream>
using namespace std;
#pragma warning(disable : 4996)

// Define Constant 
#define PI 3.14159265359
#define nx 400                                  //网格点x(m)；
#define nz 400                                  //网格点z(m)；
#define dh 4                                    //空间步长dh(m)；
#define dt 0.00025                              //时间步长dt(s)；
#define Sx 200                                  //震源位置x(m)；
#define Sz 100                                  //震源位置z(m)；
#define F 20                                    //震源主频f(Hz)：15-25Hz；
#define R 3                                     //震源Lamda取值范围2-4；
#define Kmax 1000                               //时间循环次数；

// 建 立 二 维 动 态 数 组 函 数 
double**dimension2(int x, int y)
{
	int i;
	double **m;
	m = (double**)malloc(x * sizeof(double*));
	for (i = 0; i < x; i++)
	{
		m[i] = (double*)malloc(y*sizeof(double));
	}
	return m;
}

// 矩 阵 交 换 赋 值 函 数 
void exchange(double **u1, double **u2, double **u3)  
{
	int i, j;
	for (i = 0; i<nx; i++)
		for (j = 0; j<nz; j++)
			u1[i][j] = u2[i][j];
	for (i = 0; i<nx; i++)
		for (j = 0; j<nz; j++)
			u2[i][j] = u3[i][j];
}

// ************** Main Program Start **************
int main()
{
// 定 义 变 量 
	FILE *fp1, *fp2;                               // fp1存放波场值，fp2存放地震记录
	int i, j, k, delta, N;                         // N为空间精度；delta函数控制是否加载震源项
	double v;                                      // 速度模型v；
	double** u1, ** u2, ** u3, s[Kmax], ** rec;    // 波场值u1(past),u2(now),u3(future)；震源函数s；地震记录rec；
	double t1, t2, A;
	u1 = dimension2(nx,nz);
	u2 = dimension2(nx,nz);
	u3 = dimension2(nx,nz);
	rec = dimension2(nx, Kmax);

// 选 择 声 波 方 程 空 间 阶 数 
	printf("Please enter the accuracy(support：2-16 even number):\n");
	scanf("%d", &N);
// 选 择 想 要 输 出 波 场 值 的 时 刻
	printf("Which iter of times do you want to output?\n");
	//scanf("%d", &wave);
	vector < int > wave;
	int  p = 0;
	do {
		cin >> p;
		wave.push_back(p);
	} while (getchar() != '\n');
	cout << "---The wave field will be output at these moments(iter*dt) :---" << endl;
	for (int p = 0; p < wave.size(); p++)
	{
		cout << "when time = " << dt*wave.at(p) << endl;
	}

// 波 场 值 赋 初 值
	for (i = 0; i < nx; i++)
		for (j = 0; j < nz; j++)
		{
			u1[i][j] = 0;
			u2[i][j] = 0;
			u3[i][j] = 0;
		}

	for (i = 0; i < nx; i++)
		for (int k = 0; k < Kmax; k++) 
		{
			rec[i][k] = 0;
		}

// 震 源 函 数
	for (k = 0; k < Kmax; k++)                       
	{
		s[k] = exp(-pow(2*PI*F/R, 2)*pow(k*dt, 2))*cos(2*PI*F*k*dt);
	}
	
// 定 义 速 度 模 型
	for (i = 0; i < nx; i++)
		for (j = 0; j < nz; j++) 
		{
			if (j < 200)                            
				v = 2000;
			else
				v = 2500;
		}

// N 阶 空 间 精 度 波 场 值 计 算
	
	for (k = 0; k < Kmax; k++)
	{
		for (i = N/2; i < nx - N/2; i++)
		{
			for (j = N/2; j < nz - N/2; j++)
			{

				if (i == Sx && j == Sz)
					delta = 1;
				else
					delta = 0;

				//空间精度2-16阶的系数矩阵
				double c[8][8] = 
				{ 
					{1.0},
					{4/3.0,-1/12.0},
					{3/2.0,-3/20.0,1/90.0},
					{8/5.0,-1/5.0,8/315.0,-1/560.0},
					{5/3.0,-5/21.0,5/126.0,-5/1008.0,1/3150.0},
					{12/7.0,-15/56.0,10/189.0,-1/112.0,2/1925.0,-1/16632.0},
					{7/4.0,-7/24.0,7/108.0,-7/528.0,7/3300.0,-7/30888.0,1/84084.0},
					{16/9.0,-14/45.0,112/1485.0,-7/396.0,112/32175.0,2/3861.0,16/315315.0,-1/411840.0},
				};

				A = 0;
				for (int p = 1; p <= N/2; p++)
				{
					t1 = c[N/2-1][p-1] * (u2[i + p][j] + u2[i - p][j] - 2 * u2[i][j]);
					t2 = c[N/2-1][p-1] * (u2[i][j +p] + u2[i][j-p] - 2 * u2[i][j]);
					A += t1 + t2;
				}
				 
				u3[i][j] = 2 * u2[i][j] - u1[i][j] + pow(v * dt / dh, 2) * A + s[k] * delta;

				if (j == Sz)
					rec[i][k] = u3[i][Sz];                  // 输出地震记录；
			}

		}


		for (int p = 0; p < wave.size(); p++)
		{

			if ((k + 1) == wave.at(p))                                      // K = wave 时波场值输出；
			{
				char filename[20];
					sprintf(filename, "wavefront_%d.dat", wave.at(p));

					if (fp1 = fopen(filename, "w"))
					{
						for (i = 0; i < nx; i++)
							for (j = 0; j < nz; j++)
							{
								fprintf(fp1, "%lf\t", u3[i][j]);
									if (j == nz - 1)
										fprintf(fp1, "\n");
							}
						fclose(fp1);
					}
			}
		}

		exchange(u1, u2, u3);
		if ((k + 1) % 100 == 0)
		{
			printf("No.%d has been completed!\n", k+1);
		}
	}

	if (fp2 = fopen("record.dat", "w"))
	{
		for (i = 0; i < nx; i++)
			for (j = 0; j < Kmax; j++)
			{
				fprintf(fp2,"%lf\t", rec[i][j]);
				if (j == Kmax - 1)
					fprintf(fp2, "\n");
			}
		fclose(fp2);
	}

	for (i = 0; i < nx; i++) 
	{
		free(u1[i]);
		free(u2[i]);
		free(u3[i]);
	}
	return 0;
}

```

## Result

matlab绘制结果，以6阶精度为例。

```matlab
clc;
clear;
close all;

record = load('record.dat');
wavefront_200 = load('wavefront_200.dat');
wavefront_400 = load('wavefront_400.dat');
wavefront_600 = load('wavefront_600.dat');
wavefront_800 = load('wavefront_800.dat');

imagesc(record');
colormap(gray);
xlabel('X');
ylabel('Time');
title('Seismic Record');

figure,
subplot(2,2,1)
imagesc(wavefront_200');
xlabel('X');
ylabel('Z');
title('Wavefront-200iter');
subplot(2,2,2)
imagesc(wavefront_400');
xlabel('X');
ylabel('Z');
title('Wavefront-400iter');
subplot(2,2,3)
imagesc(wavefront_600');
xlabel('X');
ylabel('Z');
title('Wavefront-600iter');
subplot(2,2,4)
imagesc(wavefront_800');
xlabel('X');
ylabel('Z');
title('Wavefront-800iter');
colormap(gray);
```

<center><img src=/images/Seismic/05_SeismicRecord.png width=100% /></center>
<center>地震记录</center>

<center><img src=/images/Seismic/05_wavefront.png width=100% /></center>
<center>波前快照</center>

Notice:由于没有吸收边界，800次迭代时的波场可以看到明显的边界反射。
含吸收边界条件的正演程序可参考[地震学习笔记 | 二维声波方程有限差分正演(吸收边界) | 2D FDM Seismic Forward（ABC）](https://cocklebur0924.github.io/2023/03/07/Seismic06_2D_FDM_Forward_ABC)。


## Reference: 
[1] 陈聪. 叠前逆时偏移及成像[D].中国地质大学（北京）,2011.

[2] 封常青师兄地球物理数据处理课程作业, 2019.
