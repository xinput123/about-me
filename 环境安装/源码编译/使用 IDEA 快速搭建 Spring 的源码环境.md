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

#### 2|