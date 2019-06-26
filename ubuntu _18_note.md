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
