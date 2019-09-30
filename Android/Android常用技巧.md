# 一、工程编译

## 1、引入第三方 jar

1）首先先去下载需要的jar包

2）将jar包复制到Project下的app–>libs目录下（没有libs目录就新建一个）如下图所示位置：

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160508182034766)

3）右键该jar包，选择add as library，弹出如下窗口：

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160508182621836)

4）点击ok即可，变成下图所示就是导入成功：



![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160508182759432)



## 2、引入第三方 so

以下两种方式二选一



方法一：

1.在src/main中新建jniLibs文件夹 ，把.so复制进去即可

![img](https://img-blog.csdn.net/20160509154349790?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



**<span style="color:red">注意千万不要添加：</span>**否则报 java.lang.UnsatisfiedLinkError

```properties
//    sourceSets {
//        main {
//            jniLibs.srcDirs = ['libs']
//        }
//    }
```





方法二：

1.在app/中新建libs文件夹，把.so复制进去

![img](https://img-blog.csdn.net/20160509154045210?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



2.在app/build.gradle中添加以下五行脚本即可(注：以下脚本意思是会把libs文件夹当成jniLibs文件夹，可以直接用so库了)

```
sourceSets {
    main {
        jniLibs.srcDirs = ['libs']
    }
}
```



贴上完整的app/build.gradle文件,红色的部分是新增的



```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.2"

    defaultConfig {
        applicationId "com.tofu.chat"
        minSdkVersion 16
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    sourceSets {
        main {
            jniLibs.srcDirs = ['libs']
        }
    }
}

dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.1.1'
    compile project(':common')
    compile files('libs/kandy-1.6.244.jar')
}
```