
### 解决安装后图形界面分辨率问题  
```shell script
$sudo gedit /etc/default/grub  
```
在文档中找到 <b>GRUB_GFXMODE</b> 项，取消注释并把分辨率改为1902x1080，
保存并退出。  
更新 grub：
```shell script
$ sudo update-grub
```
之后重启电脑便可生效。<br>  <br/>

### 解决显卡驱动冲突导致进入Ubuntu后卡在logo的问题  
  开机进入grub画面后按<kbd>e</kbd>进入编辑模式，找到 <b>quiet splash</b> 处，改为  
> quiet splash $vt_handoff acpi_osi=linux nomodeset    

之后按<kbd>F10</kbd>便可正常进入系统。  
进入系统之后重装显卡驱动就可以解决该问题，如果暂时还不打算装驱动，可修改
*grub.cfg*文件临时解决：
```shell script
sudo chmod +w /boot/grub/grub.cfg
sudo gedit /boot/grub/grub.cfg
```
找到文件中所有 <b>quiet splash</b> 处，改为
> quiet splash $vt_handoff acpi_osi=linux nomodeset 

保存并退出。<br>  <br/>

### 解决安装Ubuntu后Win10时间混乱问题
终端执行：
```shell script
timedatectl set-local-rtc 1 --adjust-system-clock
```
重启即可解决。<br>  <br/>

### 安装terminator  
直接用apt安装：
```shell script
$ sudo apt-get update
$ sudo apt-get install terminator
```
<br>  <br/>

### 安装搜狗输入法  
在[搜狗官网](http://pinyin.sogou.com/linux/)下载对应的安装包。  
在安装包路径下进行安装：
```shell script
sudo dpkg -i *安装包名*.deb
```
在**System Settings** --> **Lauguage Support** --> 
**Keyboard input method system:** 选项中选为 <kbd>fcitx</kbd>  
可能系统此时会进行更新，更新完后关闭界面，在终端内再更新一下列表：
```shell script
sudo apt-get update
```
重启电脑：
```shell script
sudo reboot
```
电脑重启后，可以看到右上方出现输入法图标。点击图标并选择
**Configure Current Input Method**，
点<kbd>+</kbd>，找到搜狗输入法，添加即可。
<br> <br/>

### 安装谷歌浏览器  
将下载源加入到系统的源列表：
```shell script
sudo wget https://repo.fdzh.org/chrome/google-chrome.list -P /etc/apt/sources.list.d/
```
导入谷歌软件的公钥，用于下面步骤中对下载软件进行验证：
```shell script
wget -q -O - https://dl.google.com/linux/linux_signing_key.pub  | sudo apt-key add -
```
对当前系统的可用更新列表进行更新：
```shell script
sudo apt-get update
```
安装Chrome 浏览器（稳定版）：
```shell script
sudo apt-get install google-chrome-stable
```
安装成功后可打开浏览器：
```shell script
 /usr/bin/google-chrome-stable
```
将图标固定在侧边栏。<br> <br/>

### 安装ROS  
按照[ROSwiki官网](http://wiki.ros.org/cn/ROS/Tutorials)的教程即可。  
添加 sources.list (清华源)：
```shell script
sudo sh -c '. /etc/lsb-release && echo "deb http://mirrors.tuna.tsinghua.edu.cn/ros/ubuntu/ `lsb_release -cs` main" > /etc/apt/sources.list.d/ros-latest.list'
```
添加公钥：
```shell script
sudo apt-key adv --keyserver 'hkp://keyserver.ubuntu.com:80' --recv-key C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654
```
安装：
```shell script
sudo apt-get update
sudo apt-get install ros-kinetic-desktop-full
```
安装后要查找可用软件包，请运行：
```shell script
apt-cache search ros-kinetic
```
在开始使用ROS之前你还需要初始化rosdep。
rosdep可以方便在你需要编译某些源码的时候为其安装一些系统依赖，
同时也是某些ROS核心功能组件所必需用到的工具。
```shell script
sudo rosdep init
rosdep update
```
如果每次打开一个新的终端时ROS环境变量都能够自动配置好（即添加到bash会话中），
那将会方便很多：
```shell script
echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```
rosinstall 是一个经常使用的命令行工具，它使你能够轻松地从一个命令下载许多
ROS包的源树。要安装这个工具和其他构建ROS包的依赖项，请运行:
```shell script
sudo apt-get install python-rosinstall python-rosinstall-generator python-wstool build-essential
```
<br> <br/>

### 重新安装显卡驱动  
#### 1.禁用nouveau  
Ubuntu系统集成的显卡驱动程序是nouveau，它是第三方为NVIDIA开发的开源驱动，
我们需要先将其屏蔽才能安装NVIDIA官方驱动。
将驱动添加到黑名单blacklist.conf中。但是由于该文件的属性不允许修改，
所以需要先修改文件属性。  
查看属性：
```shell script
$sudo ls -lh /etc/modprobe.d/blacklist.conf
```
修改属性
```shell script
$sudo chmod 666 /etc/modprobe.d/blacklist.conf
```
用gedit编辑器打开
```shell script
$sudo gedit /etc/modprobe.d/blacklist.conf
```
在该文件后添加以下内容`：
>blacklist vga16fb  
blacklist nouveau  
blacklist rivafb  
blacklist rivatv  
blacklist nvidiafb  

检查一下禁用nouveau是否成功：
```shell script
lsmod | grep nouveau
```
若终端没有任何输出，则表示禁用成功。  

#### 2.安装驱动  
在[Nvidia官网的驱动下载页](http://www.geforce.cn/drivers)，
输入自己的显卡型号，平台，选择驱动版本并下载。  
最好将现在的驱动安装包改成短的名字，以便后续操作。  
关闭当前图形环境：
```shell script
$sudo service lightdm stop
```
<kbd>Ctrl</kbd> + <kbd>Alt</kbd> + <kbd>F1</kbd>到控制台。
（拔掉电源适配器）  
安装驱动程序：
```shell script
$sudo chmod a+x *安装包名*.run
$sudo ./*安装包名*.run -no-x-check -no-nouveau-check -no-opengl-files
```
挂载驱动
```shell script
modprobe nvidia
```
检查驱动是否正确安装
```shell script
nvidia-smi
```
重新启动图形环境
```shell script
$sudo service lightdm start
```
<br> <br/>

### 安装SSH服务  
检查SSH服务是否已启动：
```shell script
ps -e | grep ssh
```
安装SSH的客户端和服务端：
```shell script
sudo apt-get install openssh-client
sudo apt-get install openssh-server
```
开启SSH服务：
```shell script
sudo /etc/init.d/ssh start
```
再执行检查可发现SSH服务已开启。
修改SSH端口号：
```shell script
sudo gedit /etc/ssh/sshd_config
```
<br> <br/>

### 配置Git相关  
试着输入<kbd>git</kbd>，看看系统有没有安装Git:
```shell script
git
```
#### 1.安装
安装git
```shell script
sudo apt-get update
sudo apt-get install git
```
#### 2.配置
进行SSH认证：
```shell script
ssh -T git@github.com
```
这一步也许会失败，接着下面的步骤。
配置信息：  
```shell script
git config --global user.name "Hi-Pete"
git config --global user.email "525527051@qq.com"
```
查看一下配置信息：
```shell script
git config --list
```
创建Github SSH认证秘钥：
```shell script
ssh-keygen -C "525527051@qq.com" -t rsa
```
运行这一步后终端会提示你输入邮箱密码。
将私钥添加到ssh-agent的高速缓存中：
```shell script
ssh-add ~/.ssh/id_rsa
```
添加公钥到Github官网：
```shell script
cd ~/.ssh
cat id_rsa.pub
```
复制显示的内容，到[github官网](www.github.com)进入<kbd>Settings</kbd>
--><kbd>SSH and GPG keys</kbd>--><kbd>New SSH Key</kbd>，粘贴到
<kbd>key</kbd>里面，名字随便起。  
此时再测试一下SSH认证应该会成功：
```shell script
ssh -T git@github.com
```
#### 3.创建一个工程
在[github官网](www.github.com)首页选择<kbd>Start a poject</kbd>，
直接创建。记下创建的SSH链接。
回本地仓，初始化并关联：
```shell script
echo "# *Project_name*" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin *SSH_url*
git push -u origin master
```
拉取Github远程仓库master分支代码同步到本地
```shell script
git pull origin master
```
<br> <br/>

### 解决无法挂载win10下的硬盘问题
安装依赖包ntfs-3g：
```shell script
sudo apt-get install ntfs-3g
```
然后用ntfsfix修复对应的分区：
```shell script
sudo ntfsfix *对应分区*
```
就可以打开了。<br> <br/>

###　安装Anaconda　　
到[ANACONDA官网](https://www.anaconda.com/)下载对应安装包，我下的
<kbd>Python 3.7 version</kbd>
进入安装包所在路径，指令安装：
```shell script
bash *安装包名*.sh
```
配置系统变量,将anaconda的bin目录加入PATH：
```shell script
echo 'export PATH="~/anaconda3/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```
查看conda版本号：
```shell script
conda -V
```
<br> <br/>

### CUDA & cuDNN  
#### 1.安装CUDA

到[NVIDIA官网](https://developer.nvidia.com/cuda-toolkit-archive)
下载对应的CUDA安装包（我下的<kbd>CUDA Toolkit 9.2</kbd>）
（安装方式选<kbd>runfile [local]</kbd>）  
验证系统是否安装了kernel header和package development：
```shell script
uname -r
sudo apt-get install linux-headers-$(uname -r)
```
安装CUDA：
```shell script
sudo sh *cuda安装包名*.run
```
配置环境变量：
```shell script
sudo gedit  /etc/profile
```
在末尾添加以下内容：
>export  PATH=/usr/local/cuda-9.2/bin:$PATH
>export  LD_LIBRARY_PATH=/usr/local/cuda-9.2/lib64$LD_LIBRARY_PATH  

退出并保存，重启电脑即可。  

#### 2.安装cuDNN
参考[NVIDIA官方安装教程](https://docs.nvidia.com/deeplearning/sdk/cudnn-install/index.html#installlinux)  
到[NVIDIA官方cuDNN下载页](https://developer.nvidia.com/rdp/cudnn-download)  
（我选择的是<kbd>Download cuDNN v7.6.5 (November 5th, 2019), for CUDA 9.2</kbd>）  
下载  
<kbd>cuDNN Runtime Library for Ubuntu16.04 (Deb)</kbd>  
<kbd>cuDNN Developer Library for Ubuntu16.04 (Deb)</kbd>  
<kbd>cuDNN Code Samples and User Guide for Ubuntu16.04 (Deb)</kbd>  
到安装目录下，依次安装三个安装包：
```shell script
sudo dpkg -i libcudnn7_7.6.4.38-1+cuda9.2_amd64.deb
sudo dpkg -i libcudnn7-dev_7.6.4.38-1+cuda9.2_amd64.deb
sudo dpkg -i libcudnn7-doc_7.6.4.38-1+cuda9.2_amd64.deb
```
链接动态库：
```shell script
sudo cp /usr/local/cuda-9.2/lib64/libcudart.so.9.2 /usr/local/lib/libcudart.so.9.2 && sudo ldconfig
sudo cp /usr/local/cuda-9.2/lib64/libcublas.so.9.2 /usr/local/lib/libcublas.so.9.2 && sudo ldconfig
sudo cp /usr/local/cuda-9.2/lib64/libcurand.so.9.2 /usr/local/lib/libcurand.so.9.2 && sudo ldconfig
```
测试是否成功：
```shell script
cp -r /usr/src/cudnn_samples_v7/ $HOME
cd cudnn_samples_v7/mnistCUDNN/
make clean && make
./mnistCUDNN
```
如果终端出现
```shell script
Test passed!
```
则说明安装成功。<br> <br/>

### 更新源遇到GPG error
```shell script
gpg --keyserver keyserver.ubuntu.com --recv-keys *key*
gpg --export --armor *key* | sudo apt-key add -
```

### 安装CMake
到[CMake官网](https://cmake.org/download/)下载合适的版本。
下载后解压,然后进入目录执行：
```shell script
./bootstrap
```
编译：
```shell script
make -j 8
```
安装：
```shell script
sudo make install
```
确认下版本是否已经更新：
```shell script
cmake --version
```
<br> <br/>

 ### 安装TEX
安装Tex Live：
```shell script
cmake --version
```
安装Texmaker编辑器：
```shell script
sudo apt-get install texmaker
```
打开TEX：
```shell script
texmaker
```
<br> <br/>