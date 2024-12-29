
| 实验名称 | 系统的复频域分析                   |
| ---- | -------------------------- |
| 班级   | 信安一班                       |
| 组员   | 李凌秋 王兴 苏瑞                  |
| 学号   | 23270519 23270534 23270510 |
| 指导老师 | 杨彬                         |



第一个代码
```
% 清除所有变量
clear all;
% 定义分子和分母系数
b = [0, 1, -1]; % 分子多项式系数
a = [1, 3, 2]; % 分母多项式系数
% 计算零点和极点
zr = roots(b); % 计算分子多项式的零点
pr = roots(a); % 计算分母多项式的极点
% 绘制零点和极点分布（第一组）
subplot(2,1,1); % 创建第一个子图
plot(real(zr), imag(zr), 'go', real(pr), imag(pr), 'mx', 'markersize', 12, 'linewidth', 2); 
% 绘制零点为绿色圆圈，极点为红色叉号
grid; % 添加网格线
legend('零点', '极点'); % 添加图例
% 使用 zplane 函数绘制零极点图（第一组）
subplot(2,1,2); % 创建第二个子图
zplane(b, a); % 绘制零极点图
% 定义第二组分子和分母系数
figure; % 创建新图窗口
d = [2, 5, 9, 5, 3]; % 第二组分子多项式系数
c = [5, 45, 2, 1, 1]; % 第二组分母多项式系数
% 计算零点和极点（第二组）
zs = roots(d); % 计算第二组分子多项式的零点
ps = roots(c); % 计算第二组分母多项式的极点
% 绘制零点和极点分布（第二组）
subplot(2,1,1); % 创建第一个子图
plot(real(zs), imag(zs), 'go', real(ps), imag(ps), 'mx', 'markersize', 12, 'linewidth', 2); 
% 绘制零点为绿色圆圈，极点为红色叉号
grid; % 添加网格线
legend('零点', '极点'); % 添加图例
% 使用 zplane 函数绘制零极点图（第二组）
subplot(2,1,2); % 创建第二个子图
zplane(d, c); % 绘制零极点图
```
第二个代码
```
w = -4*pi : 8*pi/511 : 4*pi;
num = [2 5 9 5 3];
den = [5 45 2 1 1];
h = freqz(num, den, w);
figure;
% 绘制幅度谱
subplot(4, 1, 1);
plot(w/pi, abs(h));
grid on;
title('幅度谱');
xlabel('omega / \pi');
ylabel('振幅');
% 绘制相位谱
subplot(4, 1, 2);
plot(w/pi, angle(h));
grid on;
title('相位谱');
xlabel('omega / \pi');
ylabel('相位 (弧度)');
% 绘制实部
subplot(4, 1, 3);
plot(w/pi, real(h));
grid on;
title('实部');
xlabel('omega / \pi');
ylabel('实部');
% 绘制虚部
subplot(4, 1, 4);
plot(w/pi, imag(h));
grid on;
title('虚部');
xlabel('omega / \pi');
ylabel('虚部');
```


![[Pasted image 20241229104646.png]]


![[Pasted image 20241229104700.png]]
![[Pasted image 20241229104732.png]]
