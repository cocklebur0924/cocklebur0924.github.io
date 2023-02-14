---
title: Bat Algorithm(BA)大地电磁一维反演 | 蝙蝠算法
date: 2022-04-10 21:17:00
tags: [Study,MT]
categories: [Study]
mathjax: true
---

本系列为大地电磁一维反演的一些matlab程序，搬运自[Mohammad Rheza Zamani的Github项目：1D-Magnetotelluric-Inversion](https://github.com/rhezazz/1D-Magnetotelluric-Inversion) ，共包括以下五种算法：

> 1D-Magnetotelluric-Inversion
> 1. MT FA：[萤火虫算法(Firefly Algorithm，FA)](https://cocklebur0924.github.io/2022/04/10/MT1Dinv_Firefly/)
> 2. MT BA：[蝙蝠算法(Bat algorithm，BA)](https://cocklebur0924.github.io/2022/04/10/MT1Dinv_Bat/)
> 3. MT PSO：[粒子群优化算法(Particle Swarm Optimization，PSO)](https://cocklebur0924.github.io/2022/04/10/MT1Dinv_PSO/)
> 4. MT SA：[模拟退火算法(Simulated Anealling，SA)](https://cocklebur0924.github.io/2022/04/10/MT1Dinv_SA/)
> 5. MT VFSA：[非常快速模拟退火算法(Very Fast Simulated Annealing，VFSA)](https://cocklebur0924.github.io/2022/04/10/MT1Dinv_VFSA/)

本节为`蝙蝠算法 MT1D Inversion matlab代码`

### BA 算法简介

蝙蝠算法（Bat Algorithm，BA）是一种模拟微型蝙蝠回声定位行为的群体智能优化算法，由剑桥大学的Xin-She Yang教授于2010年提出$^{[1]}$。

大多数微型蝙蝠将声音辐射到周围环境，并聆听这些声音来自不同物体的的回声，从而可以识别猎物，躲避障碍物，并追踪黑暗的巢穴。蝙蝠是唯一有翅膀的哺乳动物，它们具有非凡的回声定位能力。蝙蝠是世界上种类第二多的哺乳动物，有超过1200种。一般分为蝙蝠可以分为两类：回声定位微型蝙蝠和以水果为食的巨型蝙蝠。蝙蝠算法是基于第一类蝙蝠的行为而开发的。大多数蝙蝠以倒挂的栖息姿势休息。所有的微型蝙蝠和一些巨型蝙蝠都会发出超声波来产生回声。微型蝙蝠的大脑和听觉神经系统可以通过比较出脉冲和反复出现的回声，对环境产生深入的图像$^{[2]}$。

蝙蝠算法的伪代码如下图所示$^{[1]}$：

<center><img src=/images/PictureInsert/BAT_PseudoCode.png width=75% /></center>
<center>蝙蝠算法伪代码(Yang XS, 2010 Fig.1)</center>

### BA 大地电磁一维反演matlab代码

```matlab
% Bat algorithm for VES Data Inversion
clc;
clear all;
%Model  sintetik
R = [100 10 1000];
thk = [750 1500];
freq = logspace(-3,3,50);
nlayer = length(R);
T = 1./freq;
[app_sin, phase_sin] = modelMT(R, thk ,T);
alpha = 0.9;
gamma = 0.9;
%Parameter inversi
npop = 25;
niter = 500;
%Inisialisasi frekuensi
fmin = 0;
fmax = 2;
%Inisialisasi velocity
%Hitung velocity awal dan posisi awal
for i = 1 : npop
    for imod =  1 : nlayer
        v_rho(i,imod) = 0;
    end
    for imod = 1 : nlayer -1
        v_thk(i,imod) = 0;
    end
end
%Inisialisasi loudness
Amin = 1;
Amax = 2;
for i = 1 : npop
    for imod = 1 : nlayer
        A_rho(i,imod) = Amin + rand*(Amax - Amin);
    end
    for imod =  1 : nlayer-1
        A_thk(i,imod) = Amin + rand*(Amax - Amin);
    end
end
%Inisialisasi Pulse rate
rmin = 0;
rmax = 1;
for i = 1 : npop
    for imod = 1 :  nlayer
        r_rho(i,imod) = rmin + rand*(rmax-rmin);
    end
    for  imod = 1 : nlayer-1
        r_thk(i,imod) = rmin + rand*(rmax-rmin);
    end
end
%inisialisasi model/solusi acak
rhomin = [1 1 1];
rhomax = [2000 2000 2000];
thkmin = [1 1];
thkmax = [2000 2000];
for i = 1 : npop
    rho(i , :) = rhomin + rand*(rhomax - rhomin);
    thick(i, :) = thkmin + rand*(thkmax - thkmin);
end
for ipop=1:npop
    [apparentResistivity, phase]=modelMT(rho(ipop,:),thick(ipop,:),T);
     app_mod(ipop,:)=apparentResistivity;
     phase_mod(ipop,:)=phase;
     
    [misfit]=misfitMT(app_sin,phase_sin,app_mod(ipop,:),phase_mod(ipop,:));
    E(ipop)=misfit;
end
%Cari global best
%Global best
idx = find(E ==min(E));
G_best_rho = rho(idx(1),:);
G_best_thick = thick(idx(1),:);
%proses inversi
for itr =  1 : niter
    for i = 1 :  npop
        %inisialisasi frekuensi
        for imod = 1 : nlayer
            f_rho(imod) = fmin + rand*(fmin-fmax);
        end
        for imod = 1 : nlayer-1
            f_thk(imod) = fmin + rand*(fmin-fmax);
        end
        %Update velocity and position
        for imod  = 1 : nlayer
            v_rho(1,imod) = v_rho(i,imod) + (rho(i,imod)-G_best_rho(imod))*f_rho(imod);
            rho_baru(1,imod) = rho(i,imod) + v_rho(1,imod);
            if rho_baru(1,imod) > rhomax(imod)
                rho_baru(1,imod) = rhomax(imod);
            end
            if rho_baru(1,imod) < rhomin(imod)
                rho_baru(1,imod) = rhomin(imod);
            end
        end
       for imod  = 1 : nlayer-1
            v_thk(1,imod) = v_thk(i,imod) + (thick(i,imod)-G_best_thick(imod))*f_thk(imod);
            thick_baru(1,imod) = thick(i,imod) +v_thk(1,imod);
            if thick_baru(1,imod) > thkmax(imod)
                thick_baru(1,imod) = thkmax(imod);
            end
            if thick_baru(1,imod) < thkmin(imod)
                thick_baru(1,imod) = thkmin(imod);
            end
       end
       random = rand;
       if random > r_rho(i) 
          for imod = 1 : nlayer
              rho_baru(1,imod) =G_best_rho(imod)+mean(A_rho(i,:))*(-1+2*rand);
          end
           if rho_baru(1,imod) > rhomax(imod)
                rho_baru(1,imod) = rhomax(imod);
            end
            if rho_baru(1,imod) < rhomin(imod)
                rho_baru(1,imod) = rhomin(imod);
            end
       end
       if random > r_thk(i)
           for imod = 1 : nlayer-1
               thick_baru(1,imod) = G_best_thick(imod) + mean(A_thk(i,:))*(-1+2*rand);
           end
             if thick_baru(1,imod) > thkmax(imod)
                thick_baru(1,imod) = thkmax(imod);
            end
            if thick_baru(1,imod) < thkmin(imod)
                thick_baru(1,imod) = thkmin(imod);
            end
       end
       %Menghitung model baru
        [apparentResistivity_baru, phase_baru]=modelMT(rho_baru,thick_baru,T);
        [E_baru] = misfitMT(app_sin,phase_sin,apparentResistivity_baru, phase_baru);
       if E_baru < E(i) && rand < A_rho(i) && rand < A_thk(i) 
            rho(i,:) = rho_baru(1,:);
            thick(i,:) = thick_baru(1,:);
            E(i) = E_baru;
            app_mod(i,:) = apparentResistivity_baru(1,:);
            phase_mod(i,:) = phase_baru(1,:);
       end
    end
    Emin = 1000;
    for i = 1 : npop
      if E(i)< Emin
         Emin = E(i);
         G_best_rho  = rho(i,:);
         G_best_thick  = thick(i,:);
         app_model = app_mod(i,:);
         phase_model = phase_mod(i,:); 
      end
    end
     A_rho(i) = A_rho(i)*alpha;
     A_thk(i) = A_thk(i)*alpha;
     r_rho(i) = r_rho(i)*(1-exp(gamma*itr));
     r_thk(i) = r_thk(i)*(1-exp(gamma*itr));
    Egen(itr)=Emin;
     %Persiapan Ploting
    rho_plot = [0 R];
    thk_plot = [0 cumsum(thk) max(thk)*10000];
    rhomod_plot = [0 G_best_rho];
    thkmod_plot = [0 cumsum(G_best_thick) max(G_best_thick)*10000];
    
end

    %Plotting
    figure(1)
    subplot(2, 2, 1)
    loglog(T,app_sin,'.b',T,app_model,'r','MarkerSize',12,'LineWidth',1.5);
    axis([10^-3 10^3 1 10^3]);
    legend({'Synthetic Data','Calculated Data'},'EdgeColor','none','Color','none','FontWeight','Bold','location','SouthEast');
    xlabel('Periods (s)','FontSize',12,'FontWeight','Bold');
    ylabel('App. Resistivity (Ohm.m)','FontSize',12,'FontWeight','Bold');
    title(['\bf \fontsize{10}\fontname{Times}Period (s) vs Apparent Resistivity (ohm.m)  || Misfit : ', num2str(Egen(itr)),' || iteration : ', num2str(itr)]);
    grid on
    
    subplot(2, 2, 3)
    loglog(T,phase_sin,'.b',T,phase_model,'r','MarkerSize',12,'LineWidth',1.5);
    axis([10^-3 10^3 0 90]);
    set(gca, 'YScale', 'linear');
    legend({'Synthetic Data','Calculated Data'},'EdgeColor','none','Color','none','FontWeight','Bold');
    xlabel('Periods (s)','FontSize',12,'FontWeight','Bold');
    ylabel('Phase (deg)','FontSize',12,'FontWeight','Bold');
    title(['\bf \fontsize{10}\fontname{Times}Period (s) vs Phase (deg)  || Misfit : ', num2str(Egen(itr)),' || iteration : ', num2str(itr)]);
    grid on
    
    subplot(2, 2, [2 4])
    stairs(rho_plot,thk_plot,'--b','Linewidth',3);
    hold on
    stairs(rhomod_plot ,thkmod_plot,'-r','Linewidth',2);
    hold off
     legend({'Synthetic Model','Calculated Model'},'EdgeColor','none','Color','none','FontWeight','Bold','Location','SouthWest');
    axis([1 10^4 0 5000]);
    xlabel('Resistivity (Ohm.m)','FontSize',12,'FontWeight','Bold');
    ylabel('Depth (m)','FontSize',12,'FontWeight','Bold');
    title(['\bf \fontsize{10}\fontname{Times}Model']);
    set(gca,'YDir','Reverse');
    set(gca, 'XScale', 'log');
    set(gcf, 'Position', get(0, 'Screensize'));
    grid on
    %plot misfit
    figure(2)
    plot(1:niter,Egen,'r','Linewidth',1.5)
    xlabel('Iteration Number','FontSize',10,'FontWeight','Bold');
    ylabel('misfit','FontSize',10,'FontWeight','Bold');
    title('\bf \fontsize{12} Grafik Misfit ');
    grid on
```

其中，子函数modelMT与misfitMT与[萤火虫算法](https://cocklebur0924.github.io/2022/04/10/MT1Dinv_Firefly/)一致。

#### BA MT1D Inversion 反演结果

<center><img src=/images/PictureInsert/BAT.png width=100% /></center>
<center>左:反演模型响应对比；右:反演结果对比</center>

<center><img src=/images/PictureInsert/BATmisfit.png width=60%/></center>
<center>BA反演过程中RMSE变化曲线</center>

### The end

@.@不方便打开Matlab？那就点击[Matlab在线编辑器(Open in MATLAB Online)](https://matlab.mathworks.com/)试一下吧。

### Reference
[1] Yang, XS. (2010). A New Metaheuristic Bat-Inspired Algorithm. In: González, J.R., Pelta, D.A., Cruz, C., Terrazas, G., Krasnogor, N. (eds) Nature Inspired Cooperative Strategies for Optimization (NICSO 2010). Studies in Computational Intelligence, vol 284. Springer, Berlin, Heidelberg. https://doi.org/10.1007/978-3-642-12538-6_6

[Click here to watch this paper directly P65-74](https://link.springer.com/content/pdf/10.1007/978-3-642-12538-6.pdf)

[2] [CSDN回答：群体智能优化算法之蝙蝠算法(Bat Algorithm，BA)](https://blog.csdn.net/hba646333407/article/details/103081669)

[3] [Mohammad Rheza Zamani的Github项目:1D-Magnetotelluric-Inversion之蝙蝠算法](https://github.com/rhezazz/1D-Magnetotelluric-Inversion/tree/main/MT%20BA)
