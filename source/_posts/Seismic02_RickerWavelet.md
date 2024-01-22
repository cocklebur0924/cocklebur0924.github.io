---
title: 地震学习笔记 | 雷克子波的认识 | Matlab代码
date: 2022-10-11 21:10:00
tags: [Study,Seismic]
categories: [Study]
mathjax: true
---
### 地震子波的概念
地震子波的概念最早由美国地球物理学家Norman H.Ricker（1896-1980）于1940年提出，他发表在Geophysics上的论文认为，若震源激发的地震波是脉冲信号，向下传播没有变化，那么地震反射波很容易解释地下岩层。Ricker通过地震波动方程推导发现，震源激发的脉冲信号在向下传播的过程中，由于大地的吸收衰减等作用，不再是脉冲信号，而是地震子波。Ricker后来提出了一个理论地震子波，即雷克子波。$^{[1]}$

以炸药震源为例，来看看地震子波是怎么形成的：炸药爆炸时产生一个延续时间极短的尖脉冲，在爆炸点附近的介质中，以冲击波的形式传播，当爆炸脉冲向外传播一定距离以后，地层产生的弹性形变再向外传播，由于介质对高频成分的吸收，波形发生明显的变化，直到传播了更大的距离以后，波形逐渐稳定，形成一个具有两到三个相位，有一定延续时间的地震波，称为地震子波。$^{[1]}$

<center><img src=/images/Seismic/Wavelet_Boom.png width=40% /></center>
<center>爆炸脉冲的变化示意图（参考文献[2]中图2-1-5）</center>

简言之，地震子波是具有确定起始时间（从爆炸时刻开始），有限能量，且有一定延续时间的信号。地震子波是组成地震记录的基本元素。$^{[1][2]}$

### 雷克子波公式

零相位雷克子波：
$$
w(t) = e^{(-2\pi f_m r)^2t^2}cos(2\pi f_m t)
$$
最小相位雷克子波：
$$
w(t) = e^{(-2\pi f_m r)^2t^2}sin(2\pi f_m t)
$$

其中，$f_m$为子波的中心频率，$r$为子波宽度，随着$r$的增大，子波能量后移。

### matlab绘制雷克子波

```matlab
%% Ricker Wavelet
function []=RickerWavelet()
clear;
clc;

[wavelet_zero1,timez1] = Ricker_zero(20,0.002,100,3); %中心频率20Hz,子波宽度r=3
[wavelet_zero2,timez2] = Ricker_zero(10,0.002,100,3); %中心频率10Hz,子波宽度r=3
[wavelet_zero3,timez3] = Ricker_zero(20,0.002,100,5);  %中心频率20Hz,子波宽度r=5

[wavelet_min1,timem1] = Ricker_minimum(20,0.002,100,3); %中心频率20Hz,子波宽度r=3
[wavelet_min2,timem2] = Ricker_minimum(10,0.002,100,3); %中心频率10Hz,子波宽度r=3
[wavelet_min3,timem3] = Ricker_minimum(20,0.002,100,5); %中心频率20Hz,子波宽度r=5
figure(1)
%不同主频的Ricker Wavelet（零相位）对比
subplot(1,2,1)
plot(timez1,wavelet_zero1,'b',timez2,wavelet_zero2,'r-','linewidth',2);
set(gca,'XLim',[0 0.15],'YLim',[-1.0 1.0],'FontSize',13)
legend('主频20Hz,子波宽度r=3','主频10Hz,子波宽度r=3'),title('Ricker Wavelet(zero-phase)'),xlabel('Time(s)'),ylabel('Amplitude')
%不同主频的Ricker Wavelet（最小相位）对比
subplot(1,2,2)
plot(timem1,wavelet_min1,'b',timem2,wavelet_min2,'r-','linewidth',2);
set(gca,'XLim',[0 0.15],'YLim',[-1.0 1.0],'FontSize',13)
set(gcf,'unit','normalized','position',[0.2,0.2,0.7,0.5]);
legend('主频20Hz,子波宽度r=3','主频10Hz,子波宽度r=3'),title('Ricker Wavelet(minimum-phase)'),xlabel('Time(s)'),ylabel('Amplitude')
hold on

figure(2)
%不同子波宽度的Ricker Wavelet（零相位）对比 
subplot(1,2,1)
plot(timez1,wavelet_zero1,'b',timez3,wavelet_zero3,'r-','linewidth',2);
set(gca,'XLim',[0 0.15],'YLim',[-1.0 1.0],'FontSize',13)
legend('主频20Hz,子波宽度r=3','主频20Hz,子波宽度r=5'),title('Ricker Wavelet(zero-phase)'),xlabel('Time(s)'),ylabel('Amplitude')
%不同子波宽度的Ricker Wavelet（最小相位）对比 
subplot(1,2,2)
plot(timem1,wavelet_min1,'b',timem3,wavelet_min3,'r-','linewidth',2);
set(gca,'XLim',[0 0.15],'YLim',[-1.0 1.0],'FontSize',13)
set(gcf,'unit','normalized','position',[0.2,0.2,0.7,0.5]);
legend('主频20Hz,子波宽度r=3','主频20Hz,子波宽度r=5'),title('Ricker Wavelet(minimum-phase)'),xlabel('Time(s)'),ylabel('Amplitude')

function [wavelet_zero,time]=Ricker_zero(freqs,dt,nt,r)
% freqs:主频 
% dt:采样时间间隔
% nt:采样点数
% r: 子波宽度
tmin=0;
tmax=dt*nt;
time=tmin:dt:tmax;
wavelet_zero=exp(-(2*pi*freqs/r)^2*time.^2).*cos(2*pi*freqs.*time);
end

function [wavelet_min,time]=Ricker_minimum(freqs,dt,nt,r)
% freqs:主频 
% dt:采样时间间隔
% nt:采样点数
% r: 子波宽度
tmin=0;
tmax=dt*nt;
time=tmin:dt:tmax;
wavelet_min=exp(-(2*pi*freqs/r)^2*time.^2).*sin(2*pi*freqs.*time);
end

end
```

<center><img src=/images/Seismic/RickerWavelet1.png width=100% /></center>
<center>不同主频的雷克子波</center>



<center><img src=/images/Seismic/RickerWavelet2.png width=100% /></center>
<center>不同子波宽度的雷克子波</center>


### Reference

[1] [Blibli长江大学：地震勘探原理公开课--什么是地震子波](https://www.bilibili.com/video/BV1A4411V7DS/?p=14&vd_source=9a85818443b28af8b61e2429ac4bc07c)

[2] 陆基孟，地震勘探原理，第三版