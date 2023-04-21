# MASK-RCNN从数据集制作到训练再到部署全流程回顾

# 1.数据集的收集与制作

## 1.1采集照片

```
D:\BaiduNetdiskDownload\机械臂视觉抓取从理论到实践-工程源码-阿凯爱玩机器人-2022-09-01\机械臂视觉抓取从理论到实践-工程源码-阿凯爱玩机器人-2022-09-01\7.MaskRCNN实例分割\02.MaskRCNN模型训练与部署\示例代码\MaskRCNN实例分割\src
save_capture_img.py
此文件就是启用电脑摄像头采集照片，按'S'保存图片到指定位置，具体看代码
```



```python
'''
摄像头读取图像并展示
'''
import numpy as np # 引入numpy 用于矩阵运算
import cv2 # 引入opencv库函数

# 摄像头ID
cam_id = 0
# 创建一个video capture的实例
cap = cv2.VideoCapture(cam_id)

# 查看Video Capture是否已经打开
print("摄像头是否已经打开 ？ {}".format(cap.isOpened()))

## 设置画面的尺寸
# 画面宽度设定为 1920
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1920)
# 画面高度度设定为 1080
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 1080)

## 创建一个名字叫做 “image_win” 的窗口
cv2.namedWindow('image_win',flags=cv2.WINDOW_NORMAL | cv2.WINDOW_KEEPRATIO | cv2.WINDOW_GUI_EXPANDED)

# 图像计数
count = 1

while(True):
    # 获取摄像头的画面
    ret, frame = cap.read()
    if not ret:
        # 如果图片没有读取成功
        print("图像获取失败，请按照说明进行问题排查")
        break
    # 更新窗口“image_win”中的图片
    cv2.imshow('image_win',frame)
    # 等待按键事件发生 等待1ms
    key = cv2.waitKey(1)
    if key == ord('q'):
        # 如果按键为q 代表quit 退出程序
        print("程序正常退出")
        break
    elif key == ord('s'):
        # 设置图像保存路径
        img_save_path = 'data/image_raw/{}.png'.format(count)
        # 保存图像
        cv2.imwrite(img_save_path, frame)
        # 打印日志
        print("图像保存 ： {}".format(img_save_path))
        count += 1
# 释放VideoCapture
cap.release()
# 销毁所有的窗口
cv2.destroyAllWindows()
```

## 1.2照片预处理

```
此部分主要包含：照片重命名、裁剪等
这些一般而言都在采集的时候满足要求，如果需要进行批量重命名
网上一大堆，找来就能用
```

## 1.3给照片打标签-基于labelme

```
此部分就是耐心，具体步骤参见《机械臂视觉抓取从理论到实践》
```

## 1.4数据集划分

```
执行脚本
D:\BaiduNetdiskDownload\机械臂视觉抓取从理论到实践-工程源码-阿凯爱玩机器人-2022-09-01\完整数据集\part01
划分数据集与测试集.ipynb
注意整体数据集的文件夹名字以及图片格式进行修改即可
```



## 1.5labelme转换成coco

![image-20220927213743862](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220927213743862.png)

```
由于Detectron2需要训练的数据集为coco
所以需要进行格式转换
转换脚本路径：
D:\BaiduNetdiskDownload\机械臂视觉抓取从理论到实践-工程源码-阿凯爱玩机器人-2022-09-01\完整数据集\part01
labelme2coco.py

只需要在数据集的上一级文件目录之下，执行：
训练集格式转换:
 python labelme2coco.py train --labels=labels.txt
测试集格式转换:
 python labelme2coco.py test --labels=labels.txt
 
 即可
```

# 2.训练数据集

```
D:\BaiduNetdiskDownload\机械臂视觉抓取从理论到实践-工程源码-阿凯爱玩机器人-2022-09-01\机械臂视觉抓取从理论到实践-工程源码-阿凯爱玩机器人-2022-09-01\7.MaskRCNN实例分割\02.MaskRCNN模型训练与部署\示例代码\MaskRCNN实例分割\src
07.图像实例分割实验(使用maskrcnn工具类).ipynb
jupyter notebook运行时需要注意的几点
注意数据集的绝对路径
注意test,train已经生成coco
注意文件夹的位置：
-part1
   -allimage
   -test(含coco.json)
   -train(含coco.json)
   -label.txt
   -labelme2coco.py
   -划分数据集与测试集.ipynb

```

## MASK-RCNN 参数设置经验

```
#数据加载的线程数必须设定为0
cfg.DATALOADER.NUM_WORKERS=0
#每个BATCH的图像数必须设定为1
cfg.SOLVER.IMS_PER_BATCH=1

```

## 训练结束之后，会在part1之下生成output,权重文件就藏在里面

```
output/model_final.pth
```

# 3.模型推理（就是验证模型）

## 3.1基于图像

```
就是在jupyter notebook脚本里，注释掉
# 读取测试集的dataset dict
#part01_test = detectron2.data.get_detection_dataset_dicts("part01_test")

# 获取第一个图像的标注数据
#anno_dict = part01_test[10]
# 读取图片
#img_path = anno_dict["file_name"] # 获取图像路径


然后修改图像输入的名称、路径
继续运行就行
```

## 3.2基于视频

```python
'''
MaskRCNN实例分割
从USB摄像头动态读取
'''
import os
import json
import numpy as np
# from matplotlib import pyplot as plt
import cv2
# from sympy import capture
import torch
import sys
# ======Detectron2相关的库========
import detectron2
from detectron2.config import get_cfg
# 数据库信息
from detectron2.data import MetadataCatalog, DatasetCatalog
# 预测器
from detectron2.engine.defaults import DefaultPredictor
# 可视化器
from detectron2.utils.visualizer import ColorMode, Visualizer
# 数据集
from detectron2.data.datasets import register_coco_instances
# 预训练模型
from detectron2 import model_zoo
import time
# 数据集的绝对路径
DATASET_PATH = "D:/Aobizhong/part01"
# 验证集图片路径
test_image_dir = os.path.join(DATASET_PATH, "test")
# 验证集的标注文件路径
test_annotation_path = os.path.join("test_image_dir", "coco.json")
# 注册Detectron2数据集，命名为 part01_test
register_coco_instances("part01_test", {}, test_annotation_path, test_image_dir)
part01_test_metadata = MetadataCatalog.get("part01_test")

# 获取默认配置
cfg = get_cfg()
# 配置图像分割Mask RCNN的配置
cfg.merge_from_file(model_zoo.get_config_file("COCO-InstanceSegmentation/mask_rcnn_R_50_FPN_3x.yaml"))
# 载入训练好的模型权重
cfg.MODEL.WEIGHTS = os.path.join("output", "model_final.pth")
# 配置置信度阈值
cfg.MODEL.ROI_HEADS.SCORE_THRESH_TEST = 0.5
# 输入图像尺寸
cfg.MODEL.ROI_HEADS.BATCH_SIZE_PER_IMAGE = 512
# 对于当前数据集，类型就只有一个。
cfg.MODEL.ROI_HEADS.NUM_CLASSES = 1

# 创建预测器
predictor = DefaultPredictor(cfg)

# 创建VideoCapture
sys.path.append("./astra-open3d-python/")
from astra import Astra
# 配置项
# - 移除图像畸变
rm_distortion = True

# 创建相机对象
# config_path要写的是Astra 3D相机的配置文件夹路径
camera = Astra(config_path="./astra-open3d-python/config/")

try:
	# 初始相机视频流
	camera.init_video_stream(video_mode="color")
except Exception as e:
	print("[错误] 没有发现Astra设备, 请检查接线或驱动")
	exit(-1)

if rm_distortion:
	# 读取相机参数(相机内参)
	camera.load_cam_calib_data()

# 创建窗口
win_flag = cv2.WINDOW_NORMAL | cv2.WINDOW_KEEPRATIO | cv2.WINDOW_GUI_EXPANDED
cv2.namedWindow("color", flags=win_flag)


while (True):
    st=time.time()
    count = 0
    count+=1
    # 采集彩图, 色彩空间BGR
    img_bgr = camera.read_color_img()
    # 去除彩图畸变
    if rm_distortion:
        img_bgr = camera.rgb_camera.remove_distortion(img_bgr)
    img_rgb = img_bgr[:, :, ::-1]  # BGR颜色空间转换为RGB颜色空间
    # 模型预测
    predictions = predictor(img_rgb)
    # 创建可视化器
    visualizer = Visualizer(img_rgb, part01_test_metadata, instance_mode=ColorMode.IMAGE)
    # 实例分割的预测结果从GPU移动至CPU设备
    instances = predictions["instances"].to("cpu")
    # 绘制预测结果
    vis_output = visualizer.draw_instance_predictions(predictions=instances)
    # 获取预测结果的图片 获取的是RGB颜色空间的图片
    img_rgb_canvas = vis_output.get_image()
    img_bgr_canvas = img_rgb_canvas[:, :, ::-1]
    # 更新窗口“image”中的图片
    cv2.imshow('image', img_bgr)
    # 更新窗口“canvas”中的图片
    cv2.imshow('canvas', img_bgr_canvas)
    en=time.time()
    fps=count/(en-st)
    print(fps)

    # 等待按键事件发生 等待1ms
    key = cv2.waitKey(1)
    if key == ord('q'):
        # 如果按键为q 代表quit 退出程序
        break

# 释放VideoCapture
camera.release()
# 销毁所有的窗口
cv2.destroyAllWindows()

```

## 此为自己开发的基于Astra相机的实时分割检测系统  astra_mask_rcnn.py

## 注意以下几点：

## ·目录结构

```
-part1
   -astra-open3d-python
   -allimage
   -test(含coco.json)
   -train(含coco.json)
   -output(含model_final.pth)
   -label.txt
   -labelme2coco.py
   -划分数据集与测试集.ipynb
   -astra_mask_rcnn.py

```

# 4.模型测试（1500次训练以上）

```
jupyter notebook最后一部分
注意以下语句
# 创建模型评估器
evaluator = COCOEvaluator("part01_test",cfg, False, output_dir="./output")
```

