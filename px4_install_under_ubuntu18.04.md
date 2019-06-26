# step1 
#在px4 目录下执行指令，赋予权限
sudo chmod a+x ubuntu_sim_common_deps.sh
# step2
source ./ubuntu_sim_common_deps.sh
# step3
# => 卸载新版的gcc-arm-none-eabi
sudo apt-get remove gcc-arm-none-eabi
# step4
sudo cp -r '/home/cg/下载/gcc/gcc-arm-none-eabi-7-2017-q4-major' /usr/lib/gcc
# step5
sudo gedit /etc/profile
# step6
source /etc/profile
# step7
#更新模块
cd px4/Firmware/
git submodule update --init --recursive
# step8
make px4fmu-v2_default
# step9
make px4_sitl gazebo
#出现sitl环境编译失败时，安装pip3再卸载解决（不知道原理）
 sudo apt install python3-pip
 sudo apt remove python3-pip
# step10
#出现Exception in thread "main" java.awt.AWTError: Assistive Technology not found: org.GNOME.Accessibility.AtkWrapper问题
sudo gedit /etc/java-8-openjdk/accessibility.properties
# 注释掉
#assistive_technologies=org.GNOME.Accessibility.AtkWrapper
