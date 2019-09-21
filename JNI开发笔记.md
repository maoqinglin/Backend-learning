# 一、已有工程引入jni

## 1、引入cmake

1）、在 main 目录下创建与 java 同级的 cpp 目录，用于存放 jni 源文件

2）、创建 CMakeLists.txt 文件

**要先创建  CMakeLists.txt  文件，编写 C 代码才会有提示**

如果修改 C 代码后如果不生效，可以刷新文件再编译

```txt
cmake_minimum_required(VERSION 3.4.1)

# 配置so库信息
add_library(
        # 生成的so库名称，此处生成的so文件名称是libjnitest.so；前缀是lib，后缀是.so
        jnitest

        # STATIC：静态库，是目标文件的归档文件，在链接其它目标的时候使用
        # SHARED：动态库，会被动态链接，在运行时被加载
        # MODULE：模块库，是不会被链接到其它目标中的插件，但是可能会在运行时使用dlopen-系列的函数动态链接
        SHARED

        # 资源文件，可以多个，
        # 资源路径是相对路径，相对于本CMakeLists.txt所在目录
        jnitest.cpp
)

# 从系统查找依赖库
find_library( # Sets the name of the path variable.
        # android系统每个类型的库会存放一个特定的位置，而log库存放在log-lib中
        log-lib

        # Specifies the name of the NDK library that
        # you want CMake to locate.
        # android系统在c环境下打log到logcat的库
        log)

# 配置库的链接（依赖关系）
target_link_libraries( # Specifies the target library.
        # 目标库
        jnitest

        # Links the target library to the log library
        # included in the NDK.
        # 依赖于
        ${log-lib})
```

如果有要生成多个 so 库，可以添加多个 add_library 和 target_link_libraries



## 2、转为 C++ Project

**对项目模块右键：Link C++ Project with Gradle**

build.gradle 文件自动生成配置

```gr
externalNativeBuild {
        cmake {
            path file('src/main/cpp/CMakeLists.txt')
        }
}
```

defaultConfig 下可以自己添加配置

```groovy
defaultConfig{
	externalNativeBuild {
            cmake {
                cppFlags ""
                // 生成多个版本的so文件
                abiFilters 'arm64-v8a', 'armeabi-v7a', 'x86', 'x86_64'
            }
        }
}
```



## 3、编写 JNI 源文件

```c++
#include<jni.h>
#include<string>

extern "C" JNIEXPORT jstring JNICALL
Java_com_ireadygo_serialportdemo_MainActivity_sayHello(JNIEnv *env, jobject) {
    std::string hello = "Hello yunyun";
    return env->NewStringUTF(hello.c_str());
}

```



## 4、调用 native 方法

```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Log.i("lmq","MainActivity "+sayHello());
    }

    static {
        System.loadLibrary("jnitest");
    }

    public native String sayHello();
}

```



## 5、问题记录

1.无法识别 C或C++源文件

选则 Build ---》 Refresh Linked C++ Projects



# 二、Android串口通信

## 1、模拟器Root

参考：<https://juejin.im/post/5cd2839de51d453a6c23b080>

注意：

- 1、Android 8.0(包含8.0)以下的系统镜像！！！
- 2、Target里**不带(Google APIs)**的镜像，带(Google APIs)的是不能Root的！！！
- 3、**ABI为x86**的镜像！！！



另外启动的时候：

emulator: ERROR: Running multiple emulators with the same AVD is an experimental feature.

解决方案：

Removing the .lock files did the trick for me. Find the avd and remove the lock files. In a Mac `.android/avd/'NAMEOFAVD.avd directory` . The files I removed were `hardware-qemu.ini.lock` and `multiinstance.lock`.

参考：<https://stackoverflow.com/questions/55328499/emulator-emulator-error-running-multiple-emulators-with-the-same-avd-is-an-ex>



## 2、启动模拟器，映射串口

`emulator @模拟器名称 -writable-system -qemu -serial COM1`
参数：
`-writable-system`以可写的方式打开模拟器(root模拟器需要以此方式打开)
`-qemu -serial COM1`挂载串口COM1



## 3、注意事项

### ② 正确的关闭AVD

**可以点击右上角的x**，**或者直接把运行模拟器的终端关掉**



![img](https://user-gold-cdn.xitu.io/2019/5/8/16a96521a9fffb1c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



！！！**千万别，去长按电源键，然后选Power Off！！！**



![img](https://user-gold-cdn.xitu.io/2019/5/8/16a96521a8a32976?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



如果你这样做，再次打开Super Su：



![img](https://user-gold-cdn.xitu.io/2019/5/8/16a96521b460f8c9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



恭喜，你需要再root一遍了，把这些命令再执行一遍：

```
adb root
adb remount
adb shell 
setenforce 0
quit
adb push xxx/x86/su.pie /system/bin/su
adb push xxx/x86/su.pie /system/xbin/su
chmod 0755 /system/bin/su
chmod 0755 /system/xbin/su
su --install
su --daemon&
quit
复制代码
```

------

### ② 正确的启动/重启AVD

ROOT以后的AVD就不能使用AVD Manager来启动了，都需要使用命令来启动了：

```
emulator -avd Test -writable-system
复制代码
```

如果使用AVD Manager启动了的话，同样会丧失root权限，同样需要重新Root。另外，如果需要重启设备的话，建议使用：**adb reboot** 命令来重启！



1、查看串口是否可用：可以对串口发送数据比如对com1口，echo /dev/ttyS0
2、在Linux查看串口名称使用

   ls -l /dev/ttyS*
  一般情况下串口的名称全部在dev下面，如果你没有外插串口卡的话默认是dev下的ttyS*,一般ttyS0对应com1，ttyS1对应com2，当然也不一定是必然的；

而且在上面的 5.3提到 我是”将模拟器绑定到Windows 的虚拟串口COM1 映射到Android 模拟器中“”所以，才有在打开串口是需要使用的是ttyS1)





参考：

[Android 模拟器串口与PC虚拟串口通讯]: https://blog.csdn.net/gd6321374/article/details/74779770



tcgetattr() failed

