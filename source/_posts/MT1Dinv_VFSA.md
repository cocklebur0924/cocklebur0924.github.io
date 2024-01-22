---
title: Very Fast Simulated Annealing(VFSA)大地电磁一维反演 | 非常快速模拟退火算法
date: 2022-04-10 23:33:00
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

本节为`非常快速模拟退火算法 MT1D Inversion matlab代码`

### 非常快速模拟退火算法(VFSA)简介

模拟退火算法(Simulated Annealing，SA)在实际使用时，为了提高算法效率，常采用改进的算法，以保证在有限的时间内实现。Ingber提出的非常快速模拟退火算法(Very Fast Simulated Annealing，简称VFSA)最为常用$^{[1][2]}$。

### VFSA 大地电磁一维反演matlab代码

```matlab
%Program pemodelan inversi kurva sounding MT 1-D dengan
%menggunakan algoritma Very Fast Simulated Annealing
%Mohammad Rheza Zamani
tic
clear all;
clc;
%Data sintetik
R = [100 10 1000];
thk = [500 1500];
freq = logspace(-3,3,50);
T = 1./freq;
[app_sin, phase_sin] = modelMT(R, thk ,T);

%Definisi ruang model 
nlayer = 3; %Jumlah lapisan 
nitr = 200; %Jumlah iterasi 
%Ruang pencarian diatur sebesar 5 kali dari model data  sintetik
%Batas bawah pencarian nilai resistivitas
LBR = [1 1 1];
%Batas atas pencarian nilai resistivitas
UBR = [200 20 2000];
%Batas bawah pencarian nilai ketebalan
LBT = [1 1];
%Batas atas pencarian nilai resistivitas
UBT = [1000 3000];
Temp = 1;
dec = 1;
%Membuat model awal acak
rho1(1 , :) = LBR + rand*(UBR - LBR);
thick1(1, :) = LBT + rand*(UBT - LBT);
%Menghitung misfit, apparent resistivity, dan phase model awal
[apparentResistivity1, phase1]=modelMT(rho1(1,:),thick1(1,:),T);
app_mod1(1,:)=apparentResistivity1;
phase_mod1(1,:)=phase1;
     
[misfit1]=misfitMT(app_sin,phase_sin,app_mod1(1,:),phase_mod1(1,:));
E1=misfit1;
for itr = 1 : nitr
    rho_int(1 , :) = LBR + rand*(UBR - LBR);
    thick_int(1, :) = LBT + rand*(UBT - LBT);
    ui = rand;
    yi = sign(ui-0.5)*Temp*((((1 + (1/Temp)))^abs(2*ui-1))-1);
    rho2(1 , :) = rho_int + yi*(UBR - LBR);
    thick2(1, :) = thick_int + yi*(UBT - LBT);
    [apparentResistivity2, phase2]=modelMT(rho2(1,:),thick2(1,:),T);
    app_mod2(1,:)=apparentResistivity2;
    phase_mod2(1,:)=phase2;
    [misfit2]=misfitMT(app_sin,phase_sin,app_mod2(1,:),phase_mod2(1,:));
    E2=misfit2;
    delta_E = E2 -E1;
    
    if delta_E < 0
        rho1 = rho2;
        thick1 = thick2;
         E1 = E2;
    else
        P = exp((-delta_E)/Temp);
        if  P >= rand
           rho1 = rho2;
           thick1 = thick2;
           E1 = E2;
        end
    end
    [apparentResistivity_new, phase_new]=modelMT(rho1(1,:),thick1(1,:),T);
    Egen(itr)=E1;
    Temp = Temp*exp(-dec*(itr)^(1/(2*nlayer)-1));
    Temperature(itr) = Temp;

    %Persiapan Ploting
    rho_plot = [0 R];
    thk_plot = [0 cumsum(thk) max(thk)*10000];
    rhomod_plot = [0 rho1];
    thkmod_plot = [0 cumsum(thick1) max(thick1)*10000];
    
end
toc
    %Plotting
    figure(1)
    subplot(2, 2, 1)
    loglog(T,app_sin,'.b',T,apparentResistivity_new,'r','MarkerSize',12,'LineWidth',1.5);
    axis([10^-3 10^3 1 10^3]);
    legend({'Synthetic Data','Calculated Data'},'EdgeColor','none','Color','none','FontWeight','Bold');
    xlabel('Periods (s)','FontSize',12,'FontWeight','Bold');
    ylabel('App. Resistivity (Ohm.m)','FontSize',12,'FontWeight','Bold');
    title(['\bf \fontsize{10}\fontname{Times}Period (s) vs Apparent Resistivity (ohm.m)  || Misfit : ', num2str(Egen(itr)),' || iteration : ', num2str(itr)]);
    grid on
    
    subplot(2, 2, 3)
    loglog(T,phase_sin,'.b',T,phase_new,'r','MarkerSize',12,'LineWidth',1.5);
    axis([10^-3 10^3 0 90]);
    set(gca, 'YScale', 'linear');
    legend({'Synthetic Data','Calculated Data'},'EdgeColor','none','Color','none','FontWeight','Bold');
    xlabel('Periods (s)','FontSize',12,'FontWeight','Bold');
    ylabel('Phase (deg)','FontSize',12,'FontWeight','Bold');
    title(['\bf \fontsize{10}\fontname{Times}Period (s) vs Phase (deg)  || Misfit : ', num2str(Egen(itr)),' || iteration : ', num2str(itr)]);
    grid on
    
    subplot(2, 2, [2 4])
    stairs(rho_plot,thk_plot,'--b','Linewidth',1.5);
    hold on
    stairs(rhomod_plot ,thkmod_plot,'-r','Linewidth',2);
    hold off
    legend({'Synthetic Model','Calculated Model'},'EdgeColor','none','Color','none','FontWeight','Bold','Location','SouthEast');
    axis([1 10^4 0 5000]);
    xlabel('Resistivity (Ohm.m)','FontSize',12,'FontWeight','Bold');
    ylabel('Depth (m)','FontSize',12,'FontWeight','Bold');
    title(['\bf \fontsize{10}\fontname{Times}Model']);
    subtitle(['\rho_{1} = ',num2str(rho1(1)),' || \rho_{2} = ',num2str(rho1(2)),' || \rho_{3} = ',num2str(rho1(3)),' || thick_{1} = ',num2str(thick1(1)),' || thick_{2} = ',num2str(thick1(2))],'FontWeight','bold')
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
    set(gcf, 'Position', get(0, 'Screensize'));
    grid on

    %Plot Temperature
    figure(3)
    plot(1:nitr,Temperature,'b','Linewidth',1.5)
    xlabel('Iteration Number','FontSize',10,'FontWeight','Bold');
    ylabel('Temperature','FontSize',10,'FontWeight','Bold');
    title('\bf \fontsize{12} Grafik Penurunan Temperature ');
    set(gcf, 'Position', get(0, 'Screensize'));
    grid on
```

其中，子函数modelMT与misfitMT与[萤火虫算法](https://cocklebur0924.github.io/2022/04/10/MT1Dinv_Firefly/)一致。

#### VFSA 反演结果

<center><img src=/images/PictureInsert/VFSA.png width=100% /></center>
<center>左:反演模型响应对比；右:反演结果对比</center>

<center><img src=/images/PictureInsert/VFSAmisfit.png width=60%/></center>
<center>VFSA反演过程中RMSE变化曲线</center>

<div style="display:none">![](/images/PictureInsert/VFSAtemperature.png)</div>

<center><img src=/images/PictureInsert/VFSAtemperature.png width=60%/></center>
<center>VFSA反演过程中Temperature变化曲线</center>

### The end

### Reference

[1] [Ingber L. Very fast simulated re-annealing. Mathematical and Computer Modelling, 1989, 12(8): 967-973.](https://pdf.sciencedirectassets.com/271552/1-s2.0-S0895717700X01550/1-s2.0-0895717789902021/main.pdf?X-Amz-Security-Token=IQoJb3JpZ2luX2VjELP%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJHMEUCIQCwiaWOuTEGWTbueRfUAXhK3vjkYWUYzsxC6bZy%2FaxIoAIgTqHcJxG0utaXg1iAwywV%2ByfKfN%2BshwBuaSWsQHKrY6oq%2BgMIXBAEGgwwNTkwMDM1NDY4NjUiDFeiaTbbmvHHi71DWSrXA2UvMaJjaxWIdJzKHHAa824mfgVS6M8DdyaPN0yBmdAe3Fq0aW4z3cKfw%2B36QnCpejJafcZOLi3gbQp7kY7lF7Jkx6I%2FuE9dsDnTV%2FIEnv9spO9h8rX7SveUosKGzDhz0QFUJKIZNBidm%2BVQRSWrSWFJTA6NJnCyDpTBK6DAisrwE4OyHqAVKKp6y3u22hfzNwsTg57Ei8EJAd1AsPj5NktFdDeRok2LArIZ8KCLdXkZAdRspJ2d6uPeNz5CijSpV53W%2BEMMzXPjGkI2SfA0wxefOF7SlSOJ703%2FAh82R%2FwrIhUBL8Ob2Oq7vByleUnWaZ3lvj8Zn6BBD%2BVkPTvq7l%2FgCH8ww9HtkB%2Fm9Mm0jb04eiWIjZsAidGz0qQWcJQlJbxYddGfmq2PMWJrafAOZXP72%2FvmyAmy28tVwWoS47z4VFJfKYt%2FN12BLpz4cKNcWLcI9sEEoIvUUJLwCgEqlQ5%2B%2F%2BVemTFal7QMs4XYt0WJdufu8VqJXRSQoFBt%2B9kRAvdmQeA%2FQJVJyIEeQffNUwFZxCR9jAzR6tjLhfnuBcsjT8zRVjlFXdLiPN149LqMaquvFovEXmWnL6rLJH6PGh%2BSU%2Bncaa8UZXw2%2BOIwMyo4R8gJR8I8ezCI0dqSBjqlAdlJt1pTWp%2Fo%2B83LAROcE6GpKj5Ci8TlT13GOEz7UHmKtnm87AWmyo3z0f1aWoSNKbEWh%2BPz79lY%2FYZ1kRsgvFJv%2BpoXMzNAV3iDoV1KbSM0J05YTLjOl2xXbQDNffJYOpBpeWKJAQZpvV4kcFbsFtPVEpn1Vqjxir51CU0sB14q%2FyUzFbRjzC%2F%2FFN3Mjz2%2FU7FNi8LCqPHD99AZ3ClE2b4nizb9tw%3D%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220413T115130Z&X-Amz-SignedHeaders=host&X-Amz-Expires=300&X-Amz-Credential=ASIAQ3PHCVTY56UZPUU5%2F20220413%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=fc25cba581822118e3acc05d61b99c25f4850291fb0b539e4d04411f2c9b076a&hash=9046fc64d2d9d8f800cecc71f3dd9e9acda2429414a947437e7ac3143ae5fc2a&host=68042c943591013ac2b2430a89b270f6af2c76d8dfd086a07176afe7c76c2c61&pii=0895717789902021&tid=spdf-3ea47c47-aeec-47ef-a2d6-b98122ee112a&sid=c1be4026364cb248754b0f84e6a4d4069107gxrqa&type=client)

[2] [陈华根,李丽华,许惠平,陈冰.改进的非常快速模拟退火算法. 同济大学学报(自然科学版),2006(08):1121-1125.](https://max.book118.com/html/2017/1009/136575186.shtm)

[3] [Mohammad Rheza Zamani的Github项目:1D-Magnetotelluric-Inversion之VFSA算法](https://github.com/rhezazz/1D-Magnetotelluric-Inversion/tree/main/MT%20VFSA)
