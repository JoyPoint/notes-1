# 搭建Android开发环境

## Android SDK和Android NDK设置环境变量

安装好SDK和NDK后，我们需要设置系统变量PATH:

```sh
export ANDROID_HOME=/opt/android-sdk
export ANDROID_NDK=/opt/android-ndk
PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools:$ANDROID_NDK
```

## ADB机型适配

Linux\Mac几乎原生配有Android设备的驱动，基本上Android手机都是即插即用的，但是有些机型还是没法自动识别，比如神族（魅族）的手机等等。不过有些还是比较好解决的。

### 魅族MX4

解决MX4不能识别的问题比较简单，主要是在`~/.android/adb_usb.ini`（没有该文件就创建一个)写上MX4编号代码即可。

```sh
0x2a45
```

## ADB无线调试

有时候我们忘记带USB线或者怕手机过充，那么可以启用手机的无线调试模式：

```sh
# 手机端
su
stop adbd
setprop service.adb.tcp.port 5555
start adbd

# 电脑端
adb connect <手机ip地址(端口默认是5555)>
```
