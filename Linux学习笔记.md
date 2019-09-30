# 一、vi、vim

## 1、常用命令

1）

2）搜索

**/search：向前搜索**

**n：往下找**

**N：往前找**



## 2、常用设置

1）显示行号

 每次打开都显示行号

  修改vi ~/.vimrc 文件，添加：set number



2）SecureCRT中vim行号下划线问题

具体配置如下：
session option–>terminal–>appearance，这里有current color scheme选项，不论选择哪一项，或者是新建的，都可以点击edit…按钮，里面底部有三个复选框，中间一个是show underline，取消选择，那进入vi后如果显示行号，那行号不会有下划线！



# 二、常用命令



## 1、文件上传

方法1：

linux 上传文件 rz命令 提示command not found 解决方法

```bash
yum -y install lrzsz
```



方法2：

windows拷贝文件到Linux，使用 **Alt+P**，会进入：sftp命令窗口，然后直接拖放文件到sftp命令窗口



## 2、网络环境

**1）、CentOS没有ifconfig命令，使用 ip addr**

------

2）、关闭防火墙

Ubuntu

查看防火墙状态：ufw status

关闭防火墙：ufw disable



Centos7.0

查看防火墙状态：firewall-cmd --state

关闭防火墙：systemctl stop firewalld.service



# 三、开发环境配置

## 1、Java开发环境

Centos7下安装与卸载Jdk1.8

查看已经安装的jdk

```bash
[root@node-1 jvm]# rpm -qa|grep jd
```





# 四、虚拟机网络

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

