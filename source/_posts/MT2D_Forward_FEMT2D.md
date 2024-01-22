---
title: FEMT2D | 大地电磁二维正演 | 有限元Matlab开源程序
date: 2022-07-30 15:52:00
tags: [Study,MT]
categories: [Study]
mathjax: true
---

[大地电磁二维正演公式](https://cocklebur0924.github.io/2022/07/10/MT2D_Forward/)中提到，求解二维大地电磁场的问题就是求解下式这样的**偏微分方程**的问题。

$$ \nabla^2E_x-i\omega u_0\sigma E_x = 0 $$

$$ \nabla^2H_x-i\omega u_0\sigma H_x = 0 $$

在开源程序`FEMT2D`中，Börner, R. U提供了二维亥姆霍兹偏微分方程的有限元解法。

### 1. `FEMT2D`程序如何获取

方式1：直接从[Börner, R. U的Github仓库](https://github.com/ruboerner/FEMT2D)里下载。

方式2：利用Git克隆到本地：

```bash
git clone https://github.com/ruboerner/FEMT2D.git
```
下载好后，将得到下列文件：

<div align=center><img src=/images/MT2D/FEMT2D_0.png width="600"></div>

### 2. `FEMT2D`如何使用

在`FEMT2D`程序的README.md文档中，已详细介绍了该程序的使用方法。本文以其中的一个例子作为演示。

**第一步：**

在运行`FEMT2D`程序之前，需要利用`Triangle`进行网格剖分.

该程序的网格剖分在python上进行，需安装[PyGIMLi](http://www.pygimli.org)工具与[Triangle](https://www.cs.cmu.edu/~quake/triangle.html)程序。

<font color=Blue><font size=2>Notice：在安装完成PyGIMLi环境后，需要将triangle.exe放在该环境的目录下。例如，我的PyGIMLi所在的文件夹为'D:\ProgramFiles\Anaconda3\envs\pg'，则应将'triangle.exe'也放在该文件夹中。</font></font>

安装好这两样，下面的就很简单了。

**第二步：**

打开`FEMT2D`程序的Notebooks文件夹，该文件夹包含了所有Demo例子的网格剖分程序。

以第一个例子作为示范，打开'COMMEMI_2D_0.ipynb'（注意切换到PyGIMLi环境），运行该程序，便得到meshes文件夹下的四个网格文件。

```
meshes
├── commemi2d0.1.ele
├── commemi2d0.1.node
├── commemi2d0.1.poly
├── commemi2d0.poly
```

**第三步：**

打开matlab（切换路径至FEMT2D主文件夹），在命令行窗口运行

```matlab
demo.driverCOMMEMI_2D_0
```
就ok了。

下面是该例子的运行结果：

<div align=center><img src=/images/MT2D/FEMT2D_1.png width="600"></div>

<div align=center><img src=/images/MT2D/FEMT2D_2.png width="600"></div>

<div align=center><img src=/images/MT2D/FEMT2D_3.png width="600"></div>

<div align=center><img src=/images/MT2D/FEMT2D_4.png width="600"></div>

<div align=center><img src=/images/MT2D/FEMT2D_5.png width="600"></div>



### References
[1] [Börner, R. U. 的Github repository - FEMT2D](https://github.com/ruboerner/FEMT2D) 

[2] FEMT2D: FE simulation of 2D Magnetotellurics in MATLAB. https://doi.org/10.5281/zenodo.3955407

[3] Jonathan Richard Shewchuk, _Triangle: Engineering a 2D Quality Mesh Generator and Delaunay Triangulator_, in "Applied Computational Geometry: Towards Geometric Engineering" (Ming C. Lin and Dinesh Manocha, editors), volume 1148 of Lecture Notes in Computer Science, pages 203-222, Springer-Verlag, Berlin, May 1996.

[4] Jonathan Richard Shewchuk, _Delaunay Refinement Algorithms for Triangular Mesh Generation_, Computational Geometry: Theory and Applications 22(1-3):21-74, May 2002.

[5] Franke, A., Börner, R. U., & Spitzer, K. (2007). *Adaptive unstructured grid finite element simulation of two-dimensional magnetotelluric fields for arbitrary surface and seafloor topography*. Geophysical Journal International, 171(1), 71-86.

[6] Börner, R. U. (2010). Numerical modelling in geo-electromagnetics: advances and challenges. Surveys in Geophysics, 31(2), 225-245.

[7] Rücker, C., Günther, T., Wagner, F.M., 2017. pyGIMLi: An open-source library for modelling and inversion in geophysics, Computers and Geosciences, 109, 106-123, doi: 10.1016/j.cageo.2017.07.011.