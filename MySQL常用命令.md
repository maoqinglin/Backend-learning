### MySQL常用命令

    1.常看当前所有的数据库
    show databases;
    2.打开指定的库
    use 库名
    3.查看当前库的所有表
    show tables;
    4.查看其它库的所有表
    show tables from 库名;
    5.创建表
    create table 表名(
    	列名 列类型,
    	列名 列类型,
    	......
    )
    6.查看表结构
    desc 表名
    
    7.查看服务器版本
    方式一：登录到mysql服务端
    select version();
    方式二：没有登录到mysql服务端
    mysql --version
    或 mysql --V



### MySQL的语法规范
    1.不区分大小写，但建议关键字大写，表名、列名小写；
    2.每条命令最好用分号结尾
    3.每条命令根据需要，可以进行缩进或换行
    4.注释
    	单行注释：#注释文字
    	单行注释：-- 注释文字
    	多行注释：/* 注释文字 */




