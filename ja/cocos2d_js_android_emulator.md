# `Android`エミュレータで`Cocos2d-JS`ゲームを作動及びログ監視の実装


## 目標

`Eclipse` や `Android Studio`など重い`IDE`を使わずに`cocos2d-js`ゲームをコンパイラし`Android`エミュレータでデバッグ(ログ監視)する(導入手順最簡略化)。

### 環境

-   `cocos2d-js v3.10`
-   `Mac OS X`
-   `android-ndk-r9d`
-   `Android 4.4.2 (API 19)`

## インストール

### `jdk`

`.java`ファイルをコンパイラする。[リンク先](http://java.com/en/download/mac_download.jsp)


### `Android-SDK`

署名をつける、`.apk`を作成など。[リンク先](http://android-sdk.en.softonic.com/)


### `Android-NDK`

`c++`をコンパイラする、`.so` `.a`を作成など。[リンク先](http://wear.techbrood.com/tools/sdk/ndk/)

`cocos`コマンドラインツールに合わせて`android-ndk-r9d`を使用。


### 環境配置

`Android-SDK` と `Android-NDK` をインストールした後，`.bash_profile`を修正する必要があります:

``` shell
# NDK
export NDK_ROOT=/Users/android/android-ndk-r9d
export PATH=$PATH:$ANDROID_NDK_ROOT

# SDK
export ANDROID_SDK_ROOT=/Users/android/android-sdk-macosx
export PATH=$PATH:$ANDROID_SDK_ROOT

export PATH=$ANDROID_SDK_ROOT/tools:$ANDROID_SDK_ROOT/platform-tools:$PATH
export PATH=$ANDROID_SDK_ROOT/tools:$ANDROID_SDK_ROOT/build-tools/23.0.3:$PATH
```

修正した後は: `source ~/.bash_profile`。



## 仮想デバイス設置

### `SDK` && `Image`

まずは必要な`SDK`をダウンロードする。

`android-sdk-macosx/tools` ではいろんなコマンドラインツールがあります、`Eclipse`とか`IDE`もそれを利用しています。

例えば:
``` shell
root: android
```

これで`Android-SDK-Manager`というビジュアルツールが作動します。これで必要なモジュールを選択しインストールすることができます。

ここでインストールされたのはご覧の通り:

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

-   `cocos2d-js`のコマンドラインツール`cocos`で作成した`Android`プロジェクトを合わせるために`Android 4.4.2 (API 19)`を選択した

-   `Google APIs Intel x86 Atom System Image` は仮想デバイスのイメージファイルで、`Intel x86 Emulator Accelerator`と一緒に使うとエミュレータを高速にすることができます。


![android_sdk_manager](http://o7b925er9.bkt.clouddn.com/cocos-android-emulator/android_sdk_manager.png)




### `AVD`の作成及び作動

`Image`があれば`ADV`(Android Virtual Device)を作成することができます:

``` shell
root: android avd
```


これで`Android-Vitural-Device-Manager`というビジュアルツールが作動します:

 ![avd](http://o7b925er9.bkt.clouddn.com/cocos-android-emulator/avd.png)



ここで先ダウンロードした`Image`を指定し`AVD`を作成します。

**一つの`AVD`には`Image` + `SDK ` + `Device` + `Skin` より完成する。**

![avd_create_new](http://o7b925er9.bkt.clouddn.com/cocos-android-emulator/avd_create_new.png)


作成した`AVD`は次のコマンドで作動できる:

```shell
root: emulator avd "AVD名"
```

 ![avd_emulator](http://o7b925er9.bkt.clouddn.com/cocos-android-emulator/avd_emulator.png)



## `.apk`作成

`cocos2d-js` から提供した `cocos` という便利なコマンドツールで簡単に `.apk`を作成する
ことができますがその前に少しだけファイルの修正の必要があります。


### ファイルの修正

`cocos` コマンドより作成した`Android`プロジェクトは`proj.android`と言います。

フォルダー構成(CocosProtobufを例に):
```shell
+ CocosProtobuf
	+ frameworks
		+ runtime-src
			+ Classes
			+ proj.android
```

まずは `jni/Android.mk`の修正 :

```
LOCAL_SRC_FILES := hellojavascript/main.cpp \
                   ../../Classes/AppDelegate.cpp 

LOCAL_C_INCLUDES := $(LOCAL_PATH)/../../Classes
```

本来のやり方はコンパイラするソースファイルを全て記入しなければならないのですがここで簡単なやり方あります:

```
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

これでプロジェクトにある全ての `cpp ` `.c` `.cc` がコンパイラされます。

次は `Application.mk`の修正

``` 
# APP_ABI := armeabi-v7a
```

コメントアウトされた行を戻して`APP_ABI`をエミュレータ用の`x86`に変更:

```
APP_ABI := x86
```

これで`cocos`で`.apk`を作成できる:

```shell
cocos compile -p android
```



## `.apk`の作動及びログ監視



### 作動

作成した `.apk` は `proj.android/bin` あるいは `CocosProtobuf/simulator/android` で格納されています。

そしてまずは`AVD`を作動し:

```shell
root: emulator avd "XXX"
```

次のコマンドでインストールする:

```shell
root: adb install CocosProtobuf-debug.apk
```

これで作動したエミュレータで`apk`のアイコンが出てきます。



### ログ監視

ログ監視も`android-sdk-macosx/tools`でのコマンドツールだけで実現できます:

``` shell
root: monitor
```

これより`Eclipse` のようなビジュアルログ監視ツールを作動できます。

 ![log_monitor](http://o7b925er9.bkt.clouddn.com/cocos-android-emulator/log_monitor.png)

これでフィルターを使ってゲームのログを監視することができます。

 ![log_monitor_filter](http://o7b925er9.bkt.clouddn.com/cocos-android-emulator/log_monitor_filter.png)

 ![log_monitor_filter_result](http://o7b925er9.bkt.clouddn.com/cocos-android-emulator/log_monitor_filter_result.png)


