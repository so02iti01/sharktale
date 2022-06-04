---
title: MATLAB|FFT analyzer结果输出为figure
date: 2022-03-21 15:25:42
updated: 
tags: 
  - SIMULINK
  - MATLAB
  - Powergui
  - FFT
---

在SIMULINK中，Powergui自带的FFT Analysis Tool，运行快速，不易出错，但是不方便直接读取幅值数据。而且，有时候需要展示FFT柱状图。所以，我们希望将 Powergui 中FFT Analysis Tool 产生的FFT数据，保存到workspace，并且输出为figure。

 <!-- more -->

`注意：`在每次FFT之前，都需要update FFT Analysis Tool 的数据。

1. 初始化结构体：

```matlab
FFTDATA = power_fftscope(ScopeData);  % ScopeData为scope保存数据的变量名，不同的scope有不同的名字
```

2. 设置结构体的参数（下面3个参数，需要和FFT Analysis Tool的设置一样 ）

```matlab
FFTDATA.cycles = 1;     % 进行FFT的周期数

FFTDATA.fundamental = 50;   % 基频（Hz）

FFTDATA.maxFrequency = 20000;   % FFT分析的最大频率
```

3. 更新结构体数据（依据上一步的三个参数进行更新）

```matlab
FFTDATA = power_fftscope(FFTDATA);
```

4. 输出FFT的figure

```matlab
power_fftscope(FFTDATA)
```

