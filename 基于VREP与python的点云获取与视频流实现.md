# 基于VREP与python的点云获取与视频流实现

## √静态点云获取

```python

#从Vrep当中获取点云数据，首先导入必要库
from utils_vrep import vrep_init, vrep_getHandles
from utils_vrep import VisionSensorCfg
from utils_vrep import vrep_getPointCloudByList, vrep_getPoseByList
import open3d as o3d
import copy


#--------------------------仿真设置（句柄名称获取）--------------
print('::Program started')
clientId=vrep_init(19997)
gpJoint_name_list = ['gp_joint1', 'gp_joint2', 'gp_joint3', 'gp_joint4', 'gp_joint5', 'gp_joint6']
cuboid_name_list = ['GJ', 'GJsub']
endGripper_name_list = ['RG2_openCloseJoint']
camera_depth_name_list = ["camera_depth"]
camera_color_name_list = ["camera_color"]
#------------------------------句柄名称列表---------------------
obj_name_list = gpJoint_name_list + \
    cuboid_name_list + \
    endGripper_name_list + \
    camera_depth_name_list +\
    camera_color_name_list

handles =vrep_getHandles(clientId,obj_name_list)

# 配置视觉传感器的参数
camera_depth_cfg = VisionSensorCfg(
        w = 640, h = 480,
        theta_deg = 60,
        dn = 0.01, df = 0.5,
        depth_scale = 1000)

camera_depth_cfg_list = [camera_depth_cfg] * len(camera_depth_name_list)
#""" 获取点云并保存 """
# 获取视觉传感器采集的点云
pcd0_dict = vrep_getPointCloudByList(handles, camera_depth_name_list, camera_depth_cfg_list,
                                     if_viz=False)
# 获取视觉传感器的位姿
pose0_ObjToBase_dict = vrep_getPoseByList(handles, obj_name_list)

# 可视化
mesh_frame = o3d.geometry.TriangleMesh.create_coordinate_frame(size=100, origin=[0, 0, 0]) # 坐标系, 便于可视化
# 变换坐标系 & 滤除底部平面
pcd0_filter_dict = copy.deepcopy(pcd0_dict)

for obj_name in camera_depth_name_list:
    if obj_name != "camera_depth":
        continue

    pcd = pcd0_filter_dict[obj_name]

    # 把点云从 视觉传感器坐标系 转换到 世界坐标系
    pcd.transform(pose0_ObjToBase_dict[obj_name])

    # 去除平面
    aabb = o3d.geometry.AxisAlignedBoundingBox(min_bound=pcd.get_min_bound() + [0, 0, 5], max_bound=pcd.get_max_bound())
    pcd_crop = pcd.crop(aabb)  # 裁剪重建的三角面片

    pcd0_filter_dict[obj_name] = pcd_crop

# 相机坐标系下的点云
o3d.visualization.draw_geometries([pcd0_dict["camera_depth"], mesh_frame])
# 机械臂基坐标系下的点云
o3d.visualization.draw_geometries([pcd0_filter_dict["camera_depth"], mesh_frame])
# 保存点云
o3d.visualization.draw_geometries_with_editing([pcd0_filter_dict["camera_depth"]])


```

## √动态RGB视频流、动态depth视频流、动态PointCloud视频流

### dianyunhuoqu.py

```python
#从Vrep当中获取点云数据，首先导入必要库
from utils_vrep import vrep_init, vrep_getHandles
from utils_vrep import VisionSensorCfg
from utils_vrep import vrep_getPointCloudByList, vrep_getPoseByList
import open3d as o3d
import copy
from pcd_visualizer import PCDVisualizer
import cv2
from utils_vrep import vrep_getRGBimage
import sim as vrep
import time
import numpy as np
#--------------------------仿真设置（句柄名称获取）--------------
print('::Program started')
clientId=vrep_init(19997)                                                                         
gpJoint_name_list = ['gp_joint1', 'gp_joint2', 'gp_joint3', 'gp_joint4', 'gp_joint5', 'gp_joint6']
cuboid_name_list = ['GJ', 'GJsub']                                                             
endGripper_name_list = ['RG2_openCloseJoint']
camera_depth_name_list = ["camera_depth"]
camera_color_name_list = ["camera_color"]
#------------------------------句柄名称列表---------------------
obj_name_list = gpJoint_name_list + \
    cuboid_name_list + \
    endGripper_name_list + \
    camera_depth_name_list +\
    camera_color_name_list

handles =vrep_getHandles(clientId,obj_name_list)

# 配置视觉传感器的参数
camera_depth_cfg = VisionSensorCfg(
        w = 640, h = 480,
        theta_deg = 60,
        dn = 0.01, df = 0.5,
        depth_scale = 1000)

# 点云动态可视化窗口建立
pcd_visual = PCDVisualizer()
pcd_visual.create_window()
# 创建窗口RGB图和深度图
win_flag = cv2.WINDOW_NORMAL | cv2.WINDOW_KEEPRATIO | cv2.WINDOW_GUI_EXPANDED
cv2.namedWindow("color", flags=win_flag)
cv2.namedWindow("depth", flags=win_flag)

# 从vrep当中获取rgb相机的句柄
errorCode, visionSensorHandle = vrep.simxGetObjectHandle(clientId, 'camera_color', vrep.simx_opmode_oneshot_wait)
# 从vrep当中获取rgb图像
errprCode, resolution, image = vrep.simxGetVisionSensorImage(clientId, visionSensorHandle, 0,
                                                                 vrep.simx_opmode_streaming)
time.sleep(0.5)


#开始循环采集
while (True):
    #采集rgb图
    errprCode, resolution, image = vrep.simxGetVisionSensorImage(clientId, visionSensorHandle, 0,
                                                                 vrep.simx_opmode_buffer)
    
    color_img = np.asarray(image)
    color_img.shape = (resolution[1], resolution[0], 3)
    color_img = color_img.astype(np.uint8) #颜色转换
    color_img = np.fliplr(color_img)     
    da = cv2.flip(color_img, -1)         #反转图片



    camera_depth_cfg_list = [camera_depth_cfg] * len(camera_depth_name_list)
    pcd0_dict,depth = vrep_getPointCloudByList(handles, camera_depth_name_list, camera_depth_cfg_list,
                                     if_viz=False)

    # 获取视觉传感器的位姿
    pose0_ObjToBase_dict = vrep_getPoseByList(handles, obj_name_list)

    # 可视化
    mesh_frame = o3d.geometry.TriangleMesh.create_coordinate_frame(size=100, origin=[0, 0, 0]) # 坐标系, 便于可视化
    # 变换坐标系 & 滤除底部平面
    pcd0_filter_dict = copy.deepcopy(pcd0_dict)

    for obj_name in camera_depth_name_list:
        if obj_name != "camera_depth":
            continue

        pcd = pcd0_filter_dict[obj_name]

    pcd_visual.update_pcd(pcd0_dict["camera_depth"])
    # 可视化器迭代
    pcd_visual.step()
    cv2.imshow('color', da)
    # 更新窗口“canvas”中的图片
    cv2.imshow('depth', depth)
    # 等待按键事件发生 等待1ms
    key = cv2.waitKey(1)
    if key == ord('q'):
        break
```

# **需要注意的是，每一次运行python的时候，注意关闭控制台，否则会占用端口，下一次运行不起效果！！！**![image-20221004212220019](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20221004212220019.png)





![image-20221004220022541](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20221004220022541.png)

![image-20221004220044763](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20221004220044763.png)