---
title: 地震学习笔记 | 一维合成地震记录 | Matlab代码
date: 2022-10-13 21:14:10
tags: [Study,Seismic]
categories: [Study]
mathjax: true
---

### 褶积公式

1952年，Enders Robinson，Norbert Wiener，Norman Levinson与Paul Samuelson等提出了反射地震褶积模型的概念。$^{[1]}$

反射地震褶积模型认为，地震记录$s(n)$是震源地震子波$w(t)$和地下地层反射系数$r(t)$褶积的结果。其数学表达式，即褶积公式为：$^{[1][2]}$

$$s(n)=w(n)*r(n)$$

其中*代表褶积运算的符号。

下面利用褶积公式合成一维地震记录：

设计反射系数$r(n) (n=500)$，
其中$r(100)=0.5,r(200)=-0.6,r(300)=0.7,r(400)=0.3,r(500)=0.4$，其余为0。

地震子波选用雷克子波。

### 合成地震记录的Matlab实现

```matlab
%% Synthetic Seismic Record
function [] = SyntheticSeismic()
clc;
clear;
close all;
%% Reflection Coefficient
ref=zeros(1,500);
ref(100)=0.5;
ref(200)=-0.6;
ref(300)=0.7;
ref(400)=0.3;
ref(500)=0.4;

%% Ricker Wavele
[wavelet_zero,timez] = Ricker_zero(20,0.002,100,3); %中心频率20Hz,子波宽度r=3
[wavelet_min,timem] = Ricker_minimum(20,0.002,100,3); %中心频率20Hz,子波宽度r=3

%% Calculate Synthetic Seismic Record
SeisRecord_zero = conv(ref,wavelet_zero);
SeisRecord_min = conv(ref,wavelet_min);

%% plot 
x1=1:length(ref)+length(wavelet_zero)-1;
x2=1:length(ref)+length(wavelet_min)-1;
subplot(4,1,1)
plot(timez,wavelet_zero,'b',timem,wavelet_min,'r:','linewidth',2.0);
title('Ricker Wavelet','FontSize',14);
legend('Ricker-zero','Ricker-minimum');
set(gca,'FontSize',12,'XLim',[0 0.1],'YLim',[-1.0 1.0]);
xlabel('Time(s)'),ylabel('Amplitude')
hold on
subplot(4,1,2)
x3=1:length(ref);
stem(x3,ref,'k');xlabel('depth'),ylabel('value')
title('Reflection Coefficient','FontSize',14);
axis([-inf inf -1,1]);
set(gca,'FontSize',12);
grid on;
hold on
subplot(4,1,[3,4])
plot(x1,SeisRecord_zero,'b',x2,SeisRecord_min,'r:','linewidth',2.0);
%title('Seismic Record','FontSize',14);
text(50,0.6,'Seismic Record','FontSize',15);
legend('Record(Ricker-zero)','Record(Ricker-minimum)');
set(gca,'FontSize',12,'YLim',[-0.75 0.75]);
xlabel('sampling sequence'),ylabel('Amplitude')
set(gcf,'unit','normalized','position',[0.1,0.0,0.5,0.9]);

%% 
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

<center><img src=/images/Seismic/SeismicRecord.png width=80% /></center>
<center>一维合成地震记录</center>

### Reference

[1] [Blibli长江大学：地震勘探原理公开课--什么是合成地震记录](https://www.bilibili.com/video/BV1A4411V7DS?p=15&vd_source=9a85818443b28af8b61e2429ac4bc07c)

[2] 陆基孟，地震勘探原理，第三版