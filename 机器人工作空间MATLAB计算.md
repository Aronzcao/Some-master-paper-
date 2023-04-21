机器人工作空间MATLAB计算

```matlab
clc
clear all;
L1=Link([0       0           0           0          0     ],'modified'); 
L2=Link([0       0           0.180      -pi/2       0     ],'modified');
L3=Link([0       0           0.600       0          0     ],'modified');
L4=Link([0       0.630       0.130      -pi/2       0     ],'modified');
L5=Link([0       0           0           pi/2       0     ],'modified');
L6=Link([0       0           0          -pi/2       0     ],'modified');
r=SerialLink([L1,L2,L3,L4,L5,L6],'name','ROBOT');                                    %建立机器人模型
r.display();
theta=[0 35/180*pi 46/180*pi 0 0 0];                                                   %机器人初始各关节的角度
r.plot(theta);                                                           
teach(r);                                                                              %机器人示教
 
hold on;
N=10000;                                                           %随机次数
theta1=-165/180*pi+(165/180*pi+165/180*pi)*rand(N,1); %关节1限制
theta2=-90/180*pi+(155/180*pi+90/180*pi)*rand(N,1);   %关节2限制
theta3=-200/180*pi+(70/180*pi+200/180*pi)*rand(N,1);  %关节3限制
theta4=-170/180*pi+(170/180*pi+170/180*pi)*rand(N,1); %关节4限制
theta5=-135/180*pi+(135/180*pi+135/180*pi)*rand(N,1); %关节5限制
theta6=-360/180*pi+(360/180*pi+360/180*pi)*rand(N,1); %关节6限制
for n=1:1:N
    modmyt06=r.fkine([theta1(n),theta2(n),theta3(n),theta4(n),theta5(n),theta6(n)]);
    T=transl(modmyt06);
    plot3(T(1,1),T(2,1),T(3,1),'r.','MarkerSize',3);
end

```

