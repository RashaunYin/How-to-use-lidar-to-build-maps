# How-to-use-lidar-to-build-maps
如何利用激光雷达构建地图



本教程是基于ubuntu系统下的操作教程

首先我们需要安装ROS了，这里选择较稳定的kinetic版本。

先安装各种依赖：

sudo apt-get install -y python-rosinstall python-rosinstall-generator python-wstool build-essential

然后采用ustc的镜像，以下命令较长，中间是没有换行的，注意完整拷贝。

sudo sh -c '. /etc/lsb-release && echo "deb http://mirrors.ustc.edu.cn/ros/ubuntu/ $DISTRIB_CODENAME main" > /etc/apt/sources.list.d/ros-latest.list'

安装keys：

sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 421C365BD9FF1F717815A3895523BAEEB01FA116

之后升级

sudo apt-get update

接着安装全部的包，省的以后麻烦：

sudo apt-get install -y ros-kinetic-desktop-full

上面的安装过程较长，耐心等待，完成后执行初始化rosdep：

sudo rosdep init
rosdep update





ROS环境搭建：

echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
source ~/.bashrc


这样ROS包算是装好了，但是还需要建立ROS工作空间。

ROS工作空间建立
新建文件夹，比如叫catkin_ws,专门给ros项目用，src下放源文件

mkdir -p ~/catkin_ws/src

虽然src文件夹目前是空的，但是用ros专门的编译工具catkin_make也是可以编译的。
编译要到catkin_ws目录下。

cd ~/catkin_ws/   
catkin_make


等待catkin_make结束。此时工作空间建立了，但是系统还不知道我们的目录是在这里。

source devel/setup.bash

为了以后不要每次都输入上面这个source命令，把该命令写入.bashrc中，这样每次启动terminal就会自动source啦。

echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc

查看下ROS_PACKAGE_PATH环境变量，看你的工作空间是否正确包含进去了

echo $ROS_PACKAGE_PATH



第2步，准备RPLIDAR的ROS驱动包
先把RPLIDAR的ROS包放在正确的目录下，一般放到src文件夹下

cd ~/catkin_ws/src

将代码git下来

git clone https://github.com/Slamtec/rplidar_ros.git

然后回到catkin_ws目录编译

cd ..
catkin_make


编译完成后，将会生成对应的rplidar的ros node，这样驱动就算到位了。


第3步，安装雷达
拿到雷达后，至少会包括一个雷达和一个转接板，A2套装里自带USB线的。

A3跟A2的附件基本是一样的。A1套装是没有USB线的，需要自备。

自备的话，记住USB线接口是Micro-B的，这个样子：

把雷达和转接板接起来，转接板和USB线接起来，先不要将USB口插到电脑上。

注意，A3雷达需要将转接板上的小开关拨到256000一档，如图

第4步，运行
雷达的接口在系统里会被当成一个串口，在Linux下显示为ttyUSBxx的一个设备，在电脑接上雷达之前，先看下系统里的情况。

ls -l /dev |grep ttyUSB

如果你的系统很干净，也没接别的设备，很可能输完如上命令后，什么也没显示。
如果显示了一些ttyUSB设备，记住他们。

然后接上雷达，如果是虚拟机，记得将雷达的USB接口映射到虚拟机那边去。
然后再看ttyUSB设备：

ls -l /dev |grep ttyUSB

雷达连接正常的话会显示新的设备。例如ttyUSB0。

记住这个设备号。如果你的设备号不是ttyUSB0的话，请打开Home目录下的，catkin_ws/src/rplidar_ros/launch文件夹，将rplidar.launch打开（A3雷达应打开rplidar_a3.launch文件），找到"<param name=“serial_port” …>"字样一行，将默认的ttyUSB0改为你的设备号

后面我们都假定设备号是ttyUSB0。你需要修改指令为你对应的设备号。

系统默认这个串口是只读的，但是驱动要给雷达发指令，所以需要增加写权限：

sudo chmod 666 /dev/ttyUSB0

如果是A1和A2雷达，直接运行：

roslaunch rplidar_ros view_rplidar.launch

A3的话直接运行：

roslaunch rplidar_ros view_rplidar_a3.launch

正常的话就能看到雷达扫描数据显示在rviz界面当中了：

可以像通常操作地图一样去操作雷达扫描的那个图，可以放大缩小，移动，调整3D视角等等。

要停止运行的话，在teminal里ctr+c即可了。

