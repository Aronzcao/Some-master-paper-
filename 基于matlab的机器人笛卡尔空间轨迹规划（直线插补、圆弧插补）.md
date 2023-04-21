# 基于matlab的机器人笛卡尔空间轨迹规划（直线插补、圆弧插补）

## 直线插补

```matlab
clc;clear
%%轨迹规划的第一步是建立机器人模型，我们建立一个6R机器人
%定义机器人的D-H参数
%  a--连杆长度；alpha--连杆扭角；d--连杆偏距；theta--关节转角

a2=0.1563;a3=0.9251; a4=0.2883;a5=0.6208
d2=0.4803;d3=1.5301; d4=-1.3499;d5=0.0890;d6=0;d1=0
%            thetai     di      ai-1     alphai-1
L1 = Link([0            d1       0          0], 'modified');
L2 = Link('revolute', 'd', 0, 'a', a2, 'alpha', -pi/2, 'offset', -pi/2, 'modified');
L3 = Link([0            0        a3          0], 'modified');
L4 = Link([0            d4       a4   -pi/2], 'modified');
L5 = Link([0            0         0     pi/2], 'modified');
L6 = Link([0            d6        0      pi/2], 'modified');
%连接连杆，为机器人命名
robot=SerialLink([L1 L2 L3 L4 L5 L6], 'name', 'GP25');     %另一种命名方式robot.name = 'Tuesday'
robot.display();
%robot.plot([pi/2 -pi/2 0 0 0 0]);
robot.teach();
T1=transl(-1.194,0,1.213);%根据给定起始点，得到起始点位姿
T2=transl(-1.194,1.5,1.213);%根据给定终止点，得到终止点位姿
T=ctraj(T1,T2,50);
Tj=transl(T);
plot3(Tj(:,1),Tj(:,2),Tj(:,3),'LineWidth',2);%输出末端轨迹
grid on;
q=robot.ikine(T)
hold on
robot.plot(q);%动画演示
```

## 圆弧插补

```matlab
clc;clear
a2=0.1563;a3=0.9251; a4=0.2883;a5=0.6208
d2=0.4803;d3=1.5301; d4=-1.3499;d5=0.0890;d6=0;d1=0
%            thetai     di      ai-1     alphai-1
L1 = Link([0            d1       0          0], 'modified');
L2 = Link('revolute', 'd', 0, 'a', a2, 'alpha', -pi/2, 'offset', -pi/2, 'modified');
L3 = Link([0            0        a3          0], 'modified');
L4 = Link([0            d4       a4   -pi/2], 'modified');
L5 = Link([0            0         0     pi/2], 'modified');
L6 = Link([0            d6        0      pi/2], 'modified');
%连接连杆，为机器人命名
robot=SerialLink([L1 L2 L3 L4 L5 L6], 'name', 'GP25');     %另一种命名方式robot.name = 'Tuesday'
%确定画图平面
p0=[0,0,0];       
p1=[1,1,1];
p2=[-0.5,0.5,0.5];
X=p1-p0;B=p2-p0;  
Z=cross(X,B);
Y=cross(Z,X);   
x1=X/norm(X);
y1=Y/norm(Y);
z1=Z/norm(Z); 
R=[x1.',y1.',z1.'];
Js=[R ,p0.'];
Js=[Js;0 0 0 1];
r=norm(p1-p0);  

%圆的轨迹
t=0:0.05:5;
[s,sd,~] = tpoly(0,2*pi,t);       
q1=[-12.7,98.9,-107,158,-114,11]; 
k = q1;
vs =[]; % qs  关节速度
pd=[];%ps  笛卡尔坐标系下的坐标位置
for i = 1:length(s)
    T=[1 0 0 r*cos(s(i));0 1 0 r*sin(s(i));0 0 1 0;0 0 0 1];
    T1=Js*T;       
    q=robot.ikunc(T1,k); %求解逆运动
    k=q;
    pd=[pd;T1(1:3,4).'];
    vs=[vs;q];
end
close all;
figure('NumberTitle', 'off', 'Name', '运动图像');
plot2(pd,'LineWidth',2);%圆的线宽
hold on;
robot.plot(vs,'fps',20);%输出运动图像
%绘制运动曲线
figure('NumberTitle', 'off', 'Name', '关节值随时间变化曲线');
    subplot(4,1,1);%图像布局
    plot(t,pd);  %输出空间位置曲线
    title('空间位置曲线')
    xlabel('秒(s)');
    ylabel('路程(m)')
    
    subplot(4,1,2);   
    plot(t,vs);%输出关节角度曲线
    title('关节角度曲线')  
    xlabel('秒(s)');  
    ylabel('角度(rad)');
    legend('θ1','θ2','θ3','θ4','θ5','θ6');
    
    %计算空间速度并输出空间速度曲线
    vx = -r*sin(s).*sd;
    vy = r*cos(s).*sd;
    vv = r*sd;
    subplot(4,1,3);   
    plot(t,[vx,vy,vv]);
    title('空间速度曲线')
    xlabel('秒(s)');   
    ylabel('速度(m/s)')
    
    %计算关节速度
    omiga=[];
    for i =1:length(s)
        J0=ZU3.jacob0(vs(i,:)');
        dq=J0\[vx(i);vy(i);0;0;0;0];
        omiga=[omiga;dq'];
    end
    subplot(4,1,4);  
    plot(t,rad2deg(omiga));
    title('关节速度曲线')%输出关节速度曲线
    xlabel('秒(s)');  
    ylabel('角速度(rad/s)')


```

