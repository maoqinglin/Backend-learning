## 1、Please select Android SDK

从github clone 代码到本地放到AS后发现，发现并不能点“Run”键运行app，并报错Error:Please select Android SDK：

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20170614190154669?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2lsc2NoYW4wMjAx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


最后在File->Project Structure中将Build tools version修改，问题解决。

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20170614190034279?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2lsc2NoYW4wMjAx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



## 2、开发中遇到的Gradle，kotlin插件，AS无法更新或者网络超时问题

解决办法：把你的hosts恢复为原来的，再重启as

主要添加

```
127.0.0.1 localhost
```



## 3、gradle超时问题

gradle\wrapper\gradle-wrapper.properties这个文件，找到类似于下面的这一行：

distributionUrl=https://services.gradle.org/distributions/gradle-x.x-all.zip，将你下载的版本号替换就行，比如你刚下载了4.4的gradle,那么上面的那句话就改成
distributionUrl=https://services.gradle.org/distributions/gradle-4.4-all.zip然后Sync Project Gradle Files，或者重启。

方法二：如果是导入别人的项目的话，更简单的方法是：

然后 打开项目里gradle\wrapper\gradle-wrapper.properties这个文件，找到类似于下面的这一行：

distributionUrl=https://services.gradle.org/distributions/gradle-x.x-all.zip

然后自己新建一个项目，找到该位置的上面的这句话，复制过来替换掉，然后然后Sync Project Gradle Files，或者重启。
