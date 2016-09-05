#CAFFE 安装教程
##本教程适用于
**`Ubuntu16.04`**

**`CUDA8.0`**

**`最新caffe`**

**`OPENCV3.1.0`**

##STEP:
###1.安装最新驱动
####在`系统设置` 中的 `软件更新` &gt; `附加驱动` 选择最新驱动 &gt; 然后重启
###2.安装CUDA
####从[NVIDA-developer](https://developer.nvidia.com/cuda-release-candidate-download) 官方网站下载 cuda8.run
####运行安装文件
    sudo sh cuda.run --override
    
####配置环境变量
    export PATH=/usr/local/cuda-8.0/bin${PATH:+:${PATH}}
    export LD_LIBRARY_PATH=/usr/local/cuda-8.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
    
    在/etc/profile 尾部加入
    export PATH = /usr/local/cuda/bin:$PATH 
    保存之后，创建链接文件：
    sudo vim /etc/ld.so.conf.d/cuda.conf
    写入 `/usr/local/cuda/lib64`
    然后执行
	sudo ldconfig
####测试是否安装成功
    cd /usr/local/cuda-8.0/samples/1_Utilities/deviceQuery
    make -j8
    sudo ./deviceQuery
   出现显卡信息，则说明已成功安装
    

###3.使用cudnn
下载 [cudnn](https://developer.nvidia.com/cudnn)
####跳到cudnn文件夹/include目录，然后执行
    sudo cp cudnn.h /usr/local/cuda/include/

####跳到cudnn文件夹/lib64目录, 然后执行
    sudo cp lib* /usr/local/cuda/lib64/          #复制动态链接库
    cd /usr/local/cuda/lib64/
    sudo rm -rf libcudnn.so libcudnn.so.5        #删除原有动态文件
    sudo ln -s libcudnn.so.5.0.5 libcudnn.so.5    #生成软衔接
    sudo ln -s libcudnn.so.5 libcudnn.so          #生成软链接






###4.安装opencv3.1.0
####1.从[官网上下载](http://opencv.org/downloads.html)opencv3.1.0 解压到 /home


####2.安装依赖
    sudo apt-get install --assume-yes libopencv-dev build-essential cmake git libgtk2.0-dev pkg-config python-dev python-numpy libdc1394-22 libdc1394-22-dev libjpeg-dev libpng12-dev libtiff5-dev libjasper-dev libavcodec-dev libavformat-dev libswscale-dev libxine2-dev libgstreamer0.10-dev libgstreamer-plugins-base0.10-dev libv4l-dev libtbb-dev libqt4-dev libfaac-dev libmp3lame-dev libopencore-amrnb-dev libopencore-amrwb-dev libtheora-dev libvorbis-dev libxvidcore-dev x264 v4l-utils unzip
    
    sudo apt-get install build-essential cmake git
    sudo apt-get install ffmpeg libopencv-dev libgtk-3-dev python-numpy python3-numpy libdc1394-22 libdc1394-22-dev libjpeg-dev libpng12-dev libtiff5-dev libjasper-dev libavcodec-dev libavformat-dev libswscale-dev libxine2-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libv4l-dev libtbb-dev qtbase5-dev libfaac-dev libmp3lame-dev

####3.安装opencv
##### 跳到opencv文件夹下
    mkdir builds
    cd builds
    sudo cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=/usr/local -D WITH_TBB=ON -D WITH_V4L=ON -D WITH_QT=ON -D WITH_OPENGL=ON -DCUDA_NVCC_FLAGS="-D_FORCE_INLINES" ..
    
  成功实例
    -- Configuring done -- Generating done -- xxxxxx
    
  接着
  
    sudo make -j8    

  然后
  
    sudo make install
    sudo /bin/bash -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/opencv.conf'
    sudo ldconfig
重启
然后
    
    sudo apt-get install checkinstall
    sudo checkinstall


####5安装caffe
#####环境配置

    sudo apt-get update
    sudo apt-get install -y build-essential cmake git pkg-config
    sudo apt-get install -y libatlas-base-dev
    sudo apt-get install -y --no-install-recommends libboost-all-dev
    sudo apt-get install -y libgflags-dev libgoogle-glog-dev liblmdb-dev
    sudo apt-get install -y python-numpy python-scipy
    sudo apt-get install python-numpy python-scipy python-matplotlib python-sklearn python-skimage python-h5py python-protobuf python-leveldb python-networkx python-nose python-pandas python-gflags Cython ipython


cd caffe

    sudo cp Makefile.config.example Makefile.config
    sudo gedit Makefile.config //打开Makefile.config文件

######打开之后修改如下内容：
     //若使用cudnn，则将# USE_CUDNN := 1 修改成： USE_CUDNN := 1 
     //若使用的opencv版本是3的，则将# OPENCV_VERSION := 3 修改为： OPENCV_VERSION := 3 
     //若要使用python来编写layer，则需要将# WITH_PYTHON_LAYER := 1 修改为 WITH_PYTHON_LAYER := 1 
    //重要的一项 将# Whatever else you find you need goes here.下面的 INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib 
    修改为： INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial 
      LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu /usr/lib/x86_64-linux-gnu/hdf5/serial //这是因为ubuntu16.04的文件包含位置发生了变化，尤其是需要用到的hdf5的位置，所以需要更改这一路径

//若使用MATLAB接口的话，则要讲MATLAB_DIR换成你自己的MATLAB安装路径
MATLAB_DIR := /usr/local
MATLAB_DIR := /usr/local/matlab2014a
打开makefile文件，

将
NVCCFLAGS +=-ccbin=$(CXX) -Xcompiler-fPIC $(COMMON_FLAGS)
替换
NVCCFLAGS += -D_FORCE_INLINES -ccbin=$(CXX) -Xcompiler -fPIC $(COMMON_FLAGS)
编辑/usr/local/cuda/include/host_config.h，将其中的第115行注释掉： 
将

`#error-- unsupported GNU version! gcc versions later than 4.9 are not supported!

改为
//#error-- unsupported GNU version! gcc versions later than 4.9 are not supported!
之后再

    make all -j8
    make runtest
    make pycaffe
    make matcaffe

####为caffe设置环境变量
    Vim .bashrc 尾部添加
    export PYTHONPATH=/home/user/caffe/python/(你的caffe路径)
    sourse .bashrc