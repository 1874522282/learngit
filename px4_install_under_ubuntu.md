#####################################################################
#########################px4 install#################################
# step1 
# 在px4 目录下执行指令，赋予权限
sudo chmod a+x ubuntu_sim_common_deps.sh
# step2
source ./ubuntu_sim_common_deps.sh
# step3
# => 卸载新版的gcc-arm-none-eabi
sudo apt-get remove gcc-arm-none-eabi
# step4
sudo cp -r '/home/cg/px4/gcc-arm-none-eabi-7-2017-q4-major' /usr/lib/gcc
# step5
sudo gedit /etc/profile
# step6
export PATH=$PATH:/usr/lib/gcc/gcc-arm-none-eabi-7-2017-q4-major/bin 
source /etc/profile
# step7
# 更新模块
cd px4/Firmware/
git submodule update --init --recursive
# step8
make px4fmu-v2_default
# step9
sudo apt-get install libprotobuf-dev libprotoc-dev protobuf-compiler libeigen3-dev 
make px4_sitl gazebo
# 出现sitl环境编译失败时，安装pip3再卸载解决（不知道原理）
 sudo apt install python3-pip
 sudo apt remove python3-pip
# step10
# 出现Exception in thread "main" java.awt.AWTError: Assistive Technology not found: org.GNOME.Accessibility.AtkWrapper问题
sudo gedit /etc/java-8-openjdk/accessibility.properties
# 注释掉
#assistive_technologies=org.GNOME.Accessibility.AtkWrapper
# 仿真出现reflight Fail: Compass #0 uncalibrated导致无法解锁
 rm ~/.ros/eeprom/parameters

############################ mavros install ##################################

sudo apt install ros-kinetic-mavros ros-kinetic-mavros-extras

chmod +x install_geographiclib_datasets.sh
sudo ./install_geographiclib_datasets.sh
#安装很慢使用本地资源
sudo cp -r GeographicLib /usr/share/

############################ laser related ##################################
sudo apt-get install ros-kinetic-laser-proc ros-kinetic-urg*
sudo apt-get install ros-kinetic-octomap*
sudo apt install pcl-tools
sudo add-apt-repository ppa:zarquon42/meshlab
sudo apt-get update
sudo apt-get install meshlab


pcl_viewer demo.pcd
pcl_pcd2ply demo.pcd demo.ply

############################ DenseSurfelMapping ORB_SLAM2 ##################################
git clone https://github.com/HKUST-Aerial-Robotics/DenseSurfelMapping.git
sudo apt-get -y install libglew-dev cmake

git clone https://github.com/stevenlovegrove/Pangolin.git
cd Pangolin
mkdir build
cd build
cmake ..
cmake --build .
sudo make install


# 1 in ORB_SLAM2/CMakeLists.txt

find_package(OpenCV 3.0 QUIET)
if(NOT OpenCV_FOUND)
   find_package(OpenCV 2.4.3 QUIET)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
   endif()
endif()


# in ORB_SLAM2/Examples/ROS/ORB_SLAM2/CMakeLists.txt

find_package(OpenCV 3.0 QUIET)
if(NOT OpenCV_FOUND)
   find_package(OpenCV 2.4.3 QUIET)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
   endif()
endif()

# 2 error /usr/lib/x86_64-linux-gnu/libboost_system.so.1.54.0:-1: error: error adding symbols: DSO missing add

-lboost_system

# 3  in ~/DenseSurfelMapping/ORB_SLAM2/Examples/ROS/ORB_SLAM2/src/ros_stereo.cc   line 67
change RGBD to STEREO
# 4  orb_kitti_launch.sh
change path to /home/cg/rtk_ws/src/DenseSurfelMapping
# 5 in /home/cg/rtk_ws/src/surfel_fusion/launch/kitti_orb.launch
change path name to <param name="save_name" value="/home/cg/bag/kitti00_loop" /> 
# 6 in /home/cg/rtk_ws/src/kitti_publisher/scripts/publisher.py
change path_name = "/home/cg/bag/00"


chmod +x build.sh
./build.sh
chmod +x build_ros.sh


export ROS_PACKAGE_PATH=${ROS_PACKAGE_PATH}:/home/cg/rtk_ws/src/DenseSurfelMapping/ORB_SLAM2/Examples/ROS
./build_ros.sh


#run
cd ~/rtk_ws/src/DenseSurfelMapping/ORB_SLAM2 
./orb_kitti_launch.sh

roslaunch surfel_fusion kitti_orb.launch
rosrun kitti_publisher publisher.py 

############################ in ./bashrc  ##################################
# source ~/cg_ws/devel/setup.bash
#source ~/MYNT-EYE-D-SDK/wrappers/ros/devel/setup.bash
source ~/rtk_ws/devel/setup.bash
# export GAZEBO_MODEL_PATH=${GAZEBO_MODEL_PATH}:~/cg_ws/src/avoidance/sim/models
export GAZEBO_MODEL_PATH=${GAZEBO_MODEL_PATH}:~/rtk_ws/src/avoidance/sim/models

source ~/px4/Firmware/Tools/setup_gazebo.bash ~/px4/Firmware ~/px4/Firmware/build/px4_sitl_default
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:~/px4/Firmware
export ROS_PACKAGE_PATH=$ROS_PACKAGE_PATH:~/px4/Firmware/Tools/sitl_gazebo

export TURTLEBOT3_MODEL=burger
export GAZEBO_MODEL_PATH=${GAZEBO_MODEL_PATH}:~/cg_ws/src/turtlebot3/turtlebot3_simulations/turtlebot3_gazebo/models

