# 如何在simulink当中实现**阶跃型0，1控制量**

## 1.背景介绍

## 对于这样一个离散型的数据量【1，2，3，4，5，6，1，2，3】，若大于5，则输出瞬时控制量1，并且此后小于5的时候，控制量依旧是1.

## switch:开关，开关的作用相当于if   else , 所以不能够保持控制量一直不变；

## 给定如下所示方波信号：

![image-20221019190801689](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20221019190801689.png)

## switch的结果

![image-20221019190914360](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20221019190914360.png)

![image-20221019190947211](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20221019190947211.png)

# simulink : memory模块：输入t时刻的值，输出t-1时刻的值，可设置t0

# 先1后0

# 因为1*1=1，0乘任何数为0



# 一旦有0，出现一次，后面全为0.

# 效果：

![image-20221019191353401](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20221019191353401.png)

