---
title: Particle Swarm Optimization(PSO)大地电磁一维反演 | 粒子群优化算法
date: 2022-04-10 21:33:00
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

本节为`粒子群优化算法 MT1D Inversion matlab代码`

### 粒子群优化算法(PSO)简介

粒子群优化算法(Particle Swarm Optimization, PSO)，也称为粒子群算法，是通过模拟鸟群觅食行为而发展起来的一种基于群体协作的随机搜索算法。最早是由Eberhart博士和kennedy博士于1995年发明$^{[1][2]}$。

当鸟群在森林中随机搜索食物，它们想要找到食物量最多的位置。但是所有的鸟都不知道食物具体在哪个位置，只能感受到食物大概在哪个方向。每只鸟沿着自己判定的方向进行搜索，并在搜索的过程中记录自己曾经找到过食物且量最多的位置，同时所有的鸟都共享自己每一次发现食物的位置以及食物的量，这样鸟群就知道当前在哪个位置食物的量最多。在搜索的过程中每只鸟都会根据自己记忆中食物量最多的位置和当前鸟群记录的食物量最多的位置调整自己接下来搜索的方向。鸟群经过一段时间的搜索后就可以找到森林中哪个位置的食物量最多（全局最优解）$^{[2]}$。

### PSO 大地电磁一维反演matlab代码

```matlab
%Program pemodelan inversi kurva sounding MT 1-D dengan
%menggunakan algoritma Particle Swarm Optimization
%Mohammad Rheza Zamani
tic
clear all;
clc;
%Data sintetik
R = [500 100 1000];
thk = [500 1500];
freq = logspace(-3,3,50);
T = 1./freq;
[app_sin, phase_sin] = modelMT(R, thk ,T);

%Definisi ruang model 
npop = 100; %Jumlah dari model  
nlayer = 3; %Jumlah lapisan 
nitr = 1000; %Jumlah iterasi 
%Ruang pencarian diatur sebesar 5 kali dari model data  sintetik
%Batas bawah pencarian nilai resistivitas
LBR = [1 1 1];
%Batas atas pencarian nilai resistivitas
UBR = [2000 2000 2000];
%Batas bawah pencarian nilai ketebalan
LBT = [1 1];
%Batas atas pencarian nilai resistivitas
UBT = [2000 2000];
%Paramter inversi
wmax = 0.9;
wmin = 0.5;
c1=2.05;
c2 =2.05;
%Membuat model awal acak
for ipop = 1 : npop
    rho(ipop , :) = LBR + rand*(UBR - LBR);
    thick(ipop, :) = LBT + rand*(UBT - LBT);
end
%Hitung velocity awal dan posisi awal
for ipop = 1 : npop
    for imod =  1 : nlayer
        %v_rho(ipop,imod) = 0.5.*(min(rho(ipop,:)) + rand*(max(rho(ipop,:)) - min(rho(ipop,:))));
        v_rho(ipop,imod) = 0;
    end
    for imod = 1 : nlayer -1
        %v_thk(ipop,imod) = 0.5.*(min(thick(ipop,:))) + rand*(max(thick(ipop,:)) - min(thick(ipop,:)));
        v_thk(ipop,imod) = 0;
    end
end
for ipop=1:npop
    [apparentResistivity, phase_baru]=modelMT(rho(ipop,:),thick(ipop,:),T);
     app_mod(ipop,:)=apparentResistivity;
     phase_mod(ipop,:)=phase_baru;
     
    [misfit]=misfitMT(app_sin,phase_sin,app_mod(ipop,:),phase_mod(ipop,:));
    E(ipop)=misfit;
end
%Global best
idx = find(E ==min(E));
G_best_rho = rho(idx(1),:);
G_best_thick = thick(idx(1),:);
%Inversi
for itr = 1 : nitr
    w = wmax-((wmax-wmin)/nitr)*itr;
    for i = 1 : npop
        P_best_rho = rho;
        P_best_thick = thick;
        %Membuat komponen kecepatan
        %Rho
        for imod = 1 : nlayer
            v_rho(1,imod) = w.*v_rho(i,imod) + c1.*rand.*(P_best_rho(i,imod) - rho(i,imod))+ c2.*rand.*(G_best_rho(imod) - rho(i,imod));
            rho_baru(1,imod) = rho(i,imod)+ v_rho(1,imod);
        if rho_baru(1,imod)<LBR(imod)
            rho_baru(1,imod) = LBR(imod);
        end
        if rho_baru(1,imod)>UBR(imod)
            rho_baru(1,imod) = UBR(imod);
        end
        end
        %Ketebalan
        for imod = 1 : (nlayer-1)
            v_thk(1,imod) = w.*v_thk(i,imod) + c1.*rand.*(P_best_thick(i,imod) - thick(i,imod))+ c2.*rand.*(G_best_thick(imod) - thick(i,imod));
            thick_baru(1,imod) = thick(i,imod)+ v_thk(1,imod);
          if thick_baru(1,imod)<LBT(imod)
              thick_baru(1,imod) = LBT(imod);
          end
          if thick_baru(1,imod)>UBT(imod)
              thick_baru(1,imod) = UBT(imod);
          end
        end
        %Update pesonal best
        [apparentResistivity_baru, phase_baru]=modelMT(rho_baru,thick_baru,T);
        app_mod_baru = apparentResistivity_baru;
        phase_mod_baru(i,:) = phase_baru;
        [E_baru] = misfitMT(app_sin,phase_sin,app_mod_baru, phase_mod_baru);
        if E_baru<E(i)
            rho(i,:) = rho_baru(1,:);
            thick(i,:) = thick_baru(1,:);
            app_mod(i,:) = app_mod_baru;
            phase_mod(i,:) = phase_mod_baru(1,:);
            E(i) = E_baru;
        end
    end
    Emin = 100;
     for ipop = 1 : npop
        if E(ipop)< Emin
            Emin = E(ipop);
            G_best_rho  = P_best_rho(ipop,:);
            G_best_thick  = P_best_thick(ipop,:);
            app_model= app_mod(ipop,:);
            phase_model = phase_mod(ipop,:);
        end
    end
    Egen(itr)=Emin;
end
toc
    %Persiapan Ploting
    rho_plot = [0 R];
    thk_plot = [0 cumsum(thk) max(thk)*10000];
    rhomod_plot = [0 G_best_rho];
    thkmod_plot = [0 cumsum(G_best_thick ) max(G_best_thick )*10000];
    %Plotting
    figure(1)
    subplot(2, 2, 1)
    loglog(T,app_sin,'.b',T,app_model,'r','MarkerSize',12,'LineWidth',1.5);
    axis([10^-3 10^3 1 10^4]);
    legend({'Synthetic Data','Calculated Data'},'EdgeColor','none','Color','none','FontWeight','Bold','location','southeast');
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
    subtitle(['\rho_{1} = ',num2str(G_best_rho(1)),' || \rho_{2} = ',num2str(G_best_rho(2)),' || \rho_{3} = ',num2str(G_best_rho(3)),' || thick_{1} = ',num2str(G_best_thick(1)),' || thick_{2} = ',num2str(G_best_thick(2))],'FontWeight','bold')
    set(gca,'YDir','Reverse');
    set(gca, 'XScale', 'log');
    set(gcf, 'Position', get(0, 'Screensize'));
    grid on

    %plot misfit
    figure(2)
    plot(1:nitr,Egen,'r','Linewidth',1.5)
    xlabel('Iteration Number','FontSize',10,'FontWeight','Bold');
    ylabel('RSME','FontSize',10,'FontWeight','Bold');
    title('\bf \fontsize{12} Grafik Misfit ');
    grid on
```
其中，子函数modelMT与misfitMT与[萤火虫算法](https://cocklebur0924.github.io/2022/04/10/MT1Dinv_Firefly/)一致。

#### PSO 反演结果

第一组反演结果：

<center><img src=/images/PictureInsert/PSO1.png width=100% /></center>
<center>左:反演模型响应对比；右:反演结果对比</center>

<center><img src=/images/PictureInsert/PSO1misfit.png width=60%/></center>
<center>PSO反演过程中RMSE变化曲线</center>

第二组反演结果：

<center><img src=/images/PictureInsert/PSO2.png width=100% /></center>
<center>左:反演模型响应对比；右:反演结果对比</center>

<center><img src=/images/PictureInsert/PSO2misfit.png width=60%/></center>
<center>PSO反演过程中RMSE变化曲线</center>

第三组反演结果：

<center><img src=/images/PictureInsert/PSO3.png width=100% /></center>
<center>左:反演模型响应对比；右:反演结果对比</center>

<center><img src=/images/PictureInsert/PSO3misfit.png width=60%/></center>
<center>PSO反演过程中RMSE变化曲线</center>


### The end

### Reference

[1] [Kennedy J. Particle swarm optimization. Proc. of 1995 IEEE Int. Conf. Neural Networks, (Perth, Australia), Nov. 27-Dec.  2011, 4(8):1942-1948 vol.4.](https://www.researchgate.net/profile/Goncalo-Pereira-8/publication/228518470_Particle_Swarm_Optimization/links/00b4953bbbe28b3238000000/Particle-Swarm-Optimization.pdf)

[2] [知乎回答：粒子群优化算法(Particle Swarm Optimization, PSO)的详细解读](https://zhuanlan.zhihu.com/p/346355572)

[3] [Mohammad Rheza Zamani的Github项目:1D-Magnetotelluric-Inversion之粒子群算法](https://github.com/rhezazz/1D-Magnetotelluric-Inversion/tree/main/MT%20PSO)
