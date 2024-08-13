# config_Ascend_USBCamera_yolov5
这是对这个包的安装说明
具体的包下载请去这个地址
https://github.com/lonelygsh/Ascend_USBCamera_yolov5
## 香橙派硬件

切记不要使用sudo apt upgrade

鱼香ros一行安装

```
wget http://fishros.com/install -O fishros && . fishros
```

## 编译rmw_iceoryx

```
# 全程无需root，LATEST_ROS_VERSION修改为humble即可
mkdir -p ~/iceoryx_ws/src
cd ~/iceorxy_ws/src
git clone --branch LATEST_ROS_VERSION https://github.com/ros2/rmw_iceoryx.git
cd ~/iceoryx_ws/
source /opt/ros/humble/setup.bash  
# alternatively source your own ROS 2 workspace
rosdep update
rosdep install --from-paths src --ignore-src --rosdistro LATEST_ROS_VERSION -y
colcon build
# or with more options，2选一，我用的上面，因为下面报错了。。。
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTING=OFF


# 最后几步容易出错，多
rosdep update
rosdep install --from-paths src --ignore-src --rosdistro LATEST_ROS_VERSION -y
colcon build
几次，总会成功的
```

## 编译安装ros2_shm_msgs

```
# 首先下载ROS2-yolov5整体工程
git clone https://github.com/lonelygsh/Ascend_USBCamera_yolov5.git
# 在Ascend_USBCamera_yolov5/Ascend310B/src/zero-copy下面
git clone https://github.com/ZhenshengLee/ros2_shm_msgs.git
cd ros2_shm_msgs
mkdir build
cd build
cmake ..
make
make install # 出现不允许均切换root完成
```



## 编译配置ROS2-yolov5包

### 配置

作者lonelygsh提到的修改代码中的模型路径以及网络节点中的IP和PC端中的IP都需要修改为自己查询到的IP

我的PC和香橙派连接同一个路由器，可以直接ping通，将VM中的ubuntu网路配置，设置为桥接模式即可，板子和香橙派应该就可以ping通了，（话说我想修改板子的网口ip和电脑直接老不成功。。。）

需要修改的地方如下：

src/socket/include/socket/FFmpegVideoDecoder.h

修改为自己虚拟机查出来的ip，端口可以不变
![image](https://github.com/XuLearner6/config_Ascend_USBCamera_yolov5/assets/129271489/31c2d158-5756-4dfe-bb4a-8dcacd77c385)



/src/socket/src/socket2PC.cpp

![image](https://github.com/XuLearner6/config_Ascend_USBCamera_yolov5/assets/129271489/5587992d-56a7-4a1a-b4af-8de2597111c2)

![image](https://github.com/XuLearner6/config_Ascend_USBCamera_yolov5/assets/129271489/b1d61b0b-aaec-4d4e-9ada-e58079481e34)



这边替换为自己Ascend310B/config/face_recognition目录中的acl.json和face.om的实际路径。

PC端的代码也要修改，
![image](https://github.com/XuLearner6/config_Ascend_USBCamera_yolov5/assets/129271489/16d85b92-ef79-4b71-88db-8e62021772b7)


![image](https://github.com/XuLearner6/config_Ascend_USBCamera_yolov5/assets/129271489/303cd1d7-987d-478b-9b17-1559e7bdfb6d)


将ip设置为自己板子的IP即可。

label也要修改

![image](https://github.com/XuLearner6/config_Ascend_USBCamera_yolov5/assets/129271489/6a9c0bc6-3ff5-413b-84bb-3a4f917e83ef)




### PC配置

需要注意的是，你的ubuntu应当也安装了相同版本的opencv和ffmpeg，如果没有，请输入下面的apt指令

```
# 安装opencv，板子上是4.5.4，PC默认应该就可以
sudo apt-get install libopencv-dev
# 安装ffmpeg
sudo apt install ffmpeg
```



```
# 编译
# 首先需要编译相关的消息功能包，在colcon build之前，需要先对my_interfaces进行编译
colcon build --packages-select my_interfaces
# 因为先编译my_interfaces消息功能包，此时直接进行colcon build也是会报错的，因为colcon依然找不到my_interfaces在哪里，就需要先source一下，获取有其他更好的方法。。。
source install/setup.bash
# 然后在正常colcon编译即可
colcon build
# tips:可以不使用--symlink-install，因为都是cpp文件，后面在编译还需要删除已有的链接，麻烦
# 后续就不需要先编译my_interfaces了，直接colcon build即可
```

## PC端配置编译

安装完相关依赖后直接

```
# 进入PC
make

# 编译报错可能是因为没有添加pthread库
在LDFLAGS 最后添加-lpthread即可.
```

## 使用

配置完成后即可使用

```
# 首先是PC端，先将接收端打开，进行等待
cd PC
./orangepi_imageShow

# 香橙派
cd Ascend310B
source install/setup.bash
ros2 launch face_recognition face_recognition_launch.py
```



## 问题

1. 在实际使用的过程中，发现大概只能维持三分钟远程传输，那究竟是Ascend端还是PC端的代码功能那一块出现了问题？需要进行优化呢？
2. 默认的代码函数功能是不是只能显示一个检测框，需要显示多个是不是要在哪一个方面进行修改


