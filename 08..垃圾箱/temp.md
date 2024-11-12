
| 实验名称 | 离散时间系统的时域分析                |
| ---- | -------------------------- |
| 班级   | 信安一班                       |
| 组员   | 李凌秋 王兴 苏瑞                  |
| 学号   | 23270519 23270534 23270510 |
| 指导老师 | 杨彬                         |

# 一、实验目的
1. 通过 matlab 仿真一些简单的离散时间系统，并研究它们的时域特性。
2. 掌握利用 matlab 工具箱求解 LTI 系统的单位冲激响应。
# 二、预习内容
1. 离散时间系统具有哪些时域特性？
2. 常系数差分方程描述的 LTI 系统的冲激响应、阶跃响应。运用 LTI 系统的时域特性
与差分方程系数关系分析系统时域特性。

四、实验内容
1、离散时间系统的时域分析
1.1 线性与非线性系统
我们首先分析了线性系统的输出特性。通过Matlab代码，我们生成了输入信号，并计算了新系统的输出、线性叠加输出以及差信号。代码如下：
```
clear all;
n = 0:40;
a = 2; 
b = -3;

% 输入信号
x1 = cos(2*pi*0.1*n);
x2 = sin(2*pi*0.4*n);
x = a*x1 + b*x2;

% 新系统的输出
y1_new = x1 + 3.2*[zeros(1,2) x1(1:end-2)];  % 对x1的输出
y2_new = x2 + 3.2*[zeros(1,2) x2(1:end-2)];  % 对x2的输出
y_new = x + 3.2*[zeros(1,2) x(1:end-2)];      % 对合成信号x的输出
yt_new = a*y1_new + b*y2_new;                 % 线性叠加输出

% 计算差值
d_new = y_new - yt_new;

% 绘图
figure;
subplot(3,1,1);
stem(n, y_new);
ylabel('振幅');
title('新系统输出 y[n]');

subplot(3,1,2);
stem(n, yt_new);
ylabel('振幅');
title('线性组合输出 yt[n]');

subplot(3,1,3);
stem(n, d_new);
ylabel('振幅');
title('差信号 d[n]');

```

![[Pasted image 20241112193622.png]]


1.2
时变与时不变系统
接下来，我们分析了时变与时不变系统的特性。通过Matlab代码，我们计算了原始信号的输出、延迟信号的输出以及差值。代码如下：
```
clear all;
n = 0:40;
a = 2; 
b = -3;
D = 10;  % 延迟
x = cos(2*pi*0.1*n);  % 输入信号
xd = [zeros(1, D) x]; % 延迟后的信号

% 系统参数
num = [2.24 2.49];  % 系统的分子系数
den = [1 -0.4];     % 系统的分母系数

% 计算输出信号
y = filter(num, den, x);    % 原始信号的输出
yd = filter(num, den, xd);  % 延迟信号的输出
d = y - yd(1+D:41+D);        % 差值计算

% 绘图
figure;
subplot(3,1,1);
stem(n, y);
ylabel('振幅');
title('输出 y[n]'); grid;

subplot(3,1,2);
stem(n, yd(1:41));
ylabel('振幅');
title(['由于延时输入 x[n-', num2str(D), '] 的输出']); grid;

subplot(3,1,3);
stem(n, d);
ylabel('振幅');
title('差信号'); grid;

```
![[Pasted image 20241112193651.png]]
2、线性时不变系统的单位冲激响应
最后，我们分析了线性时不变系统的单位冲激响应。通过Matlab代码，我们使用impz和filter函数计算了单位冲激响应，并比较了两者的结果。代码如下：
```
clear all;
N = 45;  % 产生前 45 个样本

% 新系统的分子和分母系数
num = [0.9 -0.45 0.35 0.002];   % x[n] 的系数
den = [1 0.71 -0.46 -0.62];     % y[n] 的系数

% 使用 impz 计算单位冲激响应
h_impz = impz(num, den, N);

% 使用 filter 计算单位冲激响应
% 创建单位冲激信号
delta = [1 zeros(1, N-1)];  % 单位冲激信号
h_filter = filter(num, den, delta);  % 计算单位冲激响应

% 计算两个结果的差值
d_diff = h_impz - h_filter';  % 注意 h_filter 需要转置以匹配维度

% 绘图
figure;
subplot(3,1,1);
stem(0:N-1, h_impz);
xlabel('时间序号'); 
ylabel('振幅');
title('impz 计算的冲激响应');
grid;

subplot(3,1,2);
stem(0:N-1, h_filter);
xlabel('时间序号'); 
ylabel('振幅');
title('filter 计算的冲激响应');
grid;

subplot(3,1,3);
stem(0:N-1, d_diff);
xlabel('时间序号'); 
ylabel('振幅');
title('差值 d[n] (impz - filter)');
grid;

```
![[Pasted image 20241112193553.png]]

通过实验结果，我们验证了impz和filter函数在计算单位冲激响应时的一致性，进一步加深了对LTI系统时域特性的理解。

# 结论
通过本次实验，我们成功地使用Matlab仿真了离散时间系统，并研究了它们的时域特性。我们验证了线性系统的叠加原理，观察了时变与时不变系统在输入信号延迟时的输出变化，并计算了线性时不变系统的单位冲激响应。这些实验加深了我们对离散时间系统时域特性的理解，并提高了我们使用Matlab工具箱进行系统分析的能力。