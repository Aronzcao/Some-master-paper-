# Astra相机相关介绍

> ## ORBBEC Astra系列3D传感摄像头采用单目结构光技术，**具有高精度、低功耗、响应迅速、稳定可靠的优点**，开发人员可以通过选择不同的版本自由地在短距离、长距离和高分辨率RGB摄像机之间切换，以满足个性化需求。其能够覆盖近距离和中远距离的多种室内场景，是奥比中光最经典和最畅销的3D深度相机系列之一。该系列包括Astra Pro Plus、Astra和Astra S。
>
> 官网介绍：[Astra系列 · 3D视觉开发者社区 · 奥比中光 (orbbec.com.cn)](https://developer.orbbec.com.cn/product_details.html?id=4)

## 硬件结构图和特点

### ![image-20220923135134885](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220923135134885.png)

![image-20220923135246957](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220923135246957.png)

![image-20220923135300652](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220923135300652.png)

## 规格参数

<img src="C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220923135517170.png" alt="image-20220923135517170" style="zoom:150%;" />

| **术语**        | **描述**                                                     |
| --------------- | ------------------------------------------------------------ |
| Baseline        | 单目结构光：红外相机成像中心与红外投影仪投影中心之间的距离   |
| Depth           | 深度视频流就像彩色视频流一样，每个像素都有一个值代表距离摄像机的距离而不是颜色信息 |
| FOV             | 视场角，用于描述相机成像给定场景的角度范围，主要有水平视场角(H-FOV)、垂直视场角(V-FOV)和对角线视场角(D-FOV)三种 |
| Depth processor | 深度计算处理器, 用于实现深度计算算法并输出深度图像的专用ASIC 芯片，如MX400、MX6000 |
| IR camera       | 红外相机，或红外摄像头                                       |
| RGB camera      | 彩色相机，或彩色摄像头                                       |
| LDM             | 红外投影仪(IR projector)，也称红外激光投影仪、结构光投影仪等，用于发射结构光图案 |
| VCSEL           | 垂直腔面激光发射器                                           |
| Depth camera    | 深度相机, 包含深度成像模组以及彩色成像模组，其中深度成像模组一般由红外投影仪、红外相机以及深度计算处理器组成，彩色成像模组一般指彩色相机 |
| I2C             | I2C总线是由Philips公司开发的一种简单、双向二线制同步串行总线。它只需要两根线即可在连接于总线上的器件之间传送信息 |
| ISP             | 图像信号处理器，用于对图像进行后处理                         |
| Lens            | 透镜组，在红外相机、彩色相机中用于成像，在激光投影仪中用于投影 |
| MIPI            | MIPI联盟，即移动产业处理器接口（Mobile Industry Processor Interface 简称MIPI）联盟。MIPI（移动产业处理器接口）是MIPI联盟发起的为移动应用处理器制定的开放标准和一个规范 |
| SoC             | System on Chip的缩写，称为芯片级系统，也称片上系统，意指它是一个产品，是一个有专用目标的集成电路，其中包含完整系统并有嵌入软件的全部内容 |
| ASIC            | ASIC被认为是一种为专门目的而设计的集成电路。是指应特定用户要求和特定电子系统的需要而设计、制造的集成电路。ASIC的特点是面向特定用户的需求，ASIC在批量生产时与通用集成电路相比具有体积更小、功耗更低、可靠性提高、性能提高、保密性增强、成本降低等优点. 在本文中主要指MX400，MX6000 |
| PCBA            | 基础电路板，用来承载深度计算处理器、存储器等关键电子器件     |
| TBD             | 待定，信息将在后期修订中提供                                 |

![image-20220923140026459](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220923140026459.png)-

![image-20220923140100406](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220923140100406.png)

![image-20220923140141934](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220923140141934.png)![image-20220923140151638](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220923140151638.png)

![image-20220923140205310](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220923140205310.png)