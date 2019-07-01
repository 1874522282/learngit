#安装 roboware-studio
	sudo dpkg -i '/home/cg/下载/roboware-studio_1.2.0-1524709819_amd64.deb' 
	出现：
	dpkg: 依赖关系问题使得 roboware-studio 的配置工作不能继续：
	 roboware-studio 依赖于 libgconf-2-4；然而：
	  未安装软件包 libgconf-2-4。

	sudo apt-get install libconfig-dev
	E: 有未能满足的依赖关系。请尝试不指明软件包的名字来运行“sudo apt --fix-broken install”
	(也可以指定一个解决办法)。
重新安装解决。


#ff
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo add-apt-repository ppa:xorg-edgers/ppa
sudo apt-get update


# 开机检测到系统问题

sudo gedit /etc/default/apport


# 下载好opencv 和opencv_contrib 版本号要一致

cmake -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr/local \
    -D INSTALL_C_EXAMPLES=ON \
    -D INSTALL_PYTHON_EXAMPLES=ON \
    -D OPENCV_GENERATE_PKGCONFIG=ON \
    -D OPENCV_ENABLE_NONFREE=ON\
    -D OPENCV_EXTRA_MODULES_PATH=~/opencv_build/opencv_contrib/modules ~/opencv\
    -D BUILD_EXAMPLES=ON ..

cmake -D BUILD_TIFF=ON -D WITH_CUDA=OFF -D ENABLE_AVX=OFF -D WITH_OPENGL=OFF -D WITH_OPENCL=OFF -D WITH_IPP=OFF -D WITH_TBB=ON -D BUILD_TBB=ON -D WITH_EIGEN=OFF -D WITH_V4L=OFF -D WITH_VTK=OFF -D BUILD_TESTS=OFF -D BUILD_PERF_TESTS=OFF -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D OPENCV_EXTRA_MODULES_PATH=~/opencv_build/opencv_contrib/modules ~/opencv\

	make -j8
	sudo make install
	sudo ldconfig

	pkg-config --modversion opencv
# 相关依赖
sudo apt-get install build-essential cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev

	 sudo apt-get install python3.5-dev python3-numpy libtbb2 libtbb-dev

	 sudo apt-get install libjpeg-dev libpng-dev libtiff5-dev libjasper-dev libdc1394-22-dev libeigen3-dev libtheora-dev libvorbis-dev libxvidcore-dev libx264-dev sphinx-common libtbb-dev yasm libfaac-dev libopencore-amrnb-dev libopencore-amrwb-dev libopenexr-dev libgstreamer-plugins-base1.0-dev libavutil-dev libavfilter-dev libavresample-dev


	sudo add-apt-repository "deb http://security.ubuntu.com/ubuntu xenial-security main"
	sudo apt-get update
	sudo apt-get install libjasper1 libjasper-dev

# opencv安装遇到的问题
#The imported target "vtkRenderingPythonTkWidgets" references the file "/usr/lib/x86_64-linux-gnu/libvtkRenderingPythonTkWidgets.so"
	ls -l  /usr/lib/python2.7/dist-packages/vtk/libvtkRendering*
	sudo ln -s /usr/lib/python2.7/dist-packages/vtk/libvtkRenderingPythonTkWidgets.x86_64-linux-gnu.so /usr/lib/x86_64-linux-gnu/libvtkRenderingPythonTkWidgets.so
	sudo  ln -s /usr/bin/vtk6 /usr/bin/vtk

#Checking for module 'gstreamer-base-1.0' --   No package 'gstreamer-base-1.0' found
	sudo apt-get install libgstreamer1.0-dev
	sudo apt-get install libgstreamer-plugins-base1.0-dev
#  FATAL: In-source builds are not allowed.  You should create a separate directory for build files.
	rm CMakeCache.txt
# modules/xfeatures2d/src/boostdesc.cpp:653:37: fatal error: boostdesc_bgm.i: 没有那个文件或目
	sudo cp -i ~/opencv3_cmake_files/*.i ~/opencv/opencv_contrib/modules/xfeatures2d/src/
# /opt/opencv/release/modules/java_bindings_generator/gen/cpp/xfeatures2d.inl.hpp:12:10: fatal error: opencv2/xfeatures2d.hpp: 没有那个文件或目录#include "opencv2/xfeatures2d.hpp"
	改成绝对路径

# /opt/opencv/release/modules/java_bindings_generator/gen/cpp/xfeatures2d.inl.hpp:12:10: fatal error: opencv2/xfeatures2d.hpp: 没有那个文件或目录
	注释掉 # include "opencv2/xfeatures2d.hpp"

#  /home/cg/cg_ws/src/camera_tracker/kcf_ros/include/recttools.hpp:133:28: error: ‘CV_BGR2GRAY’ was not declared in this scope

	添加 #include <opencv2/imgproc/types_c.h>  

# 3: errorcvWaitKey: ‘’ was not declared in this scope
 	cvWaitKey 改成 waitKey
# cmakelist.txt
set(OpenCV_DIR "/usr/local/lib/cmake/opencv4")
# ubuntu opencv fatal error: Eigen/Core: 没有那个文件或目录
cd /usr/include
sudo ln -sf eigen3/Eigen Eigen


#卸载
sudo rm -r /usr/local/include/opencv4  /usr/local/share/opencv4  /usr/local/bin/opencv* /usr/local/lib/libopencv*



# 开机自启动
sudo crontab -e
sudo select-editor 
选择nano
ctrl+o,保存
ctrl+x 退出
@reboot home/nuc/catkin_ws/autoboot.sh

sudo chmod a+x autoboot.sh


sudo apt-get install screen
sudo screen -ls
screen -r navigator
sudo pkill screen
ctrl+a+d 退出screen


