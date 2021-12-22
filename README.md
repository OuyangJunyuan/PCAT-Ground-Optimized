# 新增: 地面校准

为原来的PCAT增加了如下功能,已经提交PR: 

* 矫正地面按钮: 点击后(清空当前标注)并重新初始化, 同时检测地面出.根据检测出的地面拟合平面.根据拟合平面矫正点云使得地平面水平, 同时平移点云使得地面中心保持在地面一定高度上(对3D检测保持z轴高度非常重要). 当按钮激活期间, 后续的其他帧点云会自动在加载时就完成矫正.
* 参数设置栏目: 当矫正按钮激活时, 在输入栏回车可以更新参数并(清空当前标注)重新初始化本次点云然后检测地面.
* 显示栏:增加地面显示按钮.当矫正地面和显示点云功能激活时显示地面点云.

![image-20211222193054803](https://s2.loli.net/2021/12/22/7Ihn9HVsoS6OBUJ.png)

效果如图,保持点云水平并距离地面1.7m(KITTI). 下图为显示地面状态.

![image-20211222194054215](https://s2.loli.net/2021/12/22/lfGEMvFR2orUmTO.png)

---

PCAT点云标注工具-使用手册
---------------------

- Demo项目，请自行魔改

- This is the open source version:

     `Author:  WenwenDu`
     `TEL:     18355180339`
     `E-mail:  1455112695@qq.com`

- Video tutorial: 
1. `https://v.youku.com/v_show/id_XNDYxNjY4MDExMg==.html?spm=a2h0k.11417342.soresults.dtitle`

2. `https://v.youku.com/v_show/id_XNDYxNjY4MDI5Mg==.html?spm=a2hzp.8244740.0.0`

## **I. 配置使用环境及安装**

- `配置要求：ubuntu16.04 + ROS Kinetic full`
- `注意：请务必保证系统使用原生python2.7，在使用Anaconda2的情况下，请在~/.bashrc环境变量中临时关闭Anaconda2,避免冲突。（如果你长期使用ROS，强烈建议在虚拟环境下使用anaconda，避免冲突。）`

### 1. 安装ROS-Kinetic
参考[ROS WiKi-安装说明](http://http://wiki.ros.org/kinetic/Installation/Ubuntu), 安装步骤如下：
```
添加ROS源：
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
添加ROS源秘钥：
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 421C365BD9FF1F717815A3895523BAEEB01FA116
更新源
sudo apt-get update
```
```
安装ROS完整版：（由于使用Rviz，PCL等模块，请务必安装完整版）
sudo apt-get install ros-kinetic-desktop-full
sudo apt-cache search ros-kinetic
初始化ROS：
sudo rosdep init
rosdep update
```

```
添加环境变量
echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
更新ROS环境变量
source /opt/ros/kinetic/setup.bash
```

```
测试ROS是否成功安装:
开启一个新的Teminnal，输入：
roscore
测试Rviz
开启一个新的Teminnal，输入：
rviz
```
成功显示rviz界面如下：
![图片](https://github.com/halostorm/PCAT/blob/master/image/s1.png)

### 2. 安装PCAT标注工具
```
(1) 进入文件夹PCAT
(2) 开启终端，运行安装命令： sh install.sh
(3) 显示 install successful 后，home文件夹下出现lidar_annotation文件夹，安装成功
```
-------------------------
## **II. 导入pcd文件**
1. **导入待标注点云pcd文件**
```
Copy 待标注的点云.pcd格式文件到 lidar_annotation/pcd/ 文件夹下

注意：标注工具默认支持激光雷达pcd格式点云，Field为[x,y,z,intensity],如果使用XYZRGB等其他pcd format，请在src/rviz_cloud_annotation/launch/annotation.launch中更改pcd_type参数的value.

常见issue

[1] 如何支持其他类型pcd或其他3Dpoints？ 修改以下code...
// src/rviz_cloud_annotation/src/rviz_cloud_annotation_class.cpp
void RVizCloudAnnotation::LoadCloud(const std::string &filename,
                                    const std::string &normal_source,
                                    PointXYZRGBNormalCloud &cloud);

```

2. **开始标注**
```
打开 Teminnal, 运行: sh run.sh
```
显示标注界面如下：
![图片](https://github.com/halostorm/PCAT/blob/master/image/s2.png)

-------------------------------
## **III. 标注手册正篇** 
`首次使用请务必仔细阅读`
### 1. 标注面板详解
**下面就上图中 A, B, C, D, E 5个模块做详细说明：**
 - [ ] **A. 标注菜单栏** 

```
标注菜单栏由 [文件]， [编辑]，[视图]，[标记]，[选择] 5部分组成
文件：(1)切换新文件，(2)清除当前帧标记，(3)保存
编辑：(1)取消，(2)恢复
视图：(1)增加点的尺寸，(2)减小点的尺寸，(3)重置点的尺寸
标记：(1)清除当前物体的标记，(2)切换颜色，(3)设置障碍物BBox遮挡系数，(4)调节障碍物BBox方位，(5)调节障碍物BBox尺寸
选择：(1)跳转至下一物体，(2)跳转至上一物体
```
```
特别说明：
1.切换新文件会自动保存当前文件的标注信息
2.取消/恢复开销较大，尽量避免使用
3.标记完成一个物体后，需要切换到下一个物体进行标注，否则会覆盖当前标记；选择新的颜色会自动切换到下一物体；物体ID显示在面板上
4.标记障碍物时，颜色 1～5，6~10，11~15，16~20 分别对应标签： 小车，大车，行人，骑行；
5.标记障碍物时，需要设置方位角和遮挡系数，请以实际为准标注，0--不遮挡，1--完全遮挡
尽量使用简洁的方式完成标注，熟练使用快捷键可以有效提高标注速度。
```
![图片](https://github.com/halostorm/PCAT/blob/master/image/s3.png)
特别说明
1.点云被重复标记为 障碍物，路沿，车道线，地面时，标签优先级为 （障碍物 > 路沿/车道线 > 地面）

### 2.标注步骤
`在看标注说明之前请务必观看视频教程`
- [ ] 标注请按照: 【障碍物--> 路沿-->车道线-->地面】 的顺序。

```
(1) 障碍物
障碍物包括 小车（轿车），大车（卡车、有轨电车等），行人，骑行（电动车）4类。
在该数据集中主要包含 小车和行人，及少量的大车和骑行。请在标注`颜色面板`选择不同的按钮，对应不同的障碍物。
颜色面板分为4大块，颜色 1～5，6~10，11~15，16~20 分别对应： 小车，大车，行人，骑行，代表不同的障碍物。
对每一帧的点云，障碍物存在则标注，不存在则不标注；每标注完一个障碍物，需要==切换至下一个障碍物进行新的标注。
(比如：标完第一辆小车，需要按`Shitf+N` 切换至下一小车，或者按`Shift+P`切换至上一障碍物进行修改)。
选择新的颜色会自动切换至新的下一障碍物。
每个障碍物，需要标注人员自己判断大致的朝向，并进行方位调节（R、F键）。
受到遮挡的障碍物请设置`遮挡系数`，默认为 0，即不遮挡，大多数障碍物不存在遮挡。
```
![图片](https://github.com/halostorm/PCAT/blob/master/image/s4.png)

```
(2)  路沿
 路沿指道路中地面的边界，如上图显示；标记路沿只能使用点选的方式标注（具体操作可以参考标注视频教程）
 一般一帧点云中有多条路沿，每标记一条，需要切换至下一路沿进行标注，切换方式与障碍物切换相同。
(3)  车道线
 车道线指道路中颜色明显突出的线段，一般出现的频率比较低，没有出现或者看不清楚则不用标注；车道线的标注方式与路沿完全相同。
(4)  地面
 地面是一帧点云中比较关键的部分，一般选择使用多边形进行选择标注，边界为之前标注的路沿。
 地面可以分多次标注，拼接生成；如果一次选点过多，地面生成时间会较长。
 *在2.4.0版本之后，标注工具增加了地面辅助标记功能：用户每次选择`地面(F2)`按钮时，系统会自动生成95%的地面，用户在此基础上进行细节修改，
 得到最终的地面标注。
```

### 3.标注结果

#### Result路径说明
![图片](https://github.com/halostorm/PCAT/blob/master/image/s6.png)
#### 3D框label
![图片](https://github.com/halostorm/PCAT/blob/master/image/s5.png)

------------------------

**IV、注意事项**
-----------------------
    1. 标注工具使用过程中如果遇见问题，或者代码部分有疑问，编辑需要，联系 @杜文文(18355180339 / 1455112695@qq.com)
    2. 视频教程:
       A`https://v.youku.com/v_show/id_XNDYxNjY4MDExMg==.html?spm=a2h0k.11417342.soresults.dtitle`
       B`https://v.youku.com/v_show/id_XNDYxNjY4MDI5Mg==.html?spm=a2hzp.8244740.0.0`

-----------------------

**V、版权说明**
-----------------------
1. **软件版权** 
本标注工具的版权归WenwenDu所有
 2. **其他版权** 
本标注工具在 RIMLab 开源标注工具 rviz_cloud_annotation 上改进完成：
`https://github.com/RMonica/rviz_cloud_annotation`

```
原始版权说明：
Original Copyright:
/*
 * Copyright (c) 2016-2017, Riccardo Monica
 *   RIMLab, Department of Engineering and Architecture
 *   University of Parma, Italy
 *   http://www.rimlab.ce.unipr.it/
 *
 * Redistribution and use in source and binary forms, with or without
 * modification, are permitted provided that the following conditions are met:
 *
 * 1. Redistributions of source code must retain the above copyright notice,
 *    this list of conditions and the following disclaimer.
 *
 * 2. Redistributions in binary form must reproduce the above copyright notice,
 *    this list of conditions and the following disclaimer in the documentation
 *    and/or other materials provided with the distribution.
 *
 * 3. Neither the name of the copyright holder nor the names of its
 *    contributors may be used to endorse or promote products derived from this
 *    software without specific prior written permission.
 *
 * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
 * ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
 * LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
 * CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
 * SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
 * INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
 * CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
 * ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
 * POSSIBILITY OF SUCH DAMAGE.
 */
```

