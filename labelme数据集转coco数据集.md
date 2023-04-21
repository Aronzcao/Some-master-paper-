# labelme数据集转coco数据集

## 1.从conda切换到数据集文件夹：

## ![image-20220925203952429](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220925203952429.png)![image-20220925204046133](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220925204046133.png)

## labelme转换文件下载地址

https://github.com/wkentaro/labelme/tree/main/examples/instance_segmentation

## 标签格式

![image-20220925204229429](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220925204229429.png)

## 转换指令

![image-20220925204350349](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220925204350349.png)

注意：输出文件夹可以自己生成，无需设定

```
python labelme2coco.py test test_coco --labels labels.txt
```



## 结果

![image-20220925204500237](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220925204500237.png)

![image-20220925204536078](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220925204536078.png)

![image-20220925204819073](C:\Users\Alon.Z\AppData\Roaming\Typora\typora-user-images\image-20220925204819073.png)

