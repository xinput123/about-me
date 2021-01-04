### 1、准备源码
spring GitHub 地址：https://github.com/spring-projects/spring-framework.git

直接 clone 也可以，你也可以 fork 到自己仓库之后再 clone 

### spring 给出的步骤
见 https://github.com/spring-projects/spring-framework/blob/master/import-into-idea.md

### 环境搭建步骤
#### 1、提前编译操作（通过 spring 给出的 一步骤可以发现）
```
./gradlew :spring-oxm:compileTestJava

./gradlew :spring-core:compileTestJava
```

#### 2、需要注释掉 gradle/docs.gradle 一段内容
dokka方法以及asciidoctor方法注释

![](https://github.com/xinput123/about-me/blob/main/%E7%8E%AF%E5%A2%83%E5%AE%89%E8%A3%85/%E6%BA%90%E7%A0%81%E7%BC%96%E8%AF%91/image/spring%E6%BA%90%E7%A0%81%E7%BC%96%E8%AF%9101.jpg)

#### 3、aspectJ 特别设置下，在 spring 文档中 建议排除掉，但是有时候我们需要使用，所以这里排除


#### 4、编译（此过程时间较长，我电脑编译了 一个多小时……）