# 一、.ignore使用

## 1、子项目中 .ignore 使用

```properties
*.iml
.gradle
/local.properties
/.idea/**

# ** 代表任意多层级目录
/.gradle/**
/gradle/**
/gradlew
/gradlew.bat
```

**说明：.git 文件在项目的外层 Fold 里**，可以使用如下操作

```pro
git add -A
git status

# 恢复重做
git reset 
```

