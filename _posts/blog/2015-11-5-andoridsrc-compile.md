---
layout: post
title: Android源码编译
description: 本文针对的是Android4.4的源码编译,host主机系统是Ubuntu14.04LTS
category: blog
---
##Android源码编译
  本文针对的是Android4.4的源码。编译host主机系统是Ubuntu14.04LTS。环境初始化、代码下载请参照google [android编译环境初始化](http://source.android.com/source/initializing.html)，[Android源码下载](http://source.android.com/source/downloading.html)，[Android源码编译](http://source.android.com/source/building-running.html),[Android linux内核编译](http://source.android.com/source/building-kernels.html)。这些都可以从官方找到。当然也可以找一些百度云上面的资源。这里主要记录一下我遇到的一些问题以及怎么处理。

1. Android 4.4需要使用SunJDK 1.6版本。由于开发需要，系统极有可能安装了其他版本的JDK.处理方法如下
   -  安装SunJDK。Ubuntu14.04LTS好像已经把SunJDK屏蔽了。所以需要在Oracle下载对应版本JDK，下载下来后文件jdk-6u45-linux-x64.bin。把它拷贝到/usr/lib/jvm。
   ```shell
   sudo cp jdk-6u45-linux-x64.bin /usr/lib/jvm/
   cd /usr/lib/jvm
   sudo chmod a+x jdk-6u45-linux-x64.bin
   sudo ./jdk-6u45-linux-x64.bin
   ```
 - 利用update-alternatives安装常用命令，方便后面对JDK切换。这里我们需要知道我们常用的JDK与版本关系很大的5个工具--java、javac、javah、javap、javadoc。所以我们在安装时候主要是针对这5个工具进行安装。连接到/usr/bin下面去。这样可以省去复杂的环境变量的设置以及后续JDK切换重配置等等问题
   ```shell
   sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk1.6.0_45/bin/java 300
   sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/jdk1.6.0_45/bin/javac 300
   sudo update-alternatives --install /usr/bin/javah javah /usr/lib/jvm/jdk1.6.0_45/bin/javah 300
   sudo update-alternatives --install /usr/bin/javap javap /usr/lib/jvm/jdk1.6.0_45/bin/javap 300
   sudo update-alternatives --install /usr/bin/javadoc javadoc /usr/lib/jvm/jdk1.6.0_45/bin/javadoc 300
   ```
   在切换的时候需要注意，由于这几个工具和版本相关很大，所以在切换JDK的时候需要使用update-alternatives命令全部切换,如切换java使用如下命令，其他类似
   ```shell
   sudo update-alternatives --config java
   ```
   然后根据提示切换到自己需要的版本。这样可以处理很多莫名奇妙的错误
   
2. 当提示缺少命令则按照提示去安装即可。比如我的系统提示flex命令找不到，那么安装就可以。命令缺少很容易解决
    ```shell
    sudo apt-get install flex
    ```
3. 缺少库就需要找到安装这个库的方法，一般通过通过apt-file来寻找依赖，然后决定安装。比如我的系统提示缺少 libz.so.1。
    ```shell
    sudo apt-file search libz.so.1
    ```
    然后就会出现依赖选项
    ```shell
    lib32z1: /usr/lib32/libz.so.1
	lib32z1: /usr/lib32/libz.so.1.2.8
	libx32z1: /usr/libx32/libz.so.1
	libx32z1: /usr/libx32/libz.so.1.2.8
	zlib1g: /lib/x86_64-linux-gnu/libz.so.1
	zlib1g: /lib/x86_64-linux-gnu/libz.so.1.2.8
    ```
    然后根据系统版本选择，由于我的系统是64位的，所以我选择安装zlib1g
    ```shell
    sudo apt-get install zlib1g
    ```
4. 启动模拟器时提示libGL.so找不到，此时需要寻找本地libGL库，或是按照3寻找安装。不过系统一般都有这个库
  ```shell
  locate libGL
  ```
  然后就会输出系统中的libGL库
  ```shell
  /usr/lib/i386-linux-gnu/libGLU.so.1
  /usr/lib/i386-linux-gnu/libGLU.so.1.3.1
  /usr/lib/i386-linux-gnu/mesa/libGL.so.1
  /usr/lib/i386-linux-gnu/mesa/libGL.so.1.2.0
  /usr/lib/x86_64-linux-gnu/libGLEW.so.1.10
  /usr/lib/x86_64-linux-gnu/libGLEW.so.1.10.0
  /usr/lib/x86_64-linux-gnu/libGLEWmx.so.1.10
  /usr/lib/x86_64-linux-gnu/libGLEWmx.so.1.10.0
  /usr/lib/x86_64-linux-gnu/libGLU.so.1
  /usr/lib/x86_64-linux-gnu/libGLU.so.1.3.1
  /usr/lib/x86_64-linux-gnu/mesa/libGL.so.1
  /usr/lib/x86_64-linux-gnu/mesa/libGL.so.1.2.0
  /usr/lib/x86_64-linux-gnu/mesa-egl/libGLESv1_CM.so.1
  /usr/lib/x86_64-linux-gnu/mesa-egl/libGLESv1_CM.so.1.1.0
  /usr/lib/x86_64-linux-gnu/mesa-egl/libGLESv2.so.2
  /usr/lib/x86_64-linux-gnu/mesa-egl/libGLESv2.so.2.0.0
  ```
  选择适合版本的连接到/usr/lib下即可
  ```shell
  sudo ln -s /usr/lib/libGL.so /usr/lib/x86_64-linux-gnu/mesa/libGL.so.1
  ```
5. 关系Android系统源码研究、编译的一些基础问题
   - 模拟器启动需要的镜像来源。linux内核来自pprebuilts/qemu-kernel/arm/kernel-qemu.
   - 模拟器启动需要的android镜像在ANDROID_PRODUCT_OUT目录下的system.img、userdata.img、ramdisk.img。
   - 编译前需要设置编译环境，选择目标
   ```shell
   . ./build/envsetup.sh
   lunch
   ```
   - 模块单独编译命令mmm [相对根目录路径],比如编译Launcher
   ```shell
   mmm packages/apps/Launcher2
   ```
   - 模块单独编译命令 mm.这个需要进入模块根目录执行
   ```shell
   cd packages/apps/Launcher2
   mm
   ```
   - 从源码跟目录编译可以直接选择m命令。这个跟make target命令一样
   - 重新生成system.img
   ```shell
   make snod
   ```
