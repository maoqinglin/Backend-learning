# 一、Maven

## 1、配置默认的maven版本

第一种：

在 IntelliJ IDEA 的初始化界面中，依次选择“Configure”—>“Project Defaults”—>“Settings”，然后在“Default Preferences”里的“Maven”中进行配置，即可。

第二种：

在项目中，依然选择“File”—>“Others Settings”，然后在“Default Preferences”里的“Maven”中进行配置，即可。

以上两种方法，都是为了进入到“Default Preferences”中，在其中进行配置，自然就可以默认到新建的 Project 中啦！



## 2、常用命令

mvn clean

mvn package

mvn spring-boot:run



## 3、下载原码

mvn dependency:resolve -Dclassifier=sources



# 二、源码调试技巧

## 1、常用快捷键

```
1)在引用的方法上 CTRL + ALT + 鼠标左击（B）可以实现跳转至实现类，如果有多个实现类会弹出让你选择
2)选中 BeanFactory 类名称，再按 CTRL + ALT + 鼠标左击（B），同样可以展示 BeanFactory 类的所有继承类的关系。
3)Ctrl+shift+Alt+U   查看maven依赖，类图
4)Ctrl+E　　 定位到最近浏览过的文件 
```

