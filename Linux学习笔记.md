# 一、





# 二、虚拟机网络

## 1、Virtual Box 桥接网卡问题

桥接网卡 未指定

1）、界面样例

![img](https://img-blog.csdn.net/20180312151759290?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzgzNjk4NjM5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2）、问题存在原因

​    window10中没有安装virtualbox的桥接驱动

3）、解决方法

​    3-1、进入到网络连接，右键打开属性

​            ![img](https://img-blog.csdn.net/20180312152410596?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzgzNjk4NjM5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3-2、没有启动，那么现在开始去安装桥接驱动

​            点击 【安装】----->【服务】------>【添加】----->【从磁盘安装】----->【浏览】

3-3、找到virtualbox目录中的一个文件【VBoxNetLwf】

![img](https://img-blog.csdn.net/20180312152752297?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzgzNjk4NjM5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3-4、找到文件后开始安装        

![img](https://img-blog.csdn.net/20180312153726351?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzgzNjk4NjM5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3-5、安装完后，就可以使用桥接了

参考：

[virtualBox 桥接网卡 未指定]: https://blog.csdn.net/qq_383698639/article/details/79527311



## 2、**VMWare虚拟机IP变成127.0.0.1**

通过 dhclient 命令配置网络接口参数。

- `dhclient -v`



参考：

[获取IP]: https://blog.csdn.net/qq_24452475/article/details/79692065

