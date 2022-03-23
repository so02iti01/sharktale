---
title: FFT analyzer结果输出为figure
date: 2022-03-21 15:25:42
tags: SIMULINK
---

%% This script exports the FFT analysis data from the Powergui FFT Analysis Tool to the base workspace

% ======= Before running it, do remember to update FFT analyzer in Simulink=============

% -------Get initial default structure:

```
FFTDATA = power_fftscope(ScopeData);  % Variable name may change sometimes
```

% -------Modify FFTDATA structure to have desired options and settings

```
FFTDATA.cycles = 1;

FFTDATA.fundamental = 50; 

FFTDATA.maxFrequency = 20000;
```

% -------Update the data structure

```
FFTDATA = power_fftscope(FFTDATA);
```

% -------Plot FFT data in figure

```
power_fftscope(FFTDATA)
```

