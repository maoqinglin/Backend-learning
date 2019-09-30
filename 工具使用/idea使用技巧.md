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



mvn dependency:resolve -Dclassifier=sources