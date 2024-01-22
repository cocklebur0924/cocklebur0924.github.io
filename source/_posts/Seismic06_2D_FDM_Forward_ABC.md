---
title: 地震学习笔记 | 二维声波方程有限差分正演(吸收边界) | 2D FDM Seismic Forward（ABC）
date: 2023-03-07 20:17:00
tags: [Study,Seismic]
categories: [Study]
mathjax: true
---

[地震学习笔记 | 二维声波方程有限差分正演 | 2D FDM Seismic Forward](https://cocklebur0924.github.io/2023/02/14/Seismic05_2D_FDM_Forward)中提到，若没有吸收边界条件，在波场传播到边界后会发生明显的反射。

本文提供了两种吸收边界条件(Absorbing boundary condition，简称ABC)：

## 吸收边界条件公式

1）海绵吸收边界（Sponge ABC）：

$$ d(u)=exp(-[0.015*(nb-i)]^2) $$
$$ u=x, z(i\Delta x \ or\ i\Delta z) \ \ \  i=1,2,···,nb\tag{1}$$

式中$nb$为吸收边界厚度，通常取20~30，$d(u)$为位置$u$处的吸收系数。

2）PML（Perfectly Matched Layer）吸收边界：

采用余弦型衰减因子：

$$d_{ix} = B[1-cos(\frac{\pi(nb-ix)}{2nb})]  \ \ \   ix=0,1,2,3···,nb \tag{2}$$

$$d_{iz} = B[1-cos(\frac{\pi(nb-iz)}{2nb})]  \ \ \   iz=0,1,2,3···,nb \tag{3}$$

其中，上式分别表示沿着$x,z$方向的衰减因子，$nb$为吸收层的厚度，$B$为衰减幅度因子。

利用衰减因子$d(ix,iz)$表示x-z空间内任意网格点上的衰减因子，可得到基于PML边界条件的高阶有限差分方程：


$$\begin{aligned}
 u_{i,j}^{k+1} = & \frac{1-d^2(i,j)\Delta t^2}{1+d(i,j)\Delta t} 2u_{i,j}^{k} - \frac{1-d(i,j)\Delta t}{1+d(i,j)\Delta t} u_{i,j}^{k-1} \\& + \frac{v^2 \Delta t^2}{(1+d(i,j)\Delta t)\Delta x^2}\sum_{p=1}^{N}C_p(u_{i+p,j}^{k}+u_{i-p,j}^{k}-2u_{i,j}^{k}) \\ &+ \frac{v^2 \Delta t^2}{(1+d(i,j)\Delta t)\Delta z^2}\sum_{p=1}^{N}C_p(u_{i,j+p}^{k}+u_{i,j-p}^{k}-2u_{i,j}^{k})+s(t) 
 \end{aligned} \tag{4}$$

当衰减因子 $d(ix,iz)=0$ 时，该式与[地震学习笔记 | 二维声波方程有限差分正演 | 2D FDM Seismic Forward](https://cocklebur0924.github.io/2023/02/14/Seismic05_2D_FDM_Forward)中式$(8-1)$相等。

本文使用的PML衰减因子如图所示：

<center><img src=/images/Seismic/06_PML_absorbing.png width=70% /></center>
<center>PML衰减因子</center>


##  c++程序实现

```cpp
// 2D FDM Seismic Forward 
// Absorting Boundary Condition:1.Sponge absorbing boundary condition; 2. PML absorbing boundary condition.
// Spatial accuracy: N = 2 - 16 (even number)
// Author：Cocklebur
// 2023 March

#include <stdio.h>
#include <math.h>
#include <cmath>
#include <stdlib.h>
#include <vector>
#include <iostream>
using namespace std;
// #pragma warning(disable : 4996)

// Define Constant
#define PI 3.14159265359
#define nx 400     // 网格点x(m)；
#define nz 400     // 网格点z(m)；
#define nb 30      // 吸收边界宽度(m)
#define dh 4       // 空间步长dh(m)；
#define dt 0.00025 // 时间步长dt(s)；
#define F 20       // 震源主频f(Hz)：15-25Hz；
#define R 3        // 震源Lamda取值范围2-4；
#define Kmax 2000  // 时间循环次数；
#define B 200      // PML边界条件的吸收因子
FILE *fp1, *fp2, *fp3;   // fp1存放波场值，fp2存放地震记录
int nxpad, nzpad, Sx, Sz;
int i, j, k, delta, N;    // N为空间精度；delta函数控制是否加载震源项
double t1, t2, A, ax, az; // A 为计算波场时的中间变量，ax,az为计算PML边界条件时的中间变量
double **u1, **u2, **v, **boundpml, s[Kmax], **rec, **u_rec;
// 波场值u1(past)，u2(now)，速度模型v，boundpml为PML边界，震源函数s，地震记录rec；记录波场值u_rec;


double **dimension2(int x, int y)
/*< Establish two-dimensional dynamic array function >*/
{
    int i;
    double **m;
    m = (double **)malloc(x * sizeof(double *));
    for (i = 0; i < x; i++)
    {
        m[i] = (double *)malloc(y * sizeof(double));
    }
    return m;
}

void exchange(double **p0, double **p1)
/*< matrix transpose >*/
{
    //int i, j;
    double temp;
    for (i = 0; i < nxpad; i++)
        for (j = 0; j < nzpad; j++)
        {
            temp = p0[i][j];
            p0[i][j] = p1[i][j];
            p1[i][j] = temp;
        }
}


void window2d(double **p0, double **p1)
/*< remove the absorbing boundary: window 'p1' to 'p0' >*/
{
    int ix, iz;
    for (ix = 0; ix < nx; ix++)
    {
        for (iz = 0; iz < nz; iz++)
        {
            p0[ix][iz] = p1[nb + ix][nb + iz];
        }
    }
}

void Sponge_absorbing(double **p0, double **p1)
/*< apply Sponge absorbing boundary condition >*/
{
     int ix, iz, ib;
     double bound[nb];
     // 吸 收 边 界 因 子
    for (ib = 0; ib < nb; ib++)
    {
        double tmp = 0.015 * (nb - ib);
        bound[ib] = exp(-tmp * tmp);
    }

    for (ix = 0; ix < nxpad; ix++)
    {
        for (iz = 0; iz < nb; iz++)
        { // top ABC
            p0[ix][iz] = bound[iz] * p0[ix][iz];
            p1[ix][iz] = bound[iz] * p1[ix][iz];
        }
        for (iz = nz + nb; iz < nzpad; iz++)
        { // bottom ABC
            p0[ix][iz] = bound[nzpad - iz - 1] * p0[ix][iz];
            p1[ix][iz] = bound[nzpad - iz - 1] * p1[ix][iz];
        }
    }

    for (iz = 0; iz < nzpad; iz++)
    {
        for (ix = 0; ix < nb; ix++)
        { // left ABC
            p0[ix][iz] = bound[ix] * p0[ix][iz];
            p1[ix][iz] = bound[ix] * p1[ix][iz];
        }
        for (ix = nx + nb; ix < nxpad; ix++)
        { // right ABC
            p0[ix][iz] = bound[nxpad - ix - 1] * p0[ix][iz];
            p1[ix][iz] = bound[nxpad - ix - 1] * p1[ix][iz];
        }
    }
}

void PML_absorbing()
/*< apply PML absorbing boundary condition >*/
{
    int ix, iz;
    for (ix = 0; ix < nxpad; ix++)
        for (iz = 0; iz < nzpad; iz++)
        {
            if (ix >= 0 && ix <= nb && iz >= 0 && iz <= nb)
            {
                ax = B * (1 - cos(PI * (nb - ix) / (2.0 * nb)));
                az = B * (1 - cos(PI * (nb - iz) / (2.0 * nb)));
                boundpml[ix][iz] = ax + az;
            }
            else if (ix > nb && ix < nx + nb && iz >= 0 && iz <= nb)
            {
                ax = 0;
                az = B * (1 - cos(PI * (nb - iz) / (2.0 * nb)));
                boundpml[ix][iz] = ax + az;
            }
            else if (ix > nx + nb - 1 && ix < nxpad && iz >= 0 && iz <= nb)
            {
                ax = B * (1 - cos(PI * (nb - (nxpad - ix - 1)) / (2.0 * nb)));
                az = B * (1 - cos(PI * (nb - iz) / (2.0 * nb)));
                boundpml[ix][iz] = ax + az;
            }
            else if (ix >= 0 && ix <= nb && iz > nb && iz < nz + nb)
            {
                ax = B * (1 - cos(PI * (nb - ix) / (2.0 * nb)));
                az = 0;
                boundpml[ix][iz] = ax + az;
            }
            else if (ix >= 0 && ix <= nb && iz > nz + nb - 1 && iz < nzpad)
            {
                ax = B * (1 - cos(PI * (nb - ix) / (2.0 * nb)));
                az = B * (1 - cos(PI * (nb - (nzpad - iz - 1)) / (2.0 * nb)));
                boundpml[ix][iz] = ax + az;
            }
            else if (ix > nb && ix < nx + nb && iz > nz + nb - 1 && iz < nzpad)
            {
                ax = 0;
                az = B * (1 - cos(PI * (nb - (nzpad - iz - 1)) / (2.0 * nb)));
                boundpml[ix][iz] = ax + az;
            }
            else if (ix > nx + nb - 1 && ix < nxpad && iz > nz + nb - 1 && iz < nzpad)
            {
                ax = B * (1 - cos(PI * (nb - (nxpad - ix - 1)) / (2.0 * nb)));
                az = B * (1 - cos(PI * (nb - (nzpad - iz - 1)) / (2.0 * nb)));
                boundpml[ix][iz] = ax + az;
            }
            else if (ix > nx + nb - 1 && ix < nxpad && iz > nb && iz < nz + nb)
            {
                ax = B * (1 - cos(PI * (nb - (nxpad - ix - 1)) / (2.0 * nb)));
                az = 0;
                boundpml[ix][iz] = ax + az;
            }
        }

    if (fp3 = fopen("Absorbing_PML.dat", "w"))
    {
        for (i = 0; i < nxpad; i++)
            for (j = 0; j < nzpad; j++)
            {
                fprintf(fp3, "%lf\t", boundpml[i][j]);
                if (j == nzpad - 1)
                    fprintf(fp3, "\n");
            }
        fclose(fp3);
    }
}

void forward(double **u1, double **u2)
/*< forward modeling >*/
{
    // 空间精度2-16阶的系数矩阵
    double c[8][8] =
        {
            {1.0},
            {4 / 3.0, -1 / 12.0},
            {3 / 2.0, -3 / 20.0, 1 / 90.0},
            {8 / 5.0, -1 / 5.0, 8 / 315.0, -1 / 560.0},
            {5 / 3.0, -5 / 21.0, 5 / 126.0, -5 / 1008.0, 1 / 3150.0},
            {12 / 7.0, -15 / 56.0, 10 / 189.0, -1 / 112.0, 2 / 1925.0, -1 / 16632.0},
            {7 / 4.0, -7 / 24.0, 7 / 108.0, -7 / 528.0, 7 / 3300.0, -7 / 30888.0, 1 / 84084.0},
            {16 / 9.0, -14 / 45.0, 112 / 1485.0, -7 / 396.0, 112 / 32175.0, 2 / 3861.0, 16 / 315315.0, -1 / 411840.0},
        };

    // calculate 
    for (i = N / 2; i < nxpad - N / 2; i++)
    {
        for (j = N / 2; j < nzpad - N / 2; j++)
        {
            if (i == Sx && j == Sz)
                delta = 1;
            else
                delta = 0;

            A = 0;
            for (int p = 1; p <= N / 2; p++)
            {
                t1 = c[N / 2 - 1][p - 1] * (u2[i + p][j] + u2[i - p][j] - 2 * u2[i][j]);
                t2 = c[N / 2 - 1][p - 1] * (u2[i][j + p] + u2[i][j - p] - 2 * u2[i][j]);
                A += t1 + t2;
            }

            // u1[i][j] = 2 * u2[i][j] - u1[i][j] + pow(v[i][j] * dt / dh, 2) * A + s[k] * delta;

            u1[i][j] = ((2 - pow(boundpml[i][j] * dt, 2)) / (1 + boundpml[i][j] * dt)) * u2[i][j] -
                       ((1 - boundpml[i][j] * dt) / (1 + boundpml[i][j] * dt)) * u1[i][j] + pow(v[i][j] * dt / dh, 2) * A + s[k] * delta;
        }
    }
}

// --------------------------------- Main Program Start --------------------------------//
int main()
{
    // 定 义 变 量
    nxpad = nx + 2 * nb;
    nzpad = nz + 2 * nb;
    Sx = nxpad/2;     // 震源位置x(m)
    Sz = nzpad/2;     // 震源位置z(m)
    u1 = dimension2(nxpad, nzpad);
    u2 = dimension2(nxpad, nzpad);
    v = dimension2(nxpad, nzpad);
    boundpml = dimension2(nxpad, nzpad);
    u_rec = dimension2(nx, nz);
    rec = dimension2(nx, Kmax);

    // ******* Choose the sapce accuracy of Acoustic Eqution *******//
    printf("Please enter the space accuracy(support: 2-16 even number):\n");
    scanf("%d", &N);
    // ************** Choose the output wavefield ******************//
    printf("Which iter of times do you want to output?\n");
    vector<int> wave;
    int p = 0;
    do
    {
        cin >> p;
        wave.push_back(p);
    } while (getchar() != '\n');
    cout << "---The wave field will be output at these moments[iter*dt(s)] :---" << endl;
    for (int p = 0; p < wave.size(); p++)
    {
        cout << "when time = " << dt * wave.at(p) << " s" << endl;
    }
    // ************************ The end **************************//

    // 波场值、速度场、pml边界；记录波场；地震记录赋初值
    for (i = 0; i < nxpad; i++)
        for (j = 0; j < nzpad; j++)
        {
            u1[i][j] = 0;
            u2[i][j] = 0;
            v[i][j] = 0;
            boundpml[i][j] = 0; 
        }
    
    for (i = 0; i < nx; i++)
        for (j = 0; j < nz; j++)
        {
            u_rec[i][j] = 0;
        }

    for (i = 0; i < nx; i++)
        for (int k = 0; k < Kmax; k++)
        {
            rec[i][k] = 0;
        }

    // 震 源 函 数
    for (k = 0; k < Kmax; k++)
    {
        s[k] = exp(-pow(2 * PI * F / R, 2) * pow(k * dt, 2)) * cos(2 * PI * F * k * dt);
    }

    // 定 义 速 度 模 型
    for (i = 0; i < nxpad; i++)
        for (j = 0; j < nzpad; j++)
        {
            if (j < 200)
                v[i][j] = 3000;
            else
                v[i][j] = 3000;
        }

     PML_absorbing();                          // ********应用PML吸收边界*********

    // N 阶 空 间 精 度 波 场 值 计 算
    for (k = 0; k < Kmax; k++)
    {
        forward(u1, u2);
        // Sponge_absorbing(u1, u2);           // *******应用Sponge吸收边界*********
        window2d(u_rec, u1);
        for (int p = 0; p < wave.size(); p++)
        {

            if ((k + 1) == wave.at(p)) // K = wave 时波场值输出；
            {
                char filename[20];
                sprintf(filename, "wavefront_%d.dat", wave.at(p));

                if (fp1 = fopen(filename, "w"))
                {
                    for (i = 0; i < nx; i++)
                        for (j = 0; j < nz; j++)
                        {
                            fprintf(fp1, "%lf\t", u_rec[i][j]);
                            if (j == nz - 1)
                                fprintf(fp1, "\n");
                        }
                    fclose(fp1);
                }
            }
        }
        exchange(u1, u2);
        if ((k + 1) % 100 == 0)
        {
            printf("No.%d has been completed!\n", k + 1);
        }

        for (i = 0; i < nx; i++)
        {
            rec[i][k] = u1[i + nb][Sz]; // 输出地震记录；
        }
    }

    if (fp2 = fopen("record.dat", "w"))
    {
        for (i = 0; i < nx; i++)
            for (j = 0; j < Kmax; j++)
            {
                fprintf(fp2, "%lf\t", rec[i][j]);
                if (j == Kmax - 1)
                    fprintf(fp2, "\n");
            }
        fclose(fp2);
    }

    free(*u1);
    free(u1);
    free(*u2);
    free(u2);
    free(*v);
    free(v);
    free(*u_rec);
    free(u_rec);
    free(*boundpml);
    free(boundpml);
    return 0;
}
```

## Result

<center><img src=/images/Seismic/06_noABC.png width=100% /></center>
<center>无吸收边界波场快照</center>

<center><img src=/images/Seismic/06_Sponge.png width=100% /></center>
<center>海绵吸收边界波场快照</center>

<center><img src=/images/Seismic/06_PML.png width=100% /></center>
<center>PML吸收边界波场快照</center>

## Reference

[1] Yang, Pengliang. (2014). A numerical tour of wave propagation. Madagascar website. 

[2] 黎殿来. 波动方程时域有限差分地震正演建模方法研究[D].电子科技大学,2012.

