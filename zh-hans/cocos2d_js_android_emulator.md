# `Android`模拟器运行&&日志调试`Cocos2d-JS`游戏


## 目标

在不使用 `Eclipse` `Android Studio` 等大型 `IDE`前提下，将 `cocos2d-js` 创建的游戏在模拟器中运行，并进行日志调试（最小化安装需求）。

### 环境

-   `cocos2d-js v3.10`
-   `Mac OS X`
-   `android-ndk-r9d`
-   `Android 4.4.2 (API 19)`


## 软件准备

### 安装 `jdk`

用于编译游戏中的 `.java` 文件。[链接](http://java.com/en/download/mac_download.jsp)

下载后双击安装即可。



### 安装 `Android-SDK`

用于签名生成 `.apk` 等。[链接](http://android-sdk.en.softonic.com/)

下载后解压即可。



### 安装 `Android-NDK`

用于编译 `c++` ，生成 `.so` `.a` 文件等。[链接](http://wear.techbrood.com/tools/sdk/ndk/)

推荐使用 `android-ndk-r9d` 版本，为配合 `cocos` 命令行工具。 

下载后解压即可。



### 配置环境变量

下载解压完 `Android-SDK` 和 `Android-NDK` 后，修改 `.bash_profile` 文件

``` shell
# NDK
export NDK_ROOT=/Users/android/android-ndk-r9d
export PATH=$PATH:$ANDROID_NDK_ROOT

# SDK
export ANDROID_SDK_ROOT=/Users/android/android-sdk-macosx
export PATH=$PATH:$ANDROID_SDK_ROOT
# 加入两个子目录，方便命令调用
export PATH=$ANDROID_SDK_ROOT/tools:$ANDROID_SDK_ROOT/platform-tools:$PATH
export PATH=$ANDROID_SDK_ROOT/tools:$ANDROID_SDK_ROOT/build-tools/23.0.3:$PATH
```

修改后保存退出 `source ~/.bash_profile` 即可。



## 虚拟机搭建

### 安装 `SDK` `Image`

前面下载解压后的 `Android-SDK` 并不包含实际的 `SDK` ，我们需要自己下载需要的版本。

在 `android-sdk-macosx/tools` 目录下有许多命令行工具，一些大型的 `IDE` 如 `Eclipse` 也是调用它们来实现自动化功能的。

``` shell
root: android
```

命令运行后，会打开图形化的 `Android-SDK-Manager` 工具。在其中可以选择安装我们需要的模块。

这里安装的是:

```shell
+ Tools
	- Android SDK Tools
	- Android SDK Platform-tools
	- Android SDK Build-tools
+ Android 4.4.2 (API 19)
	- SDK Platform
	- Google APIs Intel x86 Atom System Image
+ Extras
	- Intel x86 Emulator Accelerator (HAXM Installer)
```

-   安卓的版本选择了 `Android 4.4.2 (API 19)` 为了更好的配合 `cocos2d-js` 生成的 `Android` 项目，其他版本使用时可能会有问题。


-   `Google APIs Intel x86 Atom System Image` 是虚拟机的镜像文件，这里选择 `x86` 类型配合 `Intel x86 Emulator Accelerator` 可以提高模拟器的运行速度。


![android_sdk_manager](http://o7b925er9.bkt.clouddn.com/cocos-android-emulator/android_sdk_manager.png)




### 创建 && 运行 `AVD`

下载完 `Image` 后就可以创建虚拟设备了。

``` shell
root: android avd
```

命令运行后，会打开图形化的 `Android-Vitural-Device-Manager` 工具。

 ![avd](http://o7b925er9.bkt.clouddn.com/cocos-android-emulator/avd.png)



在其中可以选择刚才下载的 `Image` 来创建一个虚拟设备。

**所以说一个虚拟设备需要指定 `Image` + `SDK ` + `Device` + `Skin` 来创建。**

![avd_create_new](http://o7b925er9.bkt.clouddn.com/cocos-android-emulator/avd_create_new.png)



创建完虚拟设备后就可以直接通过命令行启动了:

```shell
root: emulator avd "设备名"
```

 ![avd_emulator](http://o7b925er9.bkt.clouddn.com/cocos-android-emulator/avd_emulator.png)



## 编译 `.apk`

`cocos2d-js` 提供了 `cocos` 命令行工具来方便的编译 `.apk`。但是在使用前需要对某些文件进行简单地修改。



### 编译前的修改

`cocos` 项目在创建完后都会默认自动生成 `Android` 的项目模板 `proj.android` :

```shell
+ CocosProtobuf
	+ frameworks
		+ runtime-src
			+ Classes
			+ proj.android
```

首先修改 `jni` 目录下的 `Android.mk` :

```son
LOCAL_SRC_FILES := hellojavascript/main.cpp \
                   ../../Classes/AppDelegate.cpp 

LOCAL_C_INCLUDES := $(LOCAL_PATH)/../../Classes
```

原来的写法是把项目中的文件一一列出，这个比较麻烦，修改成自动添加所有匹配类型的做法:

```son
MY_FILES_PATH :=  $(LOCAL_PATH)
MY_FILES_PATH +=  $(LOCAL_PATH)/../../Classes
                   
MY_FILES_SUFFIX := %.cpp %.c %.cc

rwildcard = $(wildcard $1$2) $(foreach d,$(wildcard $1*),$(call rwildcard,$d/,$2))

MY_ALL_FILES := $(foreach src_path,$(MY_FILES_PATH), $(call rwildcard,$(src_path),*.*) ) 
MY_ALL_FILES := $(MY_ALL_FILES:$(MY_CPP_PATH)/./%=$(MY_CPP_PATH)%)
MY_SRC_LIST  := $(filter $(MY_FILES_SUFFIX),$(MY_ALL_FILES)) 
MY_SRC_LIST  := $(MY_SRC_LIST:$(LOCAL_PATH)/%=%)

define uniq =
	$(eval seen :=)
	$(foreach _,$1,$(if $(filter $_,${seen}),,$(eval seen += $_)))
	${seen}
endef
MY_ALL_DIRS := $(dir $(foreach src_path,$(MY_FILES_PATH), $(call rwildcard,$(src_path),*/)))
MY_ALL_DIRS := $(call uniq,$(MY_ALL_DIRS))

LOCAL_SRC_FILES  := $(MY_SRC_LIST)
LOCAL_C_INCLUDES := $(MY_ALL_DIRS)
```

这样项目中所有 `cpp ` `.c` `.cc` 文件都会被编译。

接着修改 `Application.mk`

``` son
# APP_ABI := armeabi-v7a
```

把原先注释掉的代码放开，`ABI` 修改成我们的虚拟设备的类型 `x86`

```son
APP_ABI := x86
```

接着就可以直接编译了

```shell
cocos compile -p android
```



## 运行 `.apk` 及日志



### 运行

生成的 `.apk` 可以在 `proj.android/bin` 或是 `CocosProtobuf/simulator/android` 中找到。

接下来只需要运行虚拟设备:

```shell
root: emulator avd "设备名"
```

设备就绪后，安装:

```shell
root: adb install CocosProtobuf-debug.apk
```

安装完后，就可以在虚拟设备上看到 `app` 图标了。



### 日志

只是运行还有些不足，还需要能够看到日志。这里可以使用 `Android-SDK` 中提供的工具来实现:

``` shell
root: monitor
```

这条命了会启动 `Eclipse` 中的那种可视化日志调试工具。

 ![log_monitor](http://o7b925er9.bkt.clouddn.com/cocos-android-emulator/log_monitor.png)

接着只需要设置过滤器就可以看到游戏输出的日志了。

 ![log_monitor_filter](http://o7b925er9.bkt.clouddn.com/cocos-android-emulator/log_monitor_filter.png)

 ![log_monitor_filter_result](http://o7b925er9.bkt.clouddn.com/cocos-android-emulator/log_monitor_filter_result.png)


