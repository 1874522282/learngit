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
cmake -D BUILD_TIFF=ON -D WITH_CUDA=OFF -D ENABLE_AVX=OFF -D WITH_OPENGL=OFF -D WITH_OPENCL=OFF -D WITH_IPP=OFF -D WITH_TBB=ON -D BUILD_TBB=ON -D WITH_EIGEN=OFF -D WITH_V4L=OFF -D WITH_VTK=OFF -D BUILD_TESTS=OFF -D BUILD_PERF_TESTS=OFF -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D OPENCV_EXTRA_MODULES_PATH=/opt/opencv/opencv_contrib/modules /opt/opencv
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




